# Lecture 2 Linkers

Единица трансляции в Си - файл.

###  Зачем нужны заголовочный файлы?

Без них нам бы в каждый файл пришлось вставлять объявления. И copy-past рано или поздно привел бы к ошибкам. В случае языка С++ разное манглирование привло бы к появлению перегрузок. В языке Си - еще хуже - к **undefined behavior**.

### Назначение ассемблера

* Декодирование
* Сбор информации о релокациях

Линковка функций до запуска компоновщика `ld`

```
objdump -d -M intel main.o

0000000000000000 <main>:
   0:	f3 0f 1e fa          	endbr64 
...
   d:	e8 00 00 00 00       	call   12 <main+0x12>
  12:	48 8d 35 00 00 00 00 	lea    rsi,[rip+0x0]        # 19 <main+0x19>
...
  2d:	c3                   	ret
```

`endbr64` -- инструкция, которая разрешает данному PC быть таргетом для индиректного бранча. Можно настроить процессор так, что прыжки в другие адреса будут запрещены.

После линковки (статической):

```
... <main>:
...
    1136:	e8 07 00 00 00       	call   1142 <foo>
...

00..1142 <foo>:
	1142:	...	endbr64
```

Адрес вычисляется следующим образом:

```
<callPC> call: e8 <offset>
newPC = <offset> + 0x4 + <callPC> 
```

А как линкер вычисляет правильный оффсет? Дело в том, что эту информацию прописывает ассемблер при кодировании инструкций. Это информация хранится в таблице релокаций.

```
objdump -r main.o

main.o:     file format elf64-x86-64

RELOCATION RECORDS FOR [.text]:
OFFSET           TYPE              VALUE 
000000000000000e R_X86_64_PLT32    foo-0x0000000000000004
```

Вычислим вручную согласно спецификации для `R_X86_64_PLT32`:

Согласно [источнику](https://www.ucw.cz/~hubicka/papers/abi/node19.html) :

> **A** - Represents the addend used to compute the value of the relocatable field.
> **B** - Represents the base address at which a shared object has been loaded into memory during execution. Generally, a shared object is built with a 0 base virtual address, but the execution address will be different.
> **G** - Represents the offset into the global offset table at which the relocation entry's symbol will reside during execution.
> **GOT** - Represents the address of the global offset table.
> **L** - Represents the place (section offset or address) of the Procedure Linkage Table entry for a symbol.
> **P** - Represents the place (section offset or address) of the storage unit being relocated (computed using r_offset).
> **S** - Represents the value of the symbol whose index resides in the relocation entry.

`R_x86_64_PLT32 = L + A - P`

В нашем случае:

`P` - адрес, по которому записан **адрес смещения функции перехода (call)**.

Так, если мы имеем:

 ```
 1136:	e8 07 00 00 00       	call   1142 <foo>
 ```

То 

* `P = 0x1136 + 1 = 0x1137` (can be obtained by `objdump -d <executable>.out`)

* `S = value of symbol foo = PC of foo = 0x1142` (can be obtained via `nm <executable>.out` or `objdump`)
* `A = 0x4` (can be obtained via `readelf -r <objfile>.o` as "Append")

Таким образом:

```
Offset = S + A - P = 0x1142 - 0x4 - 0x1137 = 0x7 - see call;

Offset also can be obtained via readelf -r <objdump>.o
```





## Порядок линковки

Порядок линковки имеет значение

```C
// a.c
int a = 1;
```

```C
// b.c
extern int a;
int b;
int foo() {
	b = a;
}
```

```C
// main.c
int foo();
int main() {
	return foo();
}
```

`-l` **имеет область видимости текущий файл и следующих за ним.**

Порядок `-L` не важен.

```
gcc -L. -la -lb main.c # do not work, main.o do not see libs
gcc main.c -L. -la -lb # do not work, libb.a do not see liba.a
gcc main.c -L. -la -lb -la # OK
gcc main.c -L. -lb -la # OK

# а тут порядок не важен
gcc main.c -L. -Wl,-\( -la -lb -Wl,-\)
gcc main.c -L. -Wl,-\( -lb -la -Wl,-\)
```

Таким образом, флаги линковки должны идти после флагов компиляции.

`-static` - статическое разрешение релокаций. Как следствие, каждая программа будет располагаться в оперативной памяти вместе со своей статически слинкованной сущностью.

Плюс: переносимость

Минус: невозможность иметь одну копию библиотеки на множество процессов

## О статической линковке

Чем статический архив (`lib<smth>.a`) из множества объектников лучше, чем статическая библиотека из одного объектника или передача всех объектников на компилячю с `main`?

```C
// a.c
int aoo() {}
```

```C
// b.c
int boo() {}
```

```C
// ab.c
int aoo() {}
int boo() {}
```

```C
// header.h
int aoo();
int boo();
```

 ```C
 #include "header.h"
 int main() {
 	aoo();
 }
 ```

```bash
gcc -c a.c
gcc -c b.c
gcc -c ab.c
ar -cr libsep.a a.o b.o
ar -cr liball.a ab.o
```

```bash
# Случай с неразнесенными определениями функций
gcc main.c -L. -lall # или gcc main.c a.o b.o
objdump -d -M intel a.out
```

```
0000000000001129 <main>:
    1129:	f3 0f 1e fa          	endbr64 
    112d:	55                   	push   %rbp
    112e:	48 89 e5             	mov    %rsp,%rbp
    1131:	b8 00 00 00 00       	mov    $0x0,%eax
    1136:	e8 07 00 00 00       	call   1142 <aoo>
    113b:	b8 00 00 00 00       	mov    $0x0,%eax
    1140:	5d                   	pop    %rbp
    1141:	c3                   	ret    

0000000000001142 <aoo>:
    1142:	f3 0f 1e fa          	endbr64 
    1146:	55                   	push   %rbp
    1147:	48 89 e5             	mov    %rsp,%rbp
    114a:	90                   	nop
    114b:	5d                   	pop    %rbp
    114c:	c3                   	ret    

000000000000114d <boo>:
    114d:	f3 0f 1e fa          	endbr64 
    1151:	55                   	push   %rbp
    1152:	48 89 e5             	mov    %rsp,%rbp
    1155:	90                   	nop
    1156:	5d                   	pop    %rbp
    1157:	c3                   	ret 
```

Таким образом, в программе используется только `aoo`, однако в коде мы видим также реализацию `boo`.

А теперь воспользуемся линковкой с архивом с разнесенными по объектникам функциями

```bash
# статическая библиотека с разнесенными объектниками
gcc main.c -L. -lsep
```

```
0000000000001129 <main>:
    1129:	f3 0f 1e fa          	endbr64 
    112d:	55                   	push   %rbp
    112e:	48 89 e5             	mov    %rsp,%rbp
    1131:	b8 00 00 00 00       	mov    $0x0,%eax
    1136:	e8 07 00 00 00       	call   1142 <aoo>
    113b:	b8 00 00 00 00       	mov    $0x0,%eax
    1140:	5d                   	pop    %rbp
    1141:	c3                   	ret    

0000000000001142 <aoo>:
    1142:	f3 0f 1e fa          	endbr64 
    1146:	55                   	push   %rbp
    1147:	48 89 e5             	mov    %rsp,%rbp
    114a:	90                   	nop
    114b:	5d                   	pop    %rbp
    114c:	c3                   	ret
```

Видим, что реализация `boo` не содержится в коде. Круто.



## Динамическая линковка

Проблема в том, что, чтобы линковать в динамике, нужно что-то прилинковать в статике (например, сам линкер). Иначе кто будет заниматься линковкой? Или это не так работает?

```bash
# Создание динамической библиотеки
gcc -shared <file> -o lib<file>.so

# на ARM нужно также добавить -fPIC (более общий, чем -fpic)

# Использование
gcc main.c -L. -Wl,-rpath,.
```

Заметим, что `gcc main.c -L. -Wl,-rpath,. -shared` вызовет `segfault`, а не ошибку компиляции. Дело в том, что сама динамическая библиотека - это исполняемый файл. Просто в нем не предусмотрен код для запуска в одиночке.

При линковке библиотеки с главным бинарником информация о динамическом линкере хранится в `.interp`: ascii строка - путь к динамическому линкеру `ld-<os-arch>.so.2`. Его запускает сама динамическая библиотека, ибо она исполняемая.

```bash
# Читаем секцию как ASCII-строку
readelf -p .interp ./aaa
String dump of section '.interp':
  [     0]  /lib64/ld-linux-x86-64.so.2
```







 
