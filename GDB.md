# GDB basics
[src](https://habr.com/ru/post/491534/)
## Алгоритм:
1. Скомпилировать с ключом `-g`
2. `gdb ./a.out`
3. Добавить хотя бы одну точку останова, начиная с точки вхождения, т.е. с `main()`:
   `b <номер строки в исходном коде>` или `b main()`.
   Комментарии и прочий неисполняемый мусор игнорируется.
4. `r` - run the program
5. * `n` - next line (без захода в функцию)
   * `s` - step (с заходом в функцию); `s L` - step `L` lines
   * `c` - continue (к следующей точке останова - *breakpoint*)
   * `f` - run until function exit (not acceptable in main())

## Info
* `print <var_name>` - распечатать переменную
* `watch <var_name>` - отслеживать изменение переменной
* `i r` - info registers
* `f` - show current line
* `bt` - trace stack
* `info locals` - значения всех локальных переменных
* `info args` - значения аргументов функции
* `print $<num>`принт `<num>`-й кешированной переменной
* `ptype <var>` - вывод типа переменнной
* `print &<var>` - вывод адреса переменной
* `x/<size><format> <addr>` - examine mem - просмотр содержимого памяти:
  * `size`:
    * `b` - byte
    * `h` - halfword (2B)
    * `w` - word (4B)
    * `g` - doubleword (8B)
