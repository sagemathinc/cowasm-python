# CoWasm\-Python: Collaborative Python in WebAssembly for Servers and Browsers

URL: https://github.com/sagemathinc/cowasm-python

DEMOS:

- https://cowasm.org \- Python using SharedArrayBuffers
- https://zython.org \- Python using Service Workers
- https://cowasm.sh \- Dash Shell with Python, Sqlite, Lua, etc., using SharedArrayBuffers

Type this if you have nodejs at least version 16 installed:

```sh
~$ npx python-wasm
Python 3.11.0 (main, Oct 27 2022, 10:03:11) [Clang 15.0.3 (git@github.com:ziglang/zig-bootstrap.git 0ce789d0f7a4d89fdc4d9571 on wasi
Type "help", "copyright", "credits" or "license" for more information.
>>> 2 + 3
5
>>> import numpy
>>> import sympy
...
```

## What is this?

See  https://github.com/sagemathinc/cowasm for more about the CoWasm project.

<!--
[<img src="https://github.com/sagemathinc/cowasm/actions/workflows/docker-image.yml/badge.svg"  alt="Docker Image CI"  width="172px"  height="20px"  style="object-fit:cover"/>](https://github.com/sagemathinc/cowasm/actions/workflows/docker-image.yml)
-->

### Try python-wasm

Try the python-wasm REPL under node.js (version at least 16):

```py
~$ npx python-wasm
Python 3.11.0 (main, Oct 27 2022, 10:03:11) [Clang 15.0.3 (git@github.com:ziglang/zig-bootstrap.git 0ce789d0f7a4d89fdc4d9571 on wasi
Type "help", "copyright", "credits" or "license" for more information.
>>> 2 + 3
5
>>> import sys; sys.version
'3.11.0 (main, Oct 27 2022, 10:03:11) [Clang 15.0.3 (git@github.com:ziglang/zig-bootstrap.git 0ce789d0f7a4d89fdc4d9571'
>>> sys.platform
'wasi'
```

### Install python\-wasm

Install python\-wasm into your project, and try it via the library interface and the node.js terminal.

```sh
npm install python-wasm
```

Then from the nodejs REPL:

```js
~/cowasm/packages/python-wasm$ node
Welcome to Node.js v19.0.0.
Type ".help" for more information.
> {syncPython, asyncPython} = require('python-wasm')
{
  syncPython: [AsyncFunction: syncPython],
  asyncPython: [AsyncFunction: asyncPython],
  default: [AsyncFunction: asyncPython]
}
> python = await syncPython(); 0;
0
> python.exec('import sys')
undefined
> python.repr('sys.version')
"'3.11.0b3 (main, Jul 14 2022, 22:22:40) [Clang 13.0.1 (git@github.com:ziglang/zig-bootstrap.git 623481199fe17f4311cbdbbf'"
> python.exec('import numpy')
undefined
> python.repr('numpy.linspace(0, 10, num=5)')
'array([ 0. ,  2.5,  5. ,  7.5, 10. ])'
```

There is also a Python REPL that is part of python\-wasm:

```py
> python.terminal()
Python 3.11.0 (main, Oct 27 2022, 10:03:11) [Clang 15.0.3 (git@github.com:ziglang/zig-bootstrap.git 0ce789d0f7a4d89fdc4d9571 on wasi
Type "help", "copyright", "credits" or "license" for more information.
>>> 2 + 3   # you can edit using readline
5
>>> input('name? ')*3
name? william  <-- I just typed "william"
'williamwilliamwilliam'
>>> import time; time.sleep(3)  # sleep works
>>> while True: pass   # ctrl+c works, but exits this terminal
> # back in node.
```

You can also use python\-wasm in your own [web applications via webpack](https://github.com/sagemathinc/cowasm-browser/tree/main/packages/terminal).  In the browser, this transparently uses SharedArrayBuffers if available, and falls back to ServiceWorkers.

## Build from Source

We support and regularly test building CoWasm\-Python from source on the following platforms:

- x86 macOS and aarch64 macOS (Apple Silicon M1)
- x86 and aarch64 Linux

### Prerequisites

You do NOT need to install Zig or Node.js, and different packages have different requirements. A tested version of each will be
downloaded as part of the build process. Currently, this is the latest released
version of Zig and the latest released version of Node. It doesn't matter if
you have random versions of Node or Zig on your system already.  The dependency you need for every possible package are as follows:

- On MacOS, install the [XCode command line tools.](https://developer.apple.com/xcode/resources/) 

- On Linux apt\-based system, e.g., on Ubuntu 22.04:

```sh
apt-get install git make cmake curl dpkg-dev m4 yasm texinfo python-is-python3 libtool tcl zip libncurses-dev
```

- On Linux RPM based system, e.g., Fedora 37:

```sh
dnf install git make cmake curl dpkg-dev m4 yasm texinfo libtool tcl zip ncurses-devel perl
```

- Currently, the only way to build CoWasm from source on MS Windows is to use a Docker container running Linux.  Using WSL2 (maybe) works but is too slow.

### Build

See the build section of [https://github.com/sagemathinc/cowasm](https://github.com/sagemathinc/cowasm).

## Benchmarks

There is a collection of cpu\-intensive benchmarks in [packages/bench/src](./packages/bench/src), which you can run under various Python interpreters by running

```sh
your-python-interpreter src/all.py
```

Here are some grand total times. The timings are pretty stable, and the parameters of the benchmarks are chosen so a single benchmark doesn't unduly impact the results \(e.g., it is trivial to game any such benchmark by adjusting parameters\).

| Python                                                                  | x86_64 Linux | MacOS M1 max | aarch64 Linux (docker on M1 max) |
| :---------------------------------------------------------------------- | :----------: | :----------: | :------------------------------: |
| PyPy 3.9.x (Python reimplemented with a JIT)                            |   2997 ms    |   2127 ms    |       1514 ms (ver 3.6.9)        |
| pylang (Javascript Python -- see https://github.com/sagemathinc/pylang) |   6909 ms    |   2876 ms    |             4424 ms              |
| Native CPython 3.11                                                     |   9284 ms    |   4491 ms    |             4607 ms              |
| WebAssembly CPython (python-wasm)                                       |   23109 ms   |   12171 ms   |             12909 ms             |

<br/>

The quick summary is that in each case pypy is twice as fast as pylang \(basically node.js\), python\-lang is twice as fast as cpython, and _**native cpython is about 2.5x\-2.8x as fast as python\-wasm**_. However, when you study the individual benchmarks, there are some significant differences. E.g., in `brython.py` there is a benchmark "create instance of simple class" and it typically takes 4x\-5x longer in WebAssembly versus native CPython.

---

## Contact

Email [wstein@cocalc.com](mailto:wstein@cocalc.com) if you find this interesting and want to help out. **This is an open source 3\-clause BSD licensed project.**

