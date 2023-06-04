---
title: Quasar 语言包
desc: 如何配置 Quasar 的语言包。
related:
  - /options/rtl-support
  - /options/app-internationalization
---
Quasar 的语言包是针对与实现 Quasar 内置组件的多语言国际化的，因为部分组件上会有一些文案。

::: warning
需要注意的是，下述方案是针对 Quasar 内置组件的多语言国际化，如果您想实现自己的组件的多语言国际化，请参考 [App Internationalization](/options/app-internationalization) 页面。
:::
如上所述，部分组件上会有一些文案，当需要实现多语言国际化的时候，一种方案是为每个组件都配置多分不同语言的文案，但是这太耗时间，也太麻烦，您可以选择使用 quasra 语言包，其中内置了一定数量的标准字段，类似 "Cancel", "Clear", "Select", "Update"等等，就不再需要重复的翻译他们。

::: tip
完整的 Quasar 语言包列表见[Quasar Languages on GitHub](https://github.com/quasarframework/quasar/tree/dev/ui/lang).
<br><br>**如果您在列表中没有发现想要的语言包**, 您可以提交一个 PR 来添加它，大概会花费 5 到 10 分钟，我们欢迎任何一种语言！
:::

## 配置默认的语言包
在默认情况下，Quasar 使用`en-US`英文语言包，您可以通过下述方式改变默认的语言包：

### 硬编码
如果默认语言包不需要动态切换，那么您可以通过下面的方式定义：

#### Quasar CLI
修改 `/quasar.config.js`:

```js
framework: {
  lang: 'zh-CN' //使用中文简体语言包
}
```

#### Quasar Vite Plugin
修改 `main.js`:

```js
// ...
import { Quasar } from 'quasar'
// ...
import langDe from 'quasar/lang/zh-CN'
// ...
app.use(Quasar, {
  // ...,
  lang: langDe
})
```

#### Vue CLI
修改 `main.js`:

```js
// ...
import { Quasar } from 'quasar'
// ...
import langDe from 'quasar/lang/zh-CN'
// ...
app.use(Quasar, {
  // ...,
  lang: langDe
})
```

#### Quasar UMD
引入要使用的语言包，例如：
```html
<!-- include this after Quasar JS tag -->
<script src="https://cdn.jsdelivr.net/npm/quasar@2/dist/lang/zh-CN.umd.prod.js"></script>
<script>
  Quasar.lang.set(Quasar.lang.zh-CN)
</script>
```

Check what tags you need to include in your HTML files on [UMD / Standalone](/start/umd) page.

### 动态配置 (非 SSR 模式)

Quasar CLI: 如果您的语言需要动态的选择， (例如通过 cookie 来判断),那么您需要创建一个 boot 文件来处理:

使用`quasar new boot quasar-lang-pack [--format ts]`命令会帮您创建一个`/src/boot/quasar-lang-pack.ts`文件，将下面的代码复制进去：

```js
// -- 使用 @quasar/app-vite --

import { Quasar } from 'quasar'

// 这个路径相对于您的 node_modules/quasar/..
const langList = import.meta.glob('../../node_modules/quasar/lang/*.mjs')
// or just a select few (example below with only DE and FR):
// import.meta.glob('../../node_modules/quasar/lang/(de|fr).mjs')

export default async () => {
  const langIso = 'zh-CN' // ... some logic to determine it (use Cookies Plugin?)

  try {
    langList[ `../../node_modules/quasar/lang/${ langIso }.mjs` ]().then(lang => {
      Quasar.lang.set(lang.default)
    })
  }
  catch (err) {
    // Requested Quasar Language Pack does not exist,
    // let's not break the app, so catching error
  }
}
```

```js
// -- 使用 @quasar/app-webpack --

import { Quasar } from 'quasar'

export default async () => {
  const langIso = 'zh-CN' //  ... 这个语言应是动态获取的，例如使用 Cookies Plugin?

  try {

    /* 注意下面这行注释 */
    await import(
      /* webpackInclude: /(zh-CN|en-US)\.js$/ */
      'quasar/lang/' + langIso
    ).then(lang => {
      Quasar.lang.set(lang.default)
    })
  }
  catch (err) {
    // Requested Quasar Language Pack does not exist,
    // let's not break the app, so catching error
  }
}
```

然后记得将这个 boot 文件注册到 `/quasar.config.js`文件中：

```js
boot: [
  'quasar-lang-pack'
]
```

::: warning Always constrain a dynamic import
注意示例代码中的注释，这样可以防止 webpack 将所有的语言包都打包，增大代码体积，详情请看： [Webpack magic comment](https://webpack.js.org/api/module-methods/#magic-comments) - `webpackInclude`。 See [Caveat for dynamic imports](https://quasar.dev/quasar-cli/lazy-loading#Caveat-for-dynamic-imports)
:::

### 动态配置 (SSR 模式)

在处理 SSR 时，我们不能使用上面的单例对象，因为这会污染 sessions。因此，与上面的非 SSR 模式动态示例相反，还必须从 boot 文件中指定 ssrContext

```js
// -- With @quasar/app-vite --

import { Quasar } from 'quasar'

// relative path to your node_modules/quasar/..
// change to YOUR path
const langList = import.meta.glob('../../node_modules/quasar/lang/*.mjs')
// or just a select few (example below with only DE and FR):
// import.meta.glob('../../node_modules/quasar/lang/(de|fr).mjs')

// ! 注意这个 ssrContext 参数:
export default async ({ ssrContext }) => {
  const langIso = 'de' // ... some logic to determine it (use Cookies Plugin?)

  try {
    langList[ `../../node_modules/quasar/lang/${ langIso }.mjs` ]().then(lang => {
      Quasar.lang.set(lang.default, ssrContext)
    })
  }
  catch (err) {
    // Requested Quasar Language Pack does not exist,
    // let's not break the app, so catching error
  }
}
```

```js
// -- With @quasar/app-webpack --

import { Quasar } from 'quasar'

// ! 注意这个 ssrContext 参数:
export default async ({ ssrContext }) => {
  const langIso = 'de' // ... some logic to determine it (use Cookies Plugin?)

  try {
    await import(
      /* webpackInclude: /(de|en-US)\.js$/ */
      'quasar/lang/' + langIso
    ).then(lang => {
      Quasar.lang.set(lang.default, ssrContext)
    })
  }
  catch (err) {
    // Requested Quasar Language Pack does not exist,
    // let's not break the app, so catching error
  }
}
```

## 在运行时切换语言包
下面是一个使用 QSelect 组件在运行时动态切换语言包的示例：

```html
<template>
  <q-select
    v-model="lang"
    :options="langOptions"
    label="Quasar Language"
    dense
    borderless
    emit-value
    map-options
    options-dense
    style="min-width: 150px"
  />
</template>

<script>
import { useQuasar } from 'quasar'
import languages from 'quasar/lang/index.json'
import { ref, watch } from 'vue'

const appLanguages = languages.filter(lang =>
  [ 'de', 'en-US' ].includes(lang.isoName)
)

const langOptions = appLanguages.map(lang => ({
  label: lang.nativeName, value: lang.isoName
}))

export default {
  setup () {
    const $q = useQuasar()
    const lang = ref($q.lang.isoName)

    watch(lang, val => {
      // dynamic import, so loading on demand only
      import(
        /* webpackInclude: /(de|en-US)\.js$/ */
        'quasar/lang/' + val
        ).then(lang => {
          $q.lang.set(lang.default)
        })
    })

    return {
      lang,
      langOptions
    }
  }
}
</script>
```

## 在项目中使用 Quasar 语言包
尽管 Quasar 语言包是为内置的 Quasar 组件设计的，但是您任仍然可以把他们用到您自己的项目代码中。

```html
获取 "Close" 文案在当前的语言中为
{{ $q.lang.label.close }}
```

在 [GitHub](https://github.com/quasarframework/quasar/tree/dev/ui/lang)页面查看`$q.lang`对象的结构.

## 根据地区决定语言

Quasar 也提供了一个根据地区来获取语言的方法：

```js
// 在 vue 文件之外这样使用：
import { Quasar } from 'quasar'
Quasar.lang.getLocale() // returns a string

// 在 vue 文件中这样使用：
import { useQuasar } from 'quasar'

setup () {
  const $q = useQuasar()
  $q.lang.getLocale() // returns a string
}
```

## 禁用 HTML 属性  <q-badge align="top" color="brand-primary" label="v2.11.3+" />
默认情况下，Quasar 会给 `<html>` 添加 `dir` 和 `lang` 属性。当 [RTL 开启](/options/rtl-support)时，`dir` 属性对 Sass 预处理器特别重要。

如果您禁用此行为，那么您可以：

```js
// Quasar CLI > quasar.config.js
framework: {
  config: {
    lang: {
      noHtmlAttrs: true // 添加这里
    }
  }
}
// Vite plugin / Vue plugin / UMD
app.use(Quasar, {
  config: {
    lang: {
      noHtmlAttrs: true // 添加这里
    }
  }
})
```
