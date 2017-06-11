# Hello-WebAssembly

"Hello World" WebAssembly examples written in the [WasmFiddle](https://wasdk.github.io/WasmFiddle/) HTML browser editor

- [Option 1](#Option-1) - Print HELLO WORLD by reading hardcoded data [LINK](https://wasdk.github.io/WasmFiddle/?wvzhb)

- [Option 2](#Option-2) - Using an js import function to output 'HELLO WORLD' [LINK](https://wasdk.github.io/WasmFiddle/?jn7tx)

- [Option 3](#Option-3) - Writing out 'HELLO WORLD' into memory and observing the change via dumps [LINK](https://wasdk.github.io/WasmFiddle/?15zq3v)

- [Option 4](#Option-4) - Printing HELLO WORLD by reading memory (i.e. similar to Option 3) [LINK](https://wasdk.github.io/WasmFiddle/?1hayln)

-----

## Option 1

> Print HELLO WORLD by reading hardcoded data

[WASMFIDDLE](https://wasdk.github.io/WasmFiddle/?wvzhb)

### C source (.c)

``` C
char* hello() {
  return "Hello World";
}
```

### WebAssembly syntax tree (.wast)

``` LISP
(module
  (table 0 anyfunc)
  (memory $0 1)
  (data (i32.const 16) "Hello World\00")
  (export "memory" (memory $0))
  (export "hello" (func $hello))
  (func $hello (result i32)
    (i32.const 16)
  )
)
```

### Browser/client end (.js)

``` javascript
function utf8ToString(h, p) {
  let s = "";
  for (i = p; h[i]; i++) {
    s += String.fromCharCode(h[i]);
  }
  return s;
}

let m = new WebAssembly.Instance(new WebAssembly.Module(buffer));
let h = new Uint8Array(m.exports.memory.buffer);
let p = m.exports.hello();
log(utf8ToString(h, p))
```

### Output
```
Hello World
```

-----

## Option 2

> Using an js import function

[WasmFiddle](https://wasdk.github.io/WasmFiddle/?jn7tx)

### C source (.c)

``` C
void printHW();

void hello() {
  printHW();
}
```

### WebAssembly syntax tree (.wast)

``` LISP
(module
  (type $FUNCSIG$v (func))
  (import "env" "printHW" (func $printHW))
  (table 0 anyfunc)
  (memory $0 1)
  (export "memory" (memory $0))
  (export "hello" (func $hello))
  (func $hello
    (call $printHW)
  )
)
```

### Browser/client end (.js)

``` Javascript 
function printHW() {
  log("HELLO WORLD");
}

let i = { env:{ printHW: printHW } };
let m = new WebAssembly.Instance(new WebAssembly.Module(buffer), i);
let h = new Uint8Array(m.exports.memory.buffer);
m.exports.hello();
```

### Output

```
HELLO WORLD
```

-----

## Option 3

> Writing out 'HELLO WORLD' into memory and observing the change via dumps 

[WasmFiddle](https://wasdk.github.io/WasmFiddle/?15zq3v)

### C source (.c)

``` C 
char inputs[128];

char* getInputs() {
  return &inputs[0];
}

int main() {
  int i = 0;
  inputs[i++] = 'H';
  inputs[i++] = 'E';
  inputs[i++] = 'L';
  inputs[i++] = 'L';  
  inputs[i++] = 'O';
  inputs[i++] = ' ';
  inputs[i++] = 'W';
  inputs[i++] = 'O';
  inputs[i++] = 'R';
  inputs[i++] = 'L';  
  inputs[i++] = 'D';  
  inputs[i++] = 0;
  return 42;
}
```

### WebAssembly syntax tree (.wast)

``` LISP
(module
  (table 0 anyfunc)
  (memory $0 1)
  (export "memory" (memory $0))
  (export "getInputs" (func $getInputs))
  (export "main" (func $main))
  (func $getInputs (result i32)
    (i32.const 16)
  )
  (func $main (result i32)
    (i32.store offset=16
      (i32.const 0)
      (i32.const 1280066888)
    )
    (i32.store16 offset=20
      (i32.const 0)
      (i32.const 8271)
    )
    (i32.store offset=22 align=2
      (i32.const 0)
      (i32.const 1280462679)
    )
    (i32.store16 offset=26
      (i32.const 0)
      (i32.const 68)
    )
    (i32.const 42)
  )
)
```

### Browser/client end (.js)

``` javascript
var wasmModule = new WebAssembly.Module(wasmCode);
var imp = { };
var wasmInstance = new WebAssembly.Instance(wasmModule, imp);
const arrayBuffer = wasmInstance.exports.memory.buffer;
const buffer = new Uint8Array(arrayBuffer);
const BYTE_LENGTH = 64;
let offset = wasmInstance.exports.getInputs();
lib.dumpMemory(buffer, offset, BYTE_LENGTH);
log(wasmInstance.exports.main());
lib.dumpMemory(buffer, offset, BYTE_LENGTH);
```

### OUTPUT

```
00000010 00000000000000000000000000000000 ................
00000020 00000000000000000000000000000000 ................
00000030 00000000000000000000000000000000 ................
00000040 00000000000000000000000000000000 ................

42
00000010 48454C4C4F20574F524C440000000000 HELLO WORLD.....
00000020 00000000000000000000000000000000 ................
00000030 00000000000000000000000000000000 ................
00000040 00000000000000000000000000000000 ................

```

-----

## Option 4

> Print HELLO WORLD by reading memory

[WasmFiddle](https://wasdk.github.io/WasmFiddle/?1hayln)

### C source (.c)

``` C
char inputs[128];

char* getInputs() {
  return &inputs[0];
}

int setMessage() {
  int i = 0;
  inputs[i++] = 'H';
  inputs[i++] = 'E';
  inputs[i++] = 'L';
  inputs[i++] = 'L';  
  inputs[i++] = 'O';
  inputs[i++] = ' ';
  inputs[i++] = 'W';
  inputs[i++] = 'O';
  inputs[i++] = 'R';
  inputs[i++] = 'L';  
  inputs[i++] = 'D';  
  inputs[i] = 0;
  return i;
}
```

### WebAssembly syntax tree (.wast)

``` LISP
(module
  (table 0 anyfunc)
  (memory $0 1)
  (export "memory" (memory $0))
  (export "getInputs" (func $getInputs))
  (export "setMessage" (func $setMessage))
  (func $getInputs (result i32)
    (i32.const 16)
  )
  (func $setMessage (result i32)
    (i32.store offset=16
      (i32.const 0)
      (i32.const 1280066888)
    )
    (i32.store16 offset=20
      (i32.const 0)
      (i32.const 8271)
    )
    (i32.store offset=22 align=2
      (i32.const 0)
      (i32.const 1280462679)
    )
    (i32.store16 offset=26
      (i32.const 0)
      (i32.const 68)
    )
    (i32.const 11)
  )
```

### Browser/client end (.js)

``` Javascript
function utf8ToString(h, p) {
  let s = "";
  for (i = p; h[i]; i++) {
    s += String.fromCharCode(h[i]);
  }
  return s;
}

var wasmModule = new WebAssembly.Module(wasmCode);
var imp = { };
var wasmInstance = new WebAssembly.Instance(wasmModule, imp);
const arrayBuffer = wasmInstance.exports.memory.buffer;
const buffer = new Uint8Array(arrayBuffer);
const BYTE_LENGTH = 64;
let offset = wasmInstance.exports.getInputs();
log(wasmInstance.exports.setMessage());
log(utf8ToString(buffer, offset));
```

### OUTPUT

```
11
HELLO WORLD
```