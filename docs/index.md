# watchfiles

[![CI](https://github.com/samuelcolvin/watchfiles/workflows/ci/badge.svg?event=push)](https://github.com/samuelcolvin/watchfiles/actions?query=event%3Apush+branch%3Amain+workflow%3Aci)
[![Coverage](https://codecov.io/gh/samuelcolvin/watchfiles/branch/main/graph/badge.svg)](https://codecov.io/gh/samuelcolvin/watchfiles)
[![pypi](https://img.shields.io/pypi/v/watchfiles.svg)](https://pypi.python.org/pypi/watchfiles)
[![license](https://img.shields.io/github/license/samuelcolvin/watchfiles.svg)](https://github.com/samuelcolvin/watchfiles/blob/main/LICENSE)

Simple, modern and high performance file watching and code reload in python.

---

## NOTICE

This package was significantly altered and renamed from `watchgod` to `watchfiles`, this files refers to the
`watchfiles` package.

Documentation for the old version (`watchgod`) is available [here](https://github.com/samuelcolvin/watchfiles/tree/watchgod).
See [issue #102](https://github.com/samuelcolvin/watchfiles/issues/102) for details on the migration and its rationale.

---

Underlying file system notifications are handled by the [Notify](https://github.com/notify-rs/notify) rust library.

## Usage

Here are some examples of what **watchfiles** can do:

```py
title="watch Usage"
from watchfiles import watch

for changes in watch('./path/to/dir'):
    print(changes)
```
See [`watch` docs](./api/watch.md#watchfiles.watch) for more details.

```py
title="awatch Usage"
import asyncio
from watchfiles import awatch

async def main():
    async for changes in awatch('/path/to/dir'):
        print(changes)

asyncio.run(main())
```
See [`awatch` docs](./api/watch.md#watchfiles.awatch) for more details.

```py
title="run_process Usage"
from watchfiles import run_process

def foobar(a, b, c):
    ...

if __name__ == '__main__':
    run_process('./path/to/dir', target=foobar, args=(1, 2, 3))
```
See [`run_process` docs](./api/run_process.md#watchfiles.run_process) for more details.

```py
title="arun_process Usage"
import asyncio
from watchfiles import arun_process

def foobar(a, b, c):
    ...

async def main():
    await arun_process('./path/to/dir', target=foobar, args=(1, 2, 3))

if __name__ == '__main__':
    asyncio.run(main())
```
See [`arun_process` docs](./api/run_process.md#watchfiles.arun_process) for more details.

## Installation

**watchfiles** requires Python 3.7 - 3.10.

```bash
pip install watchfiles
```

Binaries are available for:

* **Linux**: `manylinux-x86_64`, `musllinux-x86_64` & `manylinux-i686`
* **MacOS**: `x86_64` & `arm64` (except python 3.7)
* **Windows**: `amd64` & `win32`

Otherwise, you can install from source which requires Rust stable to be installed.

## How Watchfiles Works

*watchfiles* is based on the [Notify](https://github.com/notify-rs/notify) rust library.

All the hard work of integrating with the OS's file system events notifications and falling back to polling is palmed
off on the rust library.

"Debouncing" changes - e.g. grouping changes into batches rather than firing a yield/reload for each file changed
is managed in rust.

The rust code takes care of creating a new thread to watch for file changes so in the case of the synchronous methods
(`watch` and `run_process`) no threading logic is required in python. When using the asynchronous methods (`awatch` and
`arun_process`) [`anyio.to_thread.run_sync`](https://anyio.readthedocs.io/en/stable/api.html#anyio.to_thread.run_sync)
is used to wait for changes in rust within a thread.