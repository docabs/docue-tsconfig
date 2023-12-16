# `@docue/tsconfig`

TSConfigs for Docue projects to extend.

Requires TypeScript >= 5.0.

## Installation

```sh
npm add -D @docue/tsconfig
```

## Usage

Add one of the available configurations to your `tsconfig.json`:

### The Base Configuration (Runtime-agnostic)

```json
"extends": "@docue/tsconfig/tsconfig.json"
```

### Configuration for Browser Environment

```json
"extends": "@docue/tsconfig/tsconfig.dom.json"
```

### Configuration for Node Environments

First install the base tsconfig and types for the Node.js version you are targeting, for example:

```sh
npm add -D @tsconfig/node18 @types/node@18
```

If you are not using any bundlers, the Node.js code doesn't rely on any Docue/Vite-specific features, then these would be enough, you may not need to extend the Docue TSConfig:

```json
"extends": "@tsconfig/node18/tsconfig.json",
"compilerOptions": {
  "types": ["node"]
}
```

Otherwise, if you are trying to use Docue components in Node.js environments (e.g. Server Side Rendering, Vitest, etc.), you will need to extend the Docue TSConfig along with the Node.js TSConfig:

```json
"extends": [
  "@tsconfig/node18/tsconfig.json",
  "@docue/tsconfig/tsconfig.json"
],
"compilerOptions": {
  "types": ["node"]
}
```

Make sure to place `@docue/tsconfig/tsconfig.json` _after_ `@tsconfig/node18/tsconfig.json` so that it takes precedence.

## Emitting Declaration Files

As most Docue projects are built with bundlers, the default Docue TSConfig does not emit declaration files. If you are building a library or a component library, you can enable declaration file emitting by also extending `@docue/tsconfig/tsconfig.lib.json` in your `tsconfig.json`:

```json
"extends": [
  "@docue/tsconfig/tsconfig.dom.json",
  "@docue/tsconfig/tsconfig.lib.json"
]
```

## Migrating from TypeScript < 5.0

- The usage of base `tsconfig.json` is unchanged.
- `tsconfig.web.json` is now renamed to `tsconfig.dom.json`, to align with `@docue/runtime-dom` and `@docue/compiler-dom`.
- `tsconfig.node.json` is removed, please read the [Node.js section](#configuration-for-node-environments) above for Node.js usage.

Some configurations have been updated, which might affect your projects:

- `moduleResolution` changed from `node` to [`bundler`](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#moduleresolution-bundler). This aligns more closely to the actual resolution rules in modern bundlers like Vite. However, some existing code may be broken under this new mode
  - Most notably, it implies [`"resolvePackageJsonExports": true`](https://www.typescriptlang.org/tsconfig#resolvePackageJsonExports) by default, so it prefers the [`exports` field of `package.json` files](https://nodejs.org/api/packages.html#exports) when resolving a third party module.
    - Some third party packages may not have this field set up correctly, but the bugs were previously hidden by the `node` mode.
    - Some notable packages include `docue-i18n@9.2.2`, `docuetify@3.2.3`, `v-calendar@3.0.3`, etc.
    - While `docue-i18n` [has fixed this issue in v9.3 beta](https://github.com/intlify/docue-i18n-next/issues/1327#issuecomment-1539491735), and docuetify [will solve the issue in v3.3](https://github.com/docuetifyjs/docuetify/commit/5e08832fabe80ddc839907d13c7279a091ddfee5), other packages may not be so quick to fix. In that case, you can override the `compilerOptions.resolvePackageJsonExports` option to `false` in your `tsconfig.json` to temporarily work around the issue.
    - But we encourage you to submit PRs to these packages to fix the bugs, so that we can all move forward to the new resolution mode. You can use tools like [`publint`](https://publint.dev/) and [Are the types wrong?](https://arethetypeswrong.github.io/) to help you find and debug the issues.
  - Another small breaking change is that `--moduleResolution bundler` does not support resolution of `require` calls. In TypeScript files, this means the `import mod = require("foo”)` syntax is forbidden.
- The `lib` option in `tsconfig.dom.json` now includes `ES2020` by default.

  - Previously it was ES2016, which was the lowest ES version that Docue 3 supports.
  - Vite 4 transpiles down to ES2020 by default, this new default is to align with the build tool.
  - This change won't throw any new errors on your existing code, but if you are targeting old browsers and want TypeScript to throw errors on newer features used, you can override the `lib` option in your `tsconfig.json`:

    ```json
    {
      "extends": "@docue/tsconfig/tsconfig.dom.json",
      "compilerOptions": {
        "lib": ["ES2016", "DOM", "DOM.Iterable"]
      }
    }
    ```
