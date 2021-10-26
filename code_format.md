# Code format
## auto-formater
make changes
```bash
clang-format -sort-includes -style=llvm -i <file>
```
dump format config into file
```bash
clang-format -style=llvm -dump-config > .clang-format
```
this file can be included in VSCode formatter
**Preferences --> Settings --> find `C_Cpp: Clang_format_path`**

## formatting on the spot
```
clang-format -style="{BasedOnStyle: llvm, IndentWidth: 4}" -i <file>
```
