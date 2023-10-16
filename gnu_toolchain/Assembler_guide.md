# Assembler guide

## Basic asm

```
asm asm-qualifiers ( AssemblerInstructions )

asm("<instr1>\t\n<instr2>...")
__asm__("<instr1>\t\n<instr2>...")
```

`__asm__` is better, because it **works even when gcc extensions are disabled** (`-pedantic -Werror`)

Always **volatile**. Istructions in it can not be modidifed or reordered

Has a file scope, can be produced outside function

All symbols in basic asm code block are unvisible

Sensitive to asm dialect (AT&T/intel/...)

sequence of asm blocks do not remain perfectly consecutive, instructions in one asm block stay consequtive

### Examples

```C
// add x3 = x1 + x2 for arm
__asm__("add x3, x1, x2");

// software breakpoint
__asm("BKPT");
```

## Extended asm

```
asm asm-qualifiers ( AssemblerTemplate 
                 : OutputOperands 
                 [ : InputOperands
                 [ : Clobbers ] ])

asm asm-qualifiers ( AssemblerTemplate 
                      : OutputOperands
                      : InputOperands
                      : Clobbers
                      : GotoLabels)
```

Extended asm can be only inside function

Extended asm statements that have no output operands and `goto` statements, are implicitly volatile. **Здесь подразумевается не их наличие в выражении, а их использование в коде:**

```C
// insert this code and try to compile with
// -masm=intel -O2 -DNDEBUG
// -masm=intel -O2

#include <stdint.h>
#include <assert.h>

void DoCheck(uint32_t dwSomeValue)
{
   uint32_t dwRes;

   // Assumes dwSomeValue is not zero.
   asm volatile ("bsf %1,%0"
     : "=r" (dwRes)
     : "r" (dwSomeValue)
     : "cc");

   assert(dwRes > 3);
}

int main() {
    DoCheck(2);
    return 0;
}
```



```C
// simple example.c
// dst += src; return dst
int foo(int dst, int src) { 
asm volatile(
    // ".intel_syntax\n\t" 
    "add %0, %1\n\t" // or "{add %1, %0|add %0, %1}\n\t" - see Assembler emplates 6.47.2.2
    // ".att_syntax\n\t"
    : "+r" (dst) 
    : "r" (src));
    return dst;

}

int main() {
    return foo(2, 3);
}
```

### Важные замечания

При работе с ассемблером нужно четко представлять себе, с каким диалектом вы работаете (**AT&T** or **Intel**). `gcc example.c && ./a.out` выдает результат 2, что неверно. Однако использование директив `.intel_syntax`, `.att_syntax` или `-masm=intel`  решает данную проблему.

Другой проблемой является ответственность пользователя за аттрибуты. Ошибки в них могут не диагностироваться компилятором. Так, замена`"+r"` на `"=r"` приведет к неправильной работе кода.

Порядок следования asm блоков или положение asm блока внутри С кода вследствие оптимизаций компилятора может  быть изменен. Согласно документации, иногда даже в случаях c `volatile`:

> Note that the compiler can move even `volatile asm` instructions relative to other code, including across jump instructions.

## Другие особенности

То, что можно прочитать в документации:

* Интерфейс для чтения/записи архитектурных флагов
* 

### Замечания по использованию GodBolt

При использовании `execute from this` флаги окна компиляции и исполнения лучше держать одинаковыми.

```C
inline int foo(int reg)
{
   asm volatile (
    "add %0, 1\n\t"
    : "+r" (reg));
    return reg;
}

int main() {
    return foo(3);
}
```

Можно попробовать собрать этот код под x86-64 gcc 13.1 с опциями компилятора `-masm=intel -O2` --> execute from this --> удалить эти флаги в execute from this ([godbolt link](https://godbolt.org/z/xjsb34vjz)) 
