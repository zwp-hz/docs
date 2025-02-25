---
title: "API: The build Property"
description: Nuxt.js lets you customize the webpack configuration for building your web application as you want.
---

# The build Property

> Nuxt.js lets you customize the webpack configuration for building your web application as you want.

## analyze

> Nuxt.js use [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer) to let you visualize your bundles and how to optimize them.

- Type: `Boolean` or `Object`
- Default: `false`

If an object, see available properties [here](https://github.com/webpack-contrib/webpack-bundle-analyzer#options-for-plugin).

Example (`nuxt.config.js`):

```js
export default {
  build: {
    analyze: true,
    // or
    analyze: {
      analyzerMode: 'static'
    }
  }
}
```

<div class="Alert Alert--teal">

**Info:** you can use the command `nuxt build --analyze` or `nuxt build -a` to build your application and launch the bundle analyzer on [http://localhost:8888](http://localhost:8888).

</div>

## babel

> Customize Babel configuration for JavaScript and Vue files. `.babelrc` is ignored by default.

- Type: `Object` See `babel-loader` [options](https://github.com/babel/babel-loader#options) and `babel` [options](https://babeljs.io/docs/en/options)
- Default:

  ```js
  {
    babelrc: false,
    cacheDirectory: undefined,
    presets: ['@nuxt/babel-preset-app']
  }
  ```

The default targets of [@nuxt/babel-preset-app](https://github.com/nuxt/nuxt.js/blob/dev/packages/babel-preset-app/src/index.js) are `ie: '9'` in the `client` build, and `node: 'current'` in the `server` build.

### presets

- Type: `Function`
- Argument:
  1. `Object`: { isServer: true | false }
  2. `Array`:
      - preset name `@nuxt/babel-preset-app`
      - [`options`](https://github.com/nuxt/nuxt.js/tree/dev/packages/babel-preset-app#options) of `@nuxt/babel-preset-app`

**Note**: The presets configured in `build.babel.presets` will be applied to both, the client and the server build. The target will be set by Nuxt accordingly (client/server). If you want configure the preset differently for the client or the server build, please use `presets` as a function:

> We **highly recommend** to use the default preset instead of below customization

```js
export default {
  build: {
    babel: {
      presets({ isServer }, [ preset, options ]) {
        // change options directly
        options.targets = isServer ? ... :  ...
        options.corejs = ...
        // return nothing
      }
    }
  }
}
```

Or override default value by returning whole presets list:

```js
export default {
  build: {
    babel: {
      presets({ isServer }, [ preset, options ]) {
        return [
          [
            preset, {
              buildTarget: isServer ? 'server' : 'client',
              ...options
          }],
          [
            // Other presets
          ]
        ]
      }
    }
  }
}
```

## cache

- Type: `Boolean`
- Default: `false`
- ⚠️ Experimental

> Enable cache of [terser-webpack-plugin](https://github.com/webpack-contrib/terser-webpack-plugin#options) and [cache-loader](https://github.com/webpack-contrib/cache-loader#cache-loader)

## crossorigin

- Type: `String`
- Default: `undefined`

  Configure the `crossorigin` attribute on `<link rel="stylesheet">` and `<script>` tags in generated HTML.

  More Info: [CORS settings attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes)

## cssSourceMap

- Type: `boolean`
- Default: `true` for dev and `false` for production.

> Enables CSS Source Map support

## devMiddleware

- Type: `Object`

See [webpack-dev-middleware](https://github.com/webpack/webpack-dev-middleware) for available options.

## devtools

- Type: `boolean`
- Default: `false`

Configure whether to allow [vue-devtools](https://github.com/vuejs/vue-devtools) inspection.

If you already activated through nuxt.config.js or otherwise, devtools enable regardless of the flag.

## extend

> Extend the webpack configuration manually for the client & server bundles.

- Type: `Function`

The extend is called twice, one time for the server bundle, and one time for the client bundle. The arguments of the method are:

1. The Webpack config object,
2. An object with the following keys (all boolean except `loaders`): `isDev`, `isClient`, `isServer`, `loaders`.


<div class="Alert Alert--orange">

  **Warning:**
  The `isClient` and `isServer` keys provided in are separate from the keys available in [`context`](/api/context).
  They are **not** deprecated. Do not use `process.client` and `process.server` here as they are `undefined` at this point.

</div>


Example (`nuxt.config.js`):

```js
export default {
  build: {
    extend (config, { isClient }) {
      // Extend only webpack config for client-bundle
      if (isClient) {
        config.devtool = '#source-map'
      }
    }
  }
}
```

If you want to see more about our default webpack configuration, take a look at our [webpack directory](https://github.com/nuxt/nuxt.js/tree/dev/packages/webpack/src/config).

### loaders in extend

`loaders` has the same object structure as [build.loaders](#loaders), so you can change the options of loaders inside `extend`.

Example (`nuxt.config.js`):

```js
export default {
  build: {
    extend (config, { isClient, loaders: { vue } }) {
      // Extend only webpack config for client-bundle
      if (isClient) {
        vue.transformAssetUrls.video = ['src', 'poster']
      }
    }
  }
}
```

## extractCSS

> Enables Common CSS Extraction using Vue Server Renderer [guidelines](https://ssr.vuejs.org/en/css.html).

- Type: `Boolean`
- Default: `false`

Using [`extract-css-chunks-webpack-plugin`](https://github.com/faceyspacey/extract-css-chunks-webpack-plugin/) under the hood, all your CSS will be extracted into separate files, usually one per component. This allows caching your CSS and JavaScript separately and is worth a try in case you have a lot of global or shared CSS.

<div class="Alert Alert--teal">

**Note:** There was a bug prior to Vue 2.5.18 that removed critical CSS imports when using this options.

</div>


## filenames

> Customize bundle filenames.

- Type: `Object`
- Default:

```js
{
  app: ({ isDev }) => isDev ? '[name].js' : '[chunkhash].js',
  chunk: ({ isDev }) => isDev ? '[name].js' : '[chunkhash].js',
  css: ({ isDev }) => isDev ? '[name].css' : '[contenthash].css',
  img: ({ isDev }) => isDev ? '[path][name].[ext]' : 'img/[hash:7].[ext]',
  font: ({ isDev }) => isDev ? '[path][name].[ext]' : 'fonts/[hash:7].[ext]',
  video: ({ isDev }) => isDev ? '[path][name].[ext]' : 'videos/[hash:7].[ext]'
}
```

This example changes fancy chunk names to numerical ids (`nuxt.config.js`):

```js
export default {
  build: {
    filenames: {
      chunk: ({ isDev }) => isDev ? '[name].js' : '[id].[chunkhash].js'
    }
  }
}
```

To understand a bit more about the use of manifests, take a look at this [webpack documentation](https://webpack.js.org/guides/code-splitting/).

## friendlyErrors

- Type: `Boolean`
- Default: `true` (Overlay enabled)

Enables or disables the overlay provided by [FriendlyErrorsWebpackPlugin](https://github.com/nuxt/friendly-errors-webpack-plugin)

## hardSource

- Type: `Boolean`
- Default: `false`
- ⚠️ Experimental

Enables the [HardSourceWebpackPlugin](https://github.com/mzgoddard/hard-source-webpack-plugin) for improved caching

## hotMiddleware

- Type: `Object`

See [webpack-hot-middleware](https://github.com/glenjamin/webpack-hot-middleware) for available options.

## html.minify

- Type: `Object`
- Default:

```js
{
  collapseBooleanAttributes: true,
  decodeEntities: true,
  minifyCSS: true,
  minifyJS: true,
  processConditionalComments: true,
  removeEmptyAttributes: true,
  removeRedundantAttributes: true,
  trimCustomFragments: true,
  useShortDoctype: true
}
```

**Attention:** If you make changes to `html.minify`, they won't be merged with the defaults!

Configuration for the [html-minifier](https://github.com/kangax/html-minifier) plugin used to minify
HTML files created during the build process (will be applied for *all modes*).

## indicator

> Display build indicator for hot module replacement in development (available in `v2.8.0+`)
- Type: `Boolean`
- Default: `true`

![nuxt-build-indicator](https://user-images.githubusercontent.com/5158436/58500509-93ba0f80-8197-11e9-8524-e115c6d32571.gif)

## loaders

> Customize options of Nuxt.js integrated webpack loaders.

- Type: `Object`
- Default:

```js
{
  file: {},
  fontUrl: { limit: 1000 },
  imgUrl: { limit: 1000 },
  pugPlain: {},
  vue: {
    transformAssetUrls: {
      video: 'src',
      source: 'src',
      object: 'src',
      embed: 'src'
    }
  },
  css: {},
  cssModules: {
    localIdentName: '[local]_[hash:base64:5]'
  },
  less: {},
  sass: {
    indentedSyntax: true
  },
  scss: {},
  stylus: {},
  ts: {
    transpileOnly: true,
    appendTsSuffixTo: [/\.vue$/]
  },
  tsx: {
    transpileOnly: true,
    appendTsxSuffixTo: [/\.vue$/]
  },
  vueStyle: {}
}
```

> Note: In addition to specifying the configurations in `nuxt.config.js`, it can also be modified by [build.extend](#extend)

### loaders.file

> More details are in [file-loader options](https://github.com/webpack-contrib/file-loader#options).

### loaders.fontUrl and loaders.imgUrl

> More details are in [url-loader options](https://github.com/webpack-contrib/url-loader#options).

### loaders.pugPlain

> More details are in [pug-plain-loader](https://github.com/yyx990803/pug-plain-loader) or [Pug compiler options](https://pugjs.org/api/reference.html#options).

### loaders.vue

> More details are in [vue-loader options](https://vue-loader.vuejs.org/options.html).

### loaders.css and loaders.cssModules

> More details are in [css-loader options](https://github.com/webpack-contrib/css-loader#options).
> Note: cssModules is loader options for usage of [CSS Modules](https://vue-loader.vuejs.org/guide/css-modules.html#css-modules)

### loaders.less

> You can pass any Less specific options to the `less-loader` via `loaders.less`. See the [Less documentation](http://lesscss.org/usage/#command-line-usage-options) for all available options in dash-case.

### loaders.sass and loaders.scss

> See the [Node Sass documentation](https://github.com/sass/node-sass/blob/master/README.md#options) for all available Sass options.
> Note: `loaders.sass` is for [Sass Indented Syntax](http://sass-lang.com/documentation/file.INDENTED_SYNTAX.html)

### loaders.ts

> Loader for typescript file and `lang="ts"` Vue SFC.
> More details are in [ts-loader options](https://github.com/TypeStrong/ts-loader#loader-options).

### loaders.tsx

> More details are in [ts-loader options](https://github.com/TypeStrong/ts-loader#options).

### loaders.vueStyle

> More details are in [vue-style-loader options](https://github.com/vuejs/vue-style-loader#options).

## optimization

- Type: `Object`
- Default:

  ```js
  {
    minimize: true,
    minimizer: [
      // terser-webpack-plugin
      // optimize-css-assets-webpack-plugin
    ],
    splitChunks: {
      chunks: 'all',
      automaticNameDelimiter: '.',
      name: undefined,
      cacheGroups: {}
    }
  }
  ```

The default value of `splitChunks.name` is `true` in `dev` or `analyze` mode.

You can set `minimizer` to a customized Array of plugins or set `minimize` to `false` to disable all minimizers.
(`minimize` is being disabled for development by default)

See [Webpack Optimization](https://webpack.js.org/configuration/optimization).

## optimizeCSS

- Type: `Object` or `Boolean`
- Default:
  - `false`
  - `{}` when extractCSS is enabled

OptimizeCSSAssets plugin options.

See [NMFR/optimize-css-assets-webpack-plugin](https://github.com/NMFR/optimize-css-assets-webpack-plugin).

## parallel

- Type: `Boolean`
- Default: `false`
- ⚠️ Experimental

> Enable [thread-loader](https://github.com/webpack-contrib/thread-loader#thread-loader) in webpack building

## plugins

> Add webpack plugins

- Type: `Array`
- Default: `[]`

Example (`nuxt.config.js`):

```js
import webpack from 'webpack'
import { version } from './package.json'
export default {
  build: {
    plugins: [
      new webpack.DefinePlugin({
        'process.VERSION': version
      })
    ]
  }
}
```

## postcss

> Customize [PostCSS Loader](https://github.com/postcss/postcss-loader#usage) plugins.

- Type: `Array` (legacy, will override defaults), `Object` (recommended), `Function` or `Boolean`

  **Note:** Nuxt.js has applied [PostCSS Preset Env](https://github.com/csstools/postcss-preset-env). By default it enables [Stage 2 features](https://cssdb.org/) and [Autoprefixer](https://github.com/postcss/autoprefixer), you can use `build.postcss.preset` to configure it.

- Default:

  ```js
  {
    plugins: {
      'postcss-import': {},
      'postcss-url': {},
      'postcss-preset-env': {},
      'cssnano': { preset: 'default' } // disabled in dev mode
    },
    order: 'cssnanoLast'
  }
  ```

Your custom plugin settings will be merged with the default plugins (unless you are using an `Array` instead of an `Object`).

Example (`nuxt.config.js`):

```js
export default {
  build: {
    postcss: {
      plugins: {
          // Disable `postcss-url`
        'postcss-url': false,
        // Add some plugins
        'postcss-nested': {},
        'postcss-responsive-type': {},
        'postcss-hexrgba': {}
      },
      preset: {
        autoprefixer: {
          grid: true
        }
      }
    }
  }
}
```

If the postcss configuration is an `Object`, `order` can be used for defining the plugin order:

- Type: `Array` (ordered plugin names), `String` (order preset name), `Function`
- Default: `cssnanoLast` (put `cssnano` in last)

Example (`nuxt.config.js`):

```js
export default {
  build: {
    postcss: {
      // preset name
      order: 'cssnanoLast',
      // ordered plugin names
      order: ['postcss-import', 'postcss-preset-env', 'cssnano']
      // Function to calculate plugin order
      order: (names, presets) => presets.cssnanoLast(names)
    }
  }
}
```

## profile

- Type: `Boolean`
- Default: enabled by command line argument `--profile`

> Enable the profiler in [WebpackBar](https://github.com/nuxt/webpackbar#profile)

## publicPath

> Nuxt.js lets you upload your dist files to your CDN for maximum performances, simply set the `publicPath` to your CDN.

- Type: `String`
- Default: `'/_nuxt/'`

Example (`nuxt.config.js`):

```js
export default {
  build: {
    publicPath: 'https://cdn.nuxtjs.org'
  }
}
```

Then, when launching `nuxt build`, upload the content of `.nuxt/dist/client` directory to your CDN and voilà!

## quiet

> Suppresses most of the build output log

- Type: `Boolean`
- Default: Enabled when a `CI` or `test` environment is detected by [std-env](https://github.com/blindmedia/std-env)

## splitChunks

- Type: `Object`
- Default:

  ```js
  {
    layouts: false,
    pages: true,
    commons: true
  }
  ```

If split codes for `layout`, `pages` and `commons` (common libs: vue|vue-loader|vue-router|vuex...).


## ssr

> Creates special webpack bundle for SSR renderer.

- Type: `Boolean`
- Default: `true` for universal mode and `false` for spa mode

This option is automatically set based on `mode` value if not provided.

## styleResources

- Type: `Object`
- Default: `{}`

<div class="Alert Alert--orange">

**Warning:** This property is deprecated. Please use the [style-resources-module](https://github.com/nuxt-community/style-resources-module/) instead for improved performance and better DX!

</div>

This is useful when you need to inject some variables and mixins in your pages without having to import them every time.

Nuxt.js uses https://github.com/yenshih/style-resources-loader to achieve this behaviour.

You need to specify the patterns/path you want to include for the given pre-processors: `less`, `sass`, `scss` or `stylus`

You cannot use path aliases here (`~` and `@`), you need to use relative or absolute paths.

`nuxt.config.js`:

```js
{
  build: {
    styleResources: {
      scss: './assets/variables.scss',
      less: './assets/*.less',
      // sass: ...,
      // scss: ...
      options: {
        // See https://github.com/yenshih/style-resources-loader#options
        // Except `patterns` property
      }
    }
  }
}
```

## templates

> Nuxt.js allows you provide your own templates which will be rendered based on Nuxt configuration. This feature is specially useful for using with [modules](/guide/modules).

- Type: `Array`

Example (`nuxt.config.js`):

```js
export default {
  build: {
    templates: [
      {
        src: '~/modules/support/plugin.js', // `src` can be absolute or relative
        dst: 'support.js', // `dst` is relative to project `.nuxt` dir
        options: { // Options are provided to template as `options` key
          live_chat: false
        }
      }
    ]
  }
}
```

Templates are rendered using [`lodash.template`](https://lodash.com/docs/#template) you can learn more about using them [here](https://github.com/learn-co-students/javascript-lodash-templates-v-000).

## terser

- Type: `Object` or `Boolean`
- Default:

```js
{
  parallel: true,
  cache: false,
  sourceMap: false,
  extractComments: {
    filename: 'LICENSES'
  },
  terserOptions: {
    output: {
      comments: /^\**!|@preserve|@license|@cc_on/
    }
  }
}
```

Terser plugin options. Set to `false` to disable this plugin.

`sourceMap` will be enabled when webpack `config.devtool` matches `source-?map`

See [webpack-contrib/terser-webpack-plugin](https://github.com/webpack-contrib/terser-webpack-plugin).

## transpile

- Type: `Array<string | RegExp>`
- Default: `[]`

If you want to transpile specific dependencies with Babel, you can add them in `build.transpile`. Each item in transpile can be a package name, or a string or regex object matching the dependency's file name.

## typescript

> Customize Nuxt.js TypeScript support.

<div class="Alert Alert--blue">

**Important**: This property will be ignored if [`TypeScript Support`](/guide/typescript) hasn't be set up in your project.

</div>

- Type: `Object`
- Default:

  ```js
  {
    typeCheck: true,
    ignoreNotFoundWarnings: false
  }
  ```

### typescript.typeCheck

> Enables TypeScript type checking on a separate process.

- Type: `Boolean` or `Object`
- Default: `true`

When enabled, Nuxt.js uses [fork-ts-checker-webpack-plugin](https://github.com/Realytics/fork-ts-checker-webpack-plugin) to provide type checking.

You can use an `Object` to override plugin options or set it to `false` to disable it.

### typescript.ignoreNotFoundWarnings

> Enables suppress not found typescript warnings.

- Type: `Boolean`
- Default: `false`

When enabled, you can suppress `export ... was not found ...` warnings.

See also about background information [https://github.com/TypeStrong/ts-loader/issues/653](https://github.com/TypeStrong/ts-loader/issues/653)

**Warning:** This property might suppress the warnings you want to see. Be careful with how you configure it.

## vueLoader

> Note: This config has been removed since Nuxt 2.0, please use [`build.loaders.vue`](#loaders)instead.

- Type: `Object`
- Default:

  ```js
  {
    productionMode: !this.options.dev,
    transformAssetUrls: {
      video: 'src',
      source: 'src',
      object: 'src',
      embed: 'src'
    }
  }
  ```

> Specify the [Vue Loader Options](https://vue-loader.vuejs.org/options.html).

## watch

> You can provide your custom files to watch and regenerate after changes. This feature is specially useful for using with [modules](/guide/modules).

- Type: `Array<String>`

```js
export default {
  build: {
    watch: [
      '~/.nuxt/support.js'
    ]
  }
}
```
