A small project to demonstrate the current limitations of typescript with node20 and module package.

## How to reproduce

```shell
$ grep "\"type\":" package.json # Check that the project is a module
  "type": "module",
$ node --version
v18.15.0
$ npm install
# [...]
$ npm run build
# [...]
$ node build/index.js 
node:internal/errors:490
    ErrorCaptureStackTrace(err);
    ^

Error [ERR_MODULE_NOT_FOUND]: Cannot find module '/tmp/abc/build/utils' imported from /tmp/abc/build/index.js
    at new NodeError (node:internal/errors:399:5)
    at finalizeResolution (node:internal/modules/esm/resolve:326:11)
    at moduleResolve (node:internal/modules/esm/resolve:945:10)
    at defaultResolve (node:internal/modules/esm/resolve:1153:11)
    at nextResolve (node:internal/modules/esm/loader:163:28)
    at ESMLoader.resolve (node:internal/modules/esm/loader:838:30)
    at ESMLoader.getModuleJob (node:internal/modules/esm/loader:424:18)
    at ModuleWrap.<anonymous> (node:internal/modules/esm/module_job:77:40)
    at link (node:internal/modules/esm/module_job:76:36) {
  code: 'ERR_MODULE_NOT_FOUND'
}

Node.js v18.15.0
```

## How to fix

### Node18

Use `--es-module-specifier-resolution=node` flag

### Node20

#### Option 1: Loader

Use a custom loader such as https://www.npmjs.com/package/extensionless to resolve the module path.

Cons: add a dependency to the project + runtime overhead

#### Option 2: Use `import ... from '*.js'`

Specify the full path of the resulting build in the TypeScript import statement.
```diff,javascript
- import { add } from './utils'
+ import { add } from './utils.js'
```

Cons: This is not natural as the "*.js" file does not exist in the source code.
Cons: `import/no-unresolved` eslint rule will complain.

### Option 3: Use tsc-esm / tsc-alias / babel

The above tools can resolve and translate the paths to the correct file.

Cons: add a dependency to the project

### Option 4: Update tsconfig.json

I was not able to find any suitable option to allow this behavior in the tsconfig.json file.

In my opinion there should be an option to resolve the module path with the correct file extension.

In fact, TypeScript already does the job to resolve the path, but it just does not output it.
