# WASM Malloc Performance

This is a proof of concept for a proposal in https://github.com/systemjs/systemjs/issues/1671.

To investigate the performance of binding malloc through JS instead of directly through WASM there are two cases of using malloc in Web Assembly, with a simple module containing the following code transpiled into Web Assembly:

```c
#include <stdlib.h>

struct Record {
  int id;
  float x;
  float y;
};

struct Record* createRecord (int id, float x, float y) {
  struct Record* r = malloc(sizeof(struct Record));
  r->id = id;
  r->x = x;
  r->y = y;
  return r;
}

void deleteRecord (struct Record* addr) {
  free(addr);
}
```

1. [malloc-js-bound](malloc-js-bound/test.html): In this example, the main program exports its memory, using a JS binding trick to get a malloc bound to its own exported memory.
2. [malloc-wasm-bound](malloc-wasm-bound/test.html): In this example, the main program is compiled to import its memory, which is initialized in JS before being passed into both the malloc and program modules.

### Performance Results

Here is a random sample of results from Chrome Canary 60.0.3108.0 for 100000 record allocations:

```
  1. malloc-js-bound
    24.905
    22.760
    25.495
    28.285
    26.575
    30.660
    30.010
    25.065
    29.225
    27.335
    33.345
    23.690
    30.575
    27.285
    27.515
    39.220
  --------
  Avg: 28.247

  2. malloc-wasm-bound
    26.115
    19.625
    18.525
    35.750
    24.620
    25.405
    29.365
    25.305
    26.205
    25.285
    19.650
    23.495
    18.305
    23.510
    54.165
    26.400
  --------
  Avg: 26.358
```
