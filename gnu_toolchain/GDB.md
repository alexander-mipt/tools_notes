# GDB basics
[habr](https://habr.com/ru/post/491534/)
[losst](https://losst.ru/kak-polzovatsya-gdb#6_%D0%92%D1%8B%D0%B2%D0%BE%D0%B4_%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%B8)
## Алгоритм:
1. Скомпилировать с ключом `-g`
2. `gdb ./a.out`
3. Добавить хотя бы одну точку останова, начиная с точки вхождения, т.е. с `main()`:
   `b <номер строки в исходном коде>` или `b main`.
4. `r` - run the program
5. * `n` - next line (без захода в функцию)
   * `s` - step (с заходом в функцию); `s <num>` - step `<num>` lines
   * `c` - continue (к следующей точке останова - *breakpoint*)
   * `f` - run until function exit (not acceptable in main())
6. * `bt` - stack backtrace
   * `p &<tab/symbol>` - get addr of object or symbol
   * `info symbol <addr>` - get symbol from addr

## Basics
* `print <var_name>` - распечатать переменную
* `watch <var_name>` - отслеживать изменение переменной
* `i r` - info registers
* `f` - show current line
* `bt` - trace stack
* `info locals` - значения всех локальных переменных
* `info args` - значения аргументов функции
* `print $<num>` - принт `<num>`-й кешированной переменной
* `ptype <var>` - вывод типа переменнной
* `print &<var>` - вывод адреса переменной
```bash
# list src
list
```
## Comments & Theory
> Комментарии и прочий неисполняемый мусор игнорируется. \
> Отладочная информация занимает от нескольких Кб до нескольких Гб данных. \
> Можно вызывать функции и выражения до их исполнения. \
## registers & flags
```bash
# registers
info registers
i r
i r x0
set $eflags = <val>
set $r1 = <val>
```
## stack
```bash
# frame
info frame
```

## window (TUI)
```bash
# get reg window with changes highlight
layout reg
# show asm
layout asm
```

## memory
```bash
explore
x/<countity><format><unit_size>
x/10xg *<addr>
x/5i $pc
x/100xb $r1
```
#### `unit-size`
* `b` - byte
* `h` - halfword (2B)
* `w` - word (4B)
* `g` - doubleword (8B)
#### `format`
* `o` - octave
* `x` - hex
* `d` - decimal
* `f` - float
* `i` - instruction
* `s` - string

## Advanced (disasm, segfault etc.)
```
# biection between src line and machine code
readelf --debug-dunp=decodedline <file.o>
# get assembly w/ substituted src lines of code
objdump -gdS -M intel <bin> > <file.dis>
``` 
```
Ctrl+x/Ctrl+a - TUI (graphical window in terminal)  on/off
```
```
# print smth in hex/dec/...
p/x p/d
# disasm
disassemble $pc-4,$pc+4
# disasm current function
disassemble
disassemble foo
```

```
# it is possible due to DWARF+tbreak
# go to next line in src in current stack frame
next
n
# go to next line in src (in current stack frame or not)
step
s
# go to next instruction
stepi
# go to next instruction, but do not break inside call instruction
nexti
# go to parent stack frame
finish
# foo(bar(), buz());
# if you want to call foo without breaking in bar or buz then type:
advance buz
```

```
# array[10] - for print all array elems:
p array
# only first
p *array
# print 5 elems in array
p *array@5
```
## builtins
```C++
// dump next pc after func addr
__attribute__((noinline)) void *foo() {
		return __builtin_return_address(0);
}
```

## breakpoints
GDB stores original instr in cache then removes it and substitutes int3 (trace/breakpoint trap)
```
info breakpoints
# breakpoints/objects can be deleted by its positional number

# set temporary breakpoint
tbreak
start # tb main + run
```
Breakpoints can be followed by commands
```
break 19 if some_src_var == 0
# let this breakpoint has a positional number = 5
command 5
<cmd>
end

# common breakpoints
break *<addr>
b main
b <src_file>:<linenum>
# get break info
info break
```

## gdb config
```
vim .gdbinit
# in file:
set pagination off
disassembly-flavor intel
# also add this to allow init gdb w/ file from folder
add-auto-load-safe-path <path_to_project>/.gdbinit
```
