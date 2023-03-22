<p align="center">
<img src="https://user-images.githubusercontent.com/40021217/226882143-361c2697-b793-43ea-9a64-65eadb031466.png" width="800" style="border-radius: 5px"/>
</p>

<h2 align="center">
<a href="https://github.com/kingyue737/vitify-next">Vitify</a> with Electron
</h2><br>

<pre align="center">
🖖 Vite + Vuetify 3 + Electron ⚛️
Opinionated Starter Template
</pre>

## Features

- ⚛️ [Electron](https://github.com/electron/electron) - Build cross-platform desktop apps with JavaScript, HTML, and CSS

- ⚡️ [Vite 4](https://github.com/vitejs/vite), [pnpm](https://pnpm.io/), [ESBuild](https://github.com/evanw/esbuild) - born with fastness

- 🗂️ [File based routing](./src/pages)

- 📑 [Layout system](./src/layouts)

- 🍍 [State Management via Pinia](https://pinia.vuejs.org/)

- 📥 [APIs auto importing](https://github.com/antfu/unplugin-auto-import) - use Composition API and others directly

- 🧪 Unit/Component Testing with [Vitest](https://github.com/vitest-dev/vitest) + [Testing Library](https://github.com/testing-library/vue-testing-library)

### App Starter Template

- 🪟 Default layout with drawer, header(title bar) and footer (status bar)

- 🧭 Auto generated navigation drawer and breadcrumbs based on routes

- 🔔 Notification store

- 📉 Data visualization with [vue-echarts](https://github.com/ecomfe/vue-echarts)

- 🎨 Theme color customization and dark mode

- 📱 Responsive layout

## Usage

### Development

Start a standalone [Vue devtools](https://github.com/vuejs/devtools) and launch Vite dev server

```bash
pnpm dev
```

### Build

To build the App, run

```bash
pnpm build
```

And you will see the generated executable and `setup.exe` in `release` folder.
