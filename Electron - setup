Build Electron App
---

Electron使开发人员能够使用Web前端技术栈构建跨平台桌面应用程序。通常我们使用Vue + Pinia + Vuetify构建漂亮的网页，而现在我们可以使用相同的代码构建桌面应用程序。

编写代码很容易，但构建软件包可能会很繁琐。过去，构建Electron应用程序需要使用不同的工具：

- [electron-builder](https://github.com/electron-userland/electron-builder)，这是构建Electron应用程序的第一个官方工具，仍在维护中。但它只构建纯Electron应用程序，不与任何其他框架集成，例如Vue、React。因此，仅使用electron-builder构建带有前端UI框架的应用程序可能会很痛苦。
- [vue-cli-plugin-electron-builder](https://github.com/nklayman/vue-cli-plugin-electron-builder)，这是一个vue-cli插件，内部使用electron-builder以启用使用Vue框架构建Electron应用程序。但由于vue-cli不再维护（已被vite替代为官方的Vue项目构建工具），不建议再使用vue-cli-plugin-electron-builder。
- [electron forge](https://github.com/electron/forge)，这是最新的官方构建工具，是几个其他工具的组合，使构建软件包变得更加容易。但与electron-builder一样，它仅打包纯Electron应用程序，难以与其他前端框架一起使用。
然后，我们使用Electron-vite来构建最新的Electron应用程序，同时使用最新的Vue以及其他支持工具，如Vuetify和Pinia。

### 1. 初始化

> 使用Vite和Electron-vite构建项目，需要Node的版本为v20+

```comandline
npm create @quick-start/electron
```
这个命令会完成electron项目的初始化，你可以在初始化的时候选择使用vue作为前端开发框架，然后：

#### 1.1 安装依赖
```commandline
npm install
```

#### 1.2 本地运行
```commandline
npm run dev
```

#### 1.3 给不同的系统打包可执行文件
```commandline
npm run build:mac  # for mac
npm run build:win  # for windows
npm run build:linux   # for linux
```
打包出来的可执行文件在`dist`文件夹下面。


### 2. 添加`Pinia`和`Vuetify`

上面的初始化步骤只会生成包含Vue的Electron项目配置，我们可以进一步的添加pinia和vuetify来完成前端构建：

> 默认情况下electron-vite创建的main.js文件位于`src/renderer/src/main.js`。

```commandline
npm install pinia vuetify
```
然后更新`src/renderer/src/main.js`文件：
```javascript
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

// Vuetify
import 'vuetify/styles'
import { createVuetify } from 'vuetify'
import * as components from 'vuetify/components'
import * as directives from 'vuetify/directives'

const vuetify = createVuetify({
  components,
  directives,
})

const pinia = createPinia()

createApp(App).use(pinia).use(vuetify).mount('#app')

```
### 3. Happy Codeing

之后就可以按照前端开发的方式来开发跨系统的桌面应用程序了。
