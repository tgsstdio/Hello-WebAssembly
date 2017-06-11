# WebAssembly Hello World

## Print HELLO WORLD by reading data

[FIDDLE](https://wasdk.github.io/WasmFiddle/?wvzhb)

### WASM Original C source
``` C
char* hello() {
  return "Hello World";
}
```

### WASM AST

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

### Browser/client end

``` javascript
function utf8ToString(h, p) {
  let s = "";
  for (i = p; h[i]; i++) {
    s += String.fromCharCode(h[i]);
  }
  return s;
}

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

## IMPORT functions

### WASM Original C source
[FIDDLE](https://wasdk.github.io/WasmFiddle/?jn7tx)
``` C
void printHW();

void hello() {
  printHW();
}
```

### Browser/client end
``` Javascript 
function printHW() {
  log("HELLO WORLD");
}

let i = { env:{ printHW: printHW } };
let m = new WebAssembly.Instance(new WebAssembly.Module(buffer), i);
let h = new Uint8Array(m.exports.memory.buffer);
m.exports.hello();
```

## Reading Data out of heap memory

### WASM Original C source

[FIDDLE](https://wasdk.github.io/WasmFiddle/?15zq3v)
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

### WAST

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

### Browser/Client
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