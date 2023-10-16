# Отладка в широком смысле

## Тулчейн

### `gcc`

#### О Machine-Dependent Options

Некоторые опции специфичны для сборки под конкретную архитектуру. Несмотря на то, что в документации перечислены опции для разных архитектур, валидными являются только те, что соответствуют конкретной машине.

Например, `-mavx256-split-unaligned-load` - только для х86; `-mtrack-speculation` - только для aarch64.

#### Умный поиск

```bash
--help
--version
--target-help
--help=<class>
	# class examples: optimizers, target
```

#### Отладка принтами

```
-v
```

#### Окружение (path, prefix, suffix, build, install)

```
-L
-I
-B
--sysroot
```



#### Hot flags

```bash
--std=
# Проверка утечек памяти, проверка гонок по потокам и другие
-fsanitize=

-Wall
-Werror
-Wextra
-fmax-errors=n

# Machine dependent
-m

# Negative form (can be used for setting optimization set):
-f<option> -f-no-<option>
-W<option> -W-no-<option>

# Проброс флагов без влияния порядка следования
-Wa,option
-Wl,option

# GCC work stages (strict order):
1. preprocessor
2. compilation
3. assembler
4. linker

-E # preprocess only
-S # compile only

# no objdump, no relocations info

-c # compile or assemble only

# objdump, relocation info, linking stubs




```



### Список тем

###### C, C++, C# specification settings

Можно выбрать стандарт

###### Static analysis

Collecting profiling information for stat analysis.

###### Program instrumentation

###### Compiler optimizations

```bash
-O0
-Og
-O1 # similar to -O
-O2
-O3
-Ofast # enables optimizations that are not valid for all standard-compliant programs

-Os


# список оптимизаций для конкретного уровня
gcc -O2 --help=optimizers # список примененных флагов
gcc -Q -O2 --help # в стиле таблицы enabled/disabled 

# O0 < Og < O1 < O2 < O3 < Ofast
```

> `-g` - добавить отладочную информацию; `-Og` - применить некоторые оптимизации, но такие, которые оставят большую часть отладочной информации в валидном виде

> -Og < -O1 например потому, что -О1 допускает inlining, а -Og - нет
>
> ```C
> // try with -fno-inline or not
> int goo() {
>     return 5;
> }
> 
> int foo() {
>     const int a = 2;
>     const int b = 3;
>     const int c = a + b;
>     return goo();
> }
> 
> int main() {
> 	return foo();
> }
> ```

**Порядок флагов `-O` важен. Выигрывает последняя. Однаков случае добавления `-f[no-]<option>` - действие опции не отменяется ничем и не зависит от положения относительно `-O`.**

> You can mix options and other arguments.  For the most part, the order you use doesn't matter.  Order does matter when you use several options of the same kind; for example, if  you specify `-L` more than once, the directories are searched in the order specified.  Also, the placement of the `-l` option is significant.
>

###### Debug

```bash
-g # -g0
-ggdb # overrides -g; more appropriate for gdb
-Og
```

###### Preprocessor

```bash
-D<name>=<definition> # -D<name> equals -D<name>=1
-U
-pthread

-M # generate dependence rule
-MM # like -M but without system include dirs
-dM -E <file> # print all predefined macros


-Wp,<option>
```



###### Assembler

```
-Wa,<option>
```



###### Linker

```bash
-nostdlib
-nostartfiles
-pie # use with -fpie, -fPIE
-pthread
-static
-shared

# find lib from std path (see -v) and -L ; works only for obj files before its position
# the order is important
# if lib<name>.so & <name>.a exist then the first will be chosen (if no -static)
-l

-Wl,<option>
```



###### Directory, path, prefixes

```

```

###### Code generation

```
-fpie
-fPIE
-fpic
-fPIC

```

###### Code instrumentation

```bash
-fsanitize=bounds
-fsanitize=null # но срабатывает не всегда, лучше также прибегнуть к valgrind и его тулам
-fsanitize=undefined
-fsanitize=address
-fstack*
```



###### Machine-dependent, target







## Словарь

target

host

build

abi

api

spec

