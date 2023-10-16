# GDB

## Prepare to debug

```bash
echo 0 > /proc/sys/kernel/randomize_va_space
gcc -fno-PIC -fno-PIE -ggdb -O0 -no-pie src.cc # exotic: -fno-eliminate-unused-debug-symbols -fno-eliminate-unused-debug-types
gdb -tui -ex start --args ./a.out
```

## Initialization

```
set pagination off
set print pretty on
start / continue
```

## Reading info

```
info sources
info functions
info variables
info scope <FunctionName>
info files
info registers

print $pc
print main
print $rax
```

## Breakpoints

```
b foo
b foo(int, int)
b <file>:12
b 12
b <class>::<method>
b <addr> # can be used for instrs
tbreak <function> / <file>:<line> / <line> / <addr>

info breakpoints
delete <breakpoint_num>
```

```bash
# break using current context
tb <line> if <context_val> == <val>
commands
break <func>
continue
end
```

```bash
# break using local context of breakpoint
tb <line> if <context_val> == <val>
```

#### About breakpoints

> When gdb breaks on some line then the context of its line is available. **But values are not still initialized. You have to do `next` before analyzing them.**
>
> ```
> break 80
> gdb output: for (int i = 0; i < 10; ++i) {
> info locals
> next
> info locals
> ```

## Watchpoints

```
watch # write
awatch # read or write
rwatch # read

watch if <expr>
watch if i == 3 # breaks when i new val equals 3

watch $rax
rwatch *<addr>

info watchpoints
delete <watchpoint_num>
```



## Printing src

### Print instructions

```
disas $pc-<start_off>,+<len>
```

> `$pc` points to the next instruction after the place of interrupt. For example, after interrupting on breakpoint `break main` `$pc` equals `main + <instr_size>`

> Также легко запомнить: `<len>` - количество байт, которое будет отображено, начиная со стартовой позиции. Если байты могут быть отображены в инструкции - будут отображаться инструкции.

**Замечание.** Лучше дизассемблировать с `<start_off> = 0`, иначе из-за неправильного выравнивания по инструкциям они будут интерпретироваться неверно.d

### Print src

```
list <function>
list <first_line> <last_line>
gdb -tui ...

show listsize
set listsize <lines_view_width>
list <line>
```

### Print object

Can process class objects & containers

```
print <obj>
```

### Print type info

Can be useful for deducing types

```
ptype <var>
whatis <var>
```

### Print memory

```
print main
print &<var>
print *<var>
```



## Setting vars

```
set $<gdb_var_name> = <value>
set var <src_var> = <value> 
```



## User-defined functions

We have already discussed break commands. We also can define general functions.

```
define gdb_foo
<command1>
...
<commandN>
end
```

## Links

[Manual](https://www.sourceware.org/gdb/onlinedocs/gdb/)

https://developer.apple.com/library/archive/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html
