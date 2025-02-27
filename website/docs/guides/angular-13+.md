---
id: angular-13+
title: Angular >=13
---

**Angular 13** introduces ESM package format for Angular packages. `jest-preset-angular`
currently supports testing with Jest in `CommonJS` mode with **Angular 13** using [default preset](../getting-started/presets.md).
Jest ESM support with **Angular 13** is new and may have issues.

:::important

With Jest 28 and `jest-preset-angular` **v12.0.0**, `ng-jest-resolver` is no longer required to have in Jest config. This
resolver is also excluded from our default and default ESM presets.

:::

Starting from **v11.0.0**, `jest-preset-angular` introduces a few extra changes to be able to run Jest with **Angular 13**:

- `moduleFileExtensions` is updated to include `mjs` files as accepted module format.

- `transformIgnorePatterns` is added to inform Jest to transform `.mjs` files.

- `transform` is updated to include `.mjs` extension to transform to `CommonJS` codes.

## Migration steps from Angular < 13

- Upgrade the project to **Angular 13** following https://update.angular.io/

- If one is using the default preset as following:

```js
// jest.config.js
module.exports = {
  preset: 'jest-preset-angular',
};
```

there are no migration steps required

### Using ES Modules

ES Modules support is new and may encounter issues. See [example-app-v13](https://github.com/thymikee/jest-preset-angular/tree/main/examples/example-app-v13) for an example with tests that run using ESM, and using ESM + isolated.

Your `jest.config.js` should be changed to something like:

```js
const { pathsToModuleNameMapper } = require('ts-jest/utils');
const { paths } = require('./tsconfig.json').compilerOptions;

/** @type {import('ts-jest/dist/types').InitialOptionsTsJest} */
module.exports = {
  preset: 'jest-preset-angular/presets/defaults-esm',
  globals: {
    'ts-jest': {
      useESM: true,
      stringifyContentPathRegex: '\\.(html|svg)$',
      tsconfig: '<rootDir>/tsconfig-esm.spec.json',
    },
  },
  moduleNameMapper: {
    ...pathsToModuleNameMapper(paths, { prefix: '<rootDir>' }),
    tslib: 'tslib/tslib.es6.js',
  },
  setupFilesAfterEnv: ['<rootDir>/setup-jest.ts'],
};
```

Before upgrading to ng13 and switching to ES Modules, your `setup-jest.ts` file most likely uses the preset `setup-jest`, like the following:

```ts
// setup-jest.ts
import 'jest-preset-angular/setup-jest';
import './jest-global-mocks';
```

The `jest-preset-angular/setup-jest` file doesn't work with ESM, because it uses `require`. For now you should skip using this file, and replace the contents of your `setup-jest.ts` with:

```ts
// setup-jest.ts
import 'zone.js/fesm2015/zone-testing-bundle.min.js';
import './jest-global-mocks';

import { getTestBed } from '@angular/core/testing';
import { BrowserDynamicTestingModule, platformBrowserDynamicTesting } from '@angular/platform-browser-dynamic/testing';

getTestBed().initTestEnvironment(BrowserDynamicTestingModule, platformBrowserDynamicTesting(), {
  teardown: {
    destroyAfterEach: true, // Angular recommends this, but it may break existing tests
  },
});
getTestBed().initTestEnvironment(BrowserDynamicTestingModule, platformBrowserDynamicTesting());
```

## Potential issues with Angular 13 ESM package format and workaround

### `Cannot find modules` error when importing any deep paths from Angular ESM format packages

```
Cannot find module '@angular/common/locales/xx' from 'src/app/app.component.spec.ts'
```

To fix this issue, one needs to add `mjs` to `moduleFileExtensions` as following

```js
// jest.config.js
module.exports = {
  // ...other options
  moduleFileExtensions: ['ts', 'html', 'js', 'json', 'mjs'],
};
```

### Usage with Angular libraries which are built with Angular CLI 13

Besides, the changes in Angular packages themselves, **Angular** libraries which are built with **Angular CLI 13** also introduce
ESM package format. Similar to Angular packages, Jest doesn't understand `.mjs` files which are in these new format
libraries in Jest **CommonJS** mode.

To fix this issue, one should modify `transformIgnorePatterns` to be as following:

```js
// jest.config.js
module.exports = {
  // ...other options
  transformIgnorePatterns: ['node_modules/(?!.*\\.mjs$)'],
};
```
