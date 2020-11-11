# electron-vue-next

This repository contains the starter template for using vue-next with the latest electron. 

*I started to learn electron & vue by the great project [electron-vue](https://github.com/SimulatedGREG/electron-vue). This project is also inspired from it.*

## Features

- Electron 10
  - Follow the [security](https://www.electronjs.org/docs/tutorial/security) guide of electron, make renderer process a browser only environment
  - Using [electron-builder](https://github.com/electron-userland/electron-builder) to build
- Empower [vue-next](https://github.com/vuejs/vue-next) and its eco-system
  - Using [vite](https://github.com/vitejs/vite) which means develop renderer process can be blazingly fast!
  - Using [vuex 4.0](https://github.com/vuejs/vuex/tree/4.0) with strong type state, getters, and commit
  - Using [vue-router-next](https://github.com/vuejs/vue-router-next)
- Using [eslint](https://www.npmjs.com/package/eslint) with Javascript Standard by default
- Built-in TypeScript Support
  - Using [esbuild](https://github.com/evanw/esbuild) in [rollup](https://github.com/rollup/rollup) (align with vite) to build main process typescript code 
- Github Action with Github Release is out-of-box
  - Auto bump version in package.json and generate CHANGELOG.md if you follow the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0)
  - Detail how this work described in [Release Process](#release-process) section
- Integrate VSCode well
  - Support debug .ts/.vue files in main/renderer process by vscode debugger
  - Detail see [Debug](#debugging) section

## Quick Start

Clone or fork this project to start.
Once you have your project, and in the project folder:

```shell
# Install dependencies
npm install

# Will start vite server, rollup devserver, and electron to dev!
npm run dev

# OPTIONAL. Will compile the main and renderer process to javascript and display output size
npm run build

# OPTIONAL. Will compile all and output an unpacked electron app. You can directly 
npm run build:dir

# Will compile all and build all products and ready to release
npm run build:production

```

### Config Your Project and Build

Once you install your project, you should change the package base info in [package.json](/package.json),
and also the build information in [build.base.config.js](/scripts/build.base.config.js).

## Project Structure

### File Tree

Your workspace should looks like

```
your-project
├─ scripts                 all dev scripts, build script directory
├─ build                   build resource and output directory
│  └─ icons/               build icon directory
├─ dist
│  └─ electron/            compiled output directory
├─ src
│  ├─ main
│  │  ├─ dialog.ts         the ipc handler to support dialog API from renderer process
│  │  ├─ global.ts         typescript global definition
│  │  ├─ index.dev.ts      the development rollup entry
│  │  ├─ index.prod.ts     the production rollup entry
│  │  ├─ index.ts          real electron start-up entry file
│  │  ├─ logger.ts         a simple logger implementation
│  │  └─ staticStore.ts
│  ├─ renderer
│  │  ├─ components/       assets directoy
│  │  ├─ components/       all components
│  │  ├─ router.ts         vue-router initializer
│  │  ├─ store.ts          vuex store initializer
│  │  ├─ App.vue           entry vue file imported by index.ts
│  │  ├─ index.css         entry css file for vite
│  │  ├─ index.html        entry html file for vite
│  │  └─ index.ts          entry script file for vite
│  └─ shared               shared folder can be access from both main and renderer side
│     ├─ store/            vuex store definition
│     └─ sharedLib.ts      an example file that can be access from both side
├─ static/                 static resource directory
├─ .eslintrc.js
├─ .gitignore
├─ package.json
└─ README.md
```

#### assets, static resources, build resources... what's the difference?

The assets is only used by the renderer process (in-browser display), like picture or font. They are **bundled by vite/rollup**. You can directly `import` them in `.vue/.ts` files under renderer directory. The default assets are in [renderer/renderer/assets](src/renderer/assets)

The static resources are the static files which main process wants to access (like read file content) in **runtime vie file system**. They might be the tray icon file, browser window icon file. The static folder is at [static](static).

The build resources are used by `electron-builder` to build the installer. They can be your program icon of installer, or installer script. Default build icons are under [build/icons](build/icons).

*Notice that your program icon can show up in multiple place! Don't mixup them!*
- *In build icons, of course you want your program has correct icon.*
- *In static directory, sometime you want your program has **tray** which require icon in static directory.*
- *In assets, sometime you want to display your program icon inside a page. You need to place them in the assets!*

### Main and Renderer Processes

Quote from electron official document about [main and renderer processes](https://www.electronjs.org/docs/tutorial/quick-start#main-and-renderer-processes). The main process is about

> - The Main process creates web pages by creating BrowserWindow instances. Each BrowserWindow instance runs the web page in its Renderer process. When a BrowserWindow instance is destroyed, the corresponding Renderer process gets terminated as well.
> - The Main process manages all web pages and their corresponding Renderer processes.

And the renderer process is about

> - The Renderer process manages only the corresponding web page. A crash in one Renderer process does not affect other Renderer processes.
> - The Renderer process communicates with the Main process via IPC to perform GUI operations in a web page. Calling native GUI-related APIs from the Renderer process directly is restricted due to security concerns and potential resource leakage.

Commonly, the main process is about your core business logic, and renderer side act as a data consumer to render the UI.

Following the [security](https://www.electronjs.org/docs/tutorial/security) guideline of electron, in this boilerplate, the renderer process [**does not** have access to nodejs module by default](https://www.electronjs.org/docs/tutorial/security#2-do-not-enable-nodejs-integration-for-remote-content). The electron provide the `preload` options in `webPreferences`. In this boilerplate, I suggest you to wrap your core logic into `Service`. 

The `Service` is a type of class defined under the `src/main/services`. All the public method can be access by the renderer process.
It's the bridge between the main and renderer. You can look at [Service](#service) for the detail.

### NPM Scripts

#### `npm run dev`

Start the vite dev server hosting the renderer webpage with hot reloading.
Start the rollup server hosting the main process script. It will auto reload the electron app if you modify the source files.

#### `npm run build`

Compile both `main` and `renderer` process code to production, located at `dist/electron`

#### `npm run build:production`

It will compile both processes, and then run `electron-builder` to build your app into executable installer or zip. The build config is defined in [scripts/build.base.config.js](scripts/build.base.config.js).

#### `npm run build:dir`

It will compile both processes, and it will run `electron-builder` to build only the directoy version of the production electron app, which for example, for windows x64, it's located at `build/win-unpacked`.

This will much faster than `npm run build:production`. So you can use it to quick testing the production app.

#### `npm run lint`

Run eslint to report eslint error.

#### `npm run lint:fix`

Run eslint to fix and report eslint error.

## Development

Due to the project is following the [security](https://www.electronjs.org/docs/tutorial/security) guideline. It does not allow the renderer to access node by default. The [Service](#service) is a simple solution to isolate renderer logic and the vulnerable logic with full nodejs module access. See [this](#option-using-node-modules-in-renderer-process) section if you want to directly use node modules in renderer process.

### Service

A Service lives in a class in `src/main/services`. It should contain some of your core logic with file or network access in main process. It exposes these logic to renderer process. You call the hook `useService('NameOfService')` to use it in renderer side.

The concept of service is totally optional. This is a design for security. ***If you think this is redundent and not fit with your program design, you can just remove it.***

#### Create a new Service

Add a file to the `/src/main/services` named `BarService.ts`

```ts
export default class BarService extends Service {
  async doSomeCoreLogic() {
    // perform some file system or network work here
  }
}
```

And you need to add it to the `interface Services` in `src/main/services/index.ts`.

```ts
import { BarService } from './BarService'

export interface Services {
  // ... other existed services
  BarService: BarService
}
```

Then, add it to the `initializeServices` in `src/main/index.ts`

```ts
async function initializeServices(logger: Logger) {
  initialize({
    // ...other services
    BarService: new BarService(logger)
  })
}
```

And this is ready to be used in renderer process by `useService('BarService')`. See [Using Service in Renderer](#using-service-in-renderer).

##### Using Other Service in a Service

If you need to use other `Service`, like `FooService`. You need to `@Inject` decorator to inject during runtime.

```ts
export default class BarService extends Service {
  @Inject('FooService')
  private fooService: FooService

  async doSomeCoreLogic() {
    const result = await fooService.foo()
    // perform some file system or network operations here
  }
}
```

#### Using Service in Renderer

You can directly access all the async methods in a service class by `useService('nameOfService')`

Here is an example in [About.vue](), using the `BaseService`.

```vue
<template>
  <div>
    <img alt="Vue logo" src="../assets/logo.png" />
    <div>Electron Version: {{ version }} </div>
    <div>Appdata Path: {{ path }} </div>
    <div>Running Platform: {{ platform }} </div>
  </div>
</template>

<script lang=ts>
import { defineComponent, reactive, toRefs } from 'vue'
import { useService } from '../hooks'

export default defineComponent({
  setup() {
    const { getBasicInformation } = useService('BaseService')
    const data = reactive({
      version: '',
      path: '',
      platform: ''
    })
    getBasicInformation().then(({ version, platform, root }) => {
      data.version = version
      data.path = root
      data.platform = platform
    })
    return {
      ...toRefs(data)
    }
  }
})
</script>
```

#### Remove Service Infra

If you don't like Service design, you just easily remove it by

1. Remove the whole `src/main/services` directory
2. Remove the import line `import { initialize } from './services'` and initialization line `initialize(logger)` in `src/main/index.ts`

### Hooks or Composable in Renderer Process

One great feature of vue 3 is the [composition-api](https://composition-api.vuejs.org/). You can write up some basic piece of logic and compose them up during the setup functions. Currently, these `hooks` are placed in `/src/renderer/hooks` by default.

Take the example from vue composition api site, you have such code in `/src/renderer/hooks/mouse.ts`

```ts
import { ref, onMounted, onUnmounted } from 'vue'

export function useMousePosition() {
  const x = ref(0)
  const y = ref(0)

  function update(e) {
    x.value = e.pageX
    y.value = e.pageY
  }

  onMounted(() => {
    window.addEventListener('mousemove', update)
  })

  onUnmounted(() => {
    window.removeEventListener('mousemove', update)
  })

  return { x, y }
}
```

You'd better to export this `mouse.ts` to `/src/renderer/hooks/index.ts`

```ts
// other exports...

export * from './mouse.ts'
```

Then in the `vue` file you can import all hooks by the alias path

```vue
<template>
  ...template content
</template>
<script lang=ts>
import { defineComponent } from 'vue'
import { useMousePosition } from '/@/hooks'

export default defineComponent({
  setup() {
    const { x, y } = useMousePosition()
    // other logic
    return { x, y }
  }
})
</script>
```

### Electron API in Renderer Process

The boilplate exposes several electron APIs by default. You can access them by `useShell`, `useClipboard`, `useIpc` and `useDialog`.
*These are provided by `static/preload.js` script. If you remove the preload during the creation of this BrowserWindow, this won't work.*

```ts
import { defineComponent } from 'vue'
import { useShell } from '/@/hooks'

export default defineComponent({
  setup() {
    const shell = useShell() // this is equivalence to the import { shell } from 'electron' normally
    // the shell object type definition works normally
  }
})
```

The only exception is the `useDialog`. You can only use `async` functions in it as the API call goes through IPC and it must be `async`.

### Dependencies Management

If you adding a new dependency, make sure if it's using any **nodejs** module, add it as `excludeOptimize` in the `package.json`. Otherwise, the vite will complain about "I cannot handle it!".

```json
{
  // ...other package.json content
  "dependencies": {
    // ...other dependencies
    "a-nodejs-package": "<version>"
  },
  "excludeOptimize": [
    // ...other existed excluded packages
    "a-nodejs-package" // your new package
  ],
  // ...rest of package.json
}
```

The raw javascript dependencies are okay for vite.

#### Native Dependencies

If you want to use the native dependencies, which need to compile when install. Usually, you need [node-gyp](https://github.com/nodejs/node-gyp) to build, the `electron-builder` will rebuild it upon your electron for you. Normally you don't need to worry much. Notice that if you are in Windows, you might want to install [windows-build-tools](https://github.com/felixrieseberg/windows-build-tools) to install the compile toolchain.

#### Dependencies Contains Compiled Binary

If you want to use the dependencies containing the compiled binary, not only you should adding it to vite `exclude`, you should also take care about the electron-builder config. See the [Build](#build-exclude-files) section for detail. The development process won't affect much by it.

### Debugging

This is really simple. In vscode debug section, you will see three profiles: 

1. Electron: Main (attach)
2. Electron: Renderer (attach)
3. Electron: Main & Renderer (attach)


```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Electron: Main (attach)",
            "type": "node",
            "request": "attach",
            "cwd": "${workspaceFolder}",
            "outFiles": [
                "${workspaceFolder}/dist/**/*.js"
            ],
            "smartStep": true,
            "sourceMaps": true,
            "protocol": "inspector",
            "port": 5858,
            "timeout": 20000
        },
        {
            "name": "Electron: Renderer (attach)",
            "type": "chrome",
            "request": "attach",
            "port": 9222,
            "webRoot": "${workspaceFolder}",
            "timeout": 15000
        },
    ],
    "compounds": [
        {
            "name": "Electron: Main & Renderer (attach)",
            "configurations": ["Electron: Main (attach)", "Electron: Renderer (attach)"]
        }
    ]
}
```

The name should be clear. The first one attach to main and the second one attach to renderer (required vscode chrome debug extension).
The third one is run the 1 and 2 at the same time.

You should first run `npm run dev` and start debugging by the vscode debug.

### Option: Using Node Modules in Renderer Process

By default, the renderer process environment is just a raw front-end environment. You cannot use any nodejs module here. (Use service alternative)

If you just want to use node modules in electron renderer/browser side anyway, you can just enable the `nodeIntegration` in BrowserWindow creation.

For example, you can enable the main window node integration like this:

```ts
const mainWindow = new BrowserWindow({
  height: 600,
  width: 800,
  webPreferences: {
    preload: join(__static, 'preload.js'),
    nodeIntegration: true // adding this to enable the node integration
  }
})
```

## Build

The project build is based on [electron-builder](https://github.com/electron-userland/electron-builder). The config file is majorly in [scripts/build.base.config.js](../scripts/build.base.config.js). And you can refer the electron-builder [document](https://www.electron.build/).

### Compile Process

The project will compile typescript/vue source code by rollup into javascript production code. The rollup config for main process is in [rollup.config.js](https://github.com/ci010/electron-vue-next/tree/master/scripts/rollup.config.js). It will output the production code to `dist/electron/index.prod.js`.

The config to compile renderer process is in [vite.config.js](https://github.com/ci010/electron-vue-next/tree/master/scripts/vite.config.js). It will compile the production code into `dist/electron/renderer/*`.

### Build Exclude Files

Normally, once you correctly config the `dependencies` in [Development](#development) section, you should not worry to much about the build. But some dependencies contains compiled binary. You might want to exclude them out of the unrelated OS builds.

For example, [7zip-min](https://github.com/onikienko/7zip-min):

Since it using the `7zip-bin` which carry binary for multiple platform, we need to correctly include them in config.
Modify the electron-builder build script `build.base.config.js`

```js
  files: [
    "dist/electron/**/*",
    "!**/node_modules/**/*",
    "node_modules/7zip-bin/**/*"
  ],
  asarUnpack: [
    "node_modules/7zip-bin/**/*"
  ],
```

Add them to `files` and `asarUnpack` to ensure the electron builder correctly pack & unpack them.

To optimize for multi-platform, you should also exclude them from `files` of each platform config `build.config.js`

```js
  mac: {
    // ... other mac configs
    files: [
      "node_modules/7zip-bin/**/*",
      "!node_modules/7zip-bin/linux/**",
      "!node_modules/7zip-bin/win/**"
    ]
  },
  win: {
    // ... other win configs
    files: [
      "node_modules/7zip-bin/**/*",
      "!node_modules/7zip-bin/linux/**",
      "!node_modules/7zip-bin/mac/**"
    ]
  },
  linux: {
    // ... other linux configs
    files: [
      "node_modules/7zip-bin/**/*",
      "!node_modules/7zip-bin/win/**",
      "!node_modules/7zip-bin/mac/**"
    ]
  },
```

## Release

The out-of-box github action will validate each your PR by eslint and run `npm run build`. It will not trigger electron-builder to build production assets.

For each push in master branch, it will build production assets for win/mac/linux platform and upload it as github action assets. It will also create a **pull request** to asking you to bump version and update the changelog. 

It using the conventional-commit. If you want to auto-generate the changelog, you should follow the [conventional commit guideline](https://www.conventionalcommits.org/en/v1.0.0).

If the **bump version PR** is approved and merged to master, it will auto build and release to github release.

**If you want to disable this github action release process, just remove the [.github/workflows/build.yml](/.github/workflows/build.yml) file.**

### AutoUpdate Support

This boilerplate include the [electron-updater](https://github.com/electron-userland/electron-builder/tree/master/packages/electron-updater) as dependencies by default. You can follow the [electron-builder](https://github.com/electron-userland/electron-builder) guideline to implement the autoUpdate process.