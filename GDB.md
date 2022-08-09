# GDB basics
[habr](https://habr.com/ru/post/491534/)
[losst](https://losst.ru/kak-polzovatsya-gdb#6_%D0%92%D1%8B%D0%B2%D0%BE%D0%B4_%D0%B8%D0%BD%D1%84%D0%BE%D1%80%D0%BC%D0%B0%D1%86%D0%B8%D0%B8)
## Алгоритм:
1. Скомпилировать с ключом `-g`
2. `gdb ./a.out`
3. Добавить хотя бы одну точку останова, начиная с точки вхождения, т.е. с `main()`:
   `b <номер строки в исходном коде>` или `b main`.
   Комментарии и прочий неисполняемый мусор игнорируется.
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
* `x/<size><format> <addr>` - examine mem - просмотр содержимого памяти:
   * `size`:
      * `b` - byte
      * `h` - halfword (2B)
      * `w` - word (4B)
      * `g` - doubleword (8B)
   * `format`:
      * `o` - octave
      * `x` - hex
      * `d` - decimal
      * `f` - float
      * `i` - instruction
      * `s` - string
## Advanced (disasm, segfault etc.)
