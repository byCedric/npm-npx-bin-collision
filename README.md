# npx binary collisions

It seems that `npx <package>` overwrites, and executes, binaries from packages that register binaries with the same name.

## ✅ Test case 1

Packages used in this test, both registering a binary `@bycedric/test-bin-package-a`:

- [`@bycedric/test-bin-package-a@1.0.0`](https://unpkg.com/browse/@bycedric/test-bin-package-a@1.0.0/) -> `"bin": "./bin.js"`
  - [`@bycedric/test-bin-package-b@1.0.0`](https://unpkg.com/browse/@bycedric/test-bin-package-b@1.0.0/) -> `"bin": { "@bycedric/test-bin-package-a": "./bin.js" }`

Running `npx @bycedric/test-bin-package-a@1.0.0` yields `You are running the binary of "package-a"`, which is correct.

## ❌ Test case 2

Packages used in this test, both registering a binary `bycedric-test-bin-package-a`:

- [`bycedric-test-bin-package-a@1.1.0`](https://unpkg.com/browse/bycedric-test-bin-package-a@1.1.0/) -> `"bin": "./bin.js"`
  - [`@bycedric/test-bin-package-b@1.1.0`](https://unpkg.com/browse/@bycedric/test-bin-package-b@1.1.0/) -> `"bin": { "bycedric-test-bin-package-a": "./bin.js" }`

Running `npx bycedric-test-bin-package-a@1.1.0` yields `You are running the binary of "package-b"`, which is **incorrect**.

## ❌ Test case 3

Packages used in this test, all registering a binary `bycedric-test-bin-package-a`:

- [`bycedric-test-bin-package-a@2.0.0`](https://unpkg.com/browse/bycedric-test-bin-package-a@2.0.0/) -> `"bin": "./bin.js"`
  - [`@bycedric/test-bin-package-b@2.0.0`](https://unpkg.com/browse/@bycedric/test-bin-package-b@2.0.0/) -> `"bin": { "bycedric-test-bin-package-a": "./bin.js" }`
    - [`@bycedric/1test-bin-package-c@2.0.0`](https://unpkg.com/browse/@bycedric/1test-bin-package-c@2.0.0/) -> `"bin": { "bycedric-test-bin-package-a": "./bin.js" }`

Running `npx bycedric-test-bin-package-a@2.0.0` yields `You are running the binary of "package-c"`, which is **incorrect**.

## Interpretation

This behavior allows 3rd party libraries to fully overwrite the binary, which gets executed through `npx`. It can be used during a supply chain attack to execute arbitrary code.

> The only requirement is that they are within the dependency chain of the original library.
