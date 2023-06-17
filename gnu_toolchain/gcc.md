# CPP - GCC preprocessor notes
## How to determine default `#include` path
```bash
cpp -v /dev/null -o /dev/null
```
use `-I` to add include path.
`<...>` - search in system dirs \
`"..."` - search in local folder **and then** works like `<...>`
## How to determine all predefined macros
```bash
gcc -dM -E - < /dev/null
```
> It depends on compiler flags!
