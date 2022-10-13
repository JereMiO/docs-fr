# Compilation en production

Lorsqu’il est temps de déployer votre application en production, lancez simplement la commande `vite build`. Par défaut, elle utilise `<racine>/index.html` comme point d’entrée pour la compilation, et produit un bundle d’application qu’il est possible de servir avec un service d’hébergement statique. Vous pouvez retrouver des guides pour les services les plus populaires sur la page [Déployer un site statique](./static-deploy).

## Compatibilité navigateur

Ce bundle de production part du principe que le JavaScript moderne est supporté. Par défaut, Vite cible les navigateurs qui supportent les [modules ES natifs](https://caniuse.com/es6-module), les [imports dynamiques de modules ES natifs](https://caniuse.com/es6-module-dynamic-import) et [`import.meta`](https://caniuse.com/mdn-javascript_statements_import_meta):

- Chrome >=87
- Firefox >=78
- Safari >=13
- Edge >=88

Vous pouvez spécifier des cibles personnalisées à l’aide de l’[option de configuration `build.target`](/config/#build-target), sachant que la cible minimum est `es2015`.

Notez que par défaut, Vite ne s’occupe que des transformations de syntaxe et **n’insère pas de polyfill**. Regardez du côté de [Polyfill.io](https://polyfill.io/v3/) qui est un service qui génère automatiquement des bundles de polyfills en se basant sur la chaîne de caractères de l’agent utilisateur.

Les navigateurs plus anciens peuvent être supportés à l’aide de [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy), qui va générer automatiquement des morceaux (_chunks_) pour navigateurs plus anciens et les polyfills de fonctionnalités du langage ES qui correspondent. Les morceaux pour navigateurs plus anciens sont chargés conditionnellement seulement dans les navigateurs qui ne supportent pas les modules ES natifs.

## Chemin public de base

- Voir aussi: [Gestion des ressources statiques](./assets)

Si vous déployez votre projet sous un chemin public imbriqué, spécifiez l’[option de configuration `base`](/config/#base) et tous les chemins de ressources seront réécris en conséquence. Cette option peut aussi être spécifiée via l’interface en ligne de commande, par exemple `vite build --base=/mon/chemin/public/`.

Les URLs de ressources importées par le JavaScript, les références `url()` dans le CSS, et les références à des ressources dans vos fichiers `.html` sont toutes ajustées automatiquement pour respecter cette option pendant la compilation.

La seule exception est quand vous devez concaténer dynamiquement des URLs à la volée. Dans ce cas, utilisez la variable injectée globalement `import.meta.env.BASE_URL` qui contiendra le chemin public de base. Notez que cette variable est remplacée statiquement pendant la compilation alors elle doit apparaître telle quelle (par exemple `import.meta.env['BASE_URL']` ne fonctionnera pas).

For advanced base path control, check out [Advanced Base Options](#advanced-base-options).

## Customiser la compilation

The build can be customized via various [build config options](/config/build-options.md). Specifically, you can directly adjust the underlying [Rollup options](https://rollupjs.org/guide/en/#big-list-of-options) via `build.rollupOptions`:

```js
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      // https://rollupjs.org/guide/en/#big-list-of-options
    }
  }
})
```

Par exemple, vous pouvez spécifier plusieurs sorties Rollup à l’aide de plugins qui ne sont appliqués que pour la compilation.

## Stratégie de découpage en morceaux (_chunks_)

You can configure how chunks are split using `build.rollupOptions.output.manualChunks` (see [Rollup docs](https://rollupjs.org/guide/en/#outputmanualchunks)). Until Vite 2.8, the default chunking strategy divided the chunks into `index` and `vendor`. It is a good strategy for some SPAs, but it is hard to provide a general solution for every Vite target use case. From Vite 2.9, `manualChunks` is no longer modified by default. You can continue to use the Split Vendor Chunk strategy by adding the `splitVendorChunkPlugin` in your config file:

```js
// vite.config.js
import { splitVendorChunkPlugin } from 'vite'
export default defineConfig({
  plugins: [splitVendorChunkPlugin()]
})
```

This strategy is also provided as a `splitVendorChunk({ cache: SplitVendorChunkCache })` factory, in case composition with custom logic is needed. `cache.reset()` needs to be called at `buildStart` for build watch mode to work correctly in this case.

## Refaire la compilation lorsque les fichiers sont modifiés

You can enable rollup watcher with `vite build --watch`. Or, you can directly adjust the underlying [`WatcherOptions`](https://rollupjs.org/guide/en/#watch-options) via `build.watch`:

```js
// vite.config.js
export default defineConfig({
  build: {
    watch: {
      // https://rollupjs.org/guide/en/#watch-options
    }
  }
})
```

Avec le signal `--watch`, les changements faits à `vite.config.js` ou à n’importe quel fichier bundlé donnera lieu à une recompilation.

## Application multi-pages

Supposons que votre code source a la structure suivante:

```
├── package.json
├── vite.config.js
├── index.html
├── main.js
└── nested
    ├── index.html
    └── nested.js
```

Pendant la développement, naviguez à `/nested/` —cela fonctionne comme attendu, du moins dans le contexte d’un serveur de fichiers statiques.

Pour la compilation, tout ce que vous aurez à faire est de spécifier plusieurs fichiers `.html` comme points d’entrée:

```js
// vite.config.js
import { resolve } from 'path'
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        nested: resolve(__dirname, 'nested/index.html')
      }
    }
  }
})
```

Si vous spécifiez une racine différente, souvenez-vous que `__dirname` sera toujours le dossier où se trouve votre fichier vite.config.js lorsque les chemins sont résolus. Ainsi, vous devrez ajouter la racine de votre entrée aux arguments de `resolve`.

## Mode librairie

Lorsque vous développerez une librairie à destination du navigateur, vous passerez sûrement une bonne partie de votre temps sur une page de démo ou de test qui importe votre librairie. Avec Vite, vous pouvez utiliser le `index.html` pour cela, et profiter d’une meilleure expérience de développement.

Lorsqu’il est temps de faire le bundle de votre librairie pour commencer à la distribuer, utilisez l’[option de configuration `build.lib`](/config/#build-lib). Assurez vous d’externaliser les dépendances que vous ne souhaitez pas retrouver dans votre bundle de librairie, comme `vue` ou `react` par exemple:


```js
// vite.config.js
import { resolve } from 'path'
import { defineConfig } from 'vite'

export default defineConfig({
  build: {
    lib: {
      // Could also be a dictionary or array of multiple entry points
      entry: resolve(__dirname, 'lib/main.js'),
      name: 'MyLib',
      // les extensions nécessaires seront ajoutées
      fileName: 'my-lib'
    },
    rollupOptions: {
      // externalisez les dépendances que vous ne souhaitez pas inclure
      // au bundle de votre librairie
      external: ['vue'],
      output: {
        // assurez-vous de fournir les variables globales à utiliser
        // pour la compilation UMD des dépendances externalisées
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
})
```

Le fichier d’entrée doit contenir les exports qui seront importés par les utilisateurs de votre package:

```js
// lib/main.js
import Foo from './Foo.vue'
import Bar from './Bar.vue'
export { Foo, Bar }
```

Lancer `vite build` avec cette configuration exploite un preset de Rollup qui est fait pour livrer des librairies et qui produit des bundles sous deux formats: `es` et `umd` (configurables avec `build.lib`):

```
$ vite build
building for production...
dist/my-lib.js      0.08 KiB / gzip: 0.07 KiB
dist/my-lib.umd.cjs 0.30 KiB / gzip: 0.16 KiB
```

Le `package.json` recommandé pour votre librairie:

```json
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.cjs",
  "module": "./dist/my-lib.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.js",
      "require": "./dist/my-lib.umd.cjs"
    }
  }
}
```

Or, if exposing multiple entry points:

```json
{
  "name": "my-lib",
  "type": "module",
  "files": ["dist"],
  "main": "./dist/my-lib.cjs",
  "module": "./dist/my-lib.mjs",
  "exports": {
    ".": {
      "import": "./dist/my-lib.mjs",
      "require": "./dist/my-lib.cjs"
    },
    "./secondary": {
      "import": "./dist/secondary.mjs",
      "require": "./dist/secondary.cjs"
    }
  }
}
```

::: tip Note
If the `package.json` does not contain `"type": "module"`, Vite will generate different file extensions for Node.js compatibility. `.js` will become `.mjs` and `.cjs` will become `.js`.
:::

::: tip Environment Variables
In library mode, all `import.meta.env.*` usage are statically replaced when building for production. However, `process.env.*` usage are not, so that consumers of your library can dynamically change it. If this is undesirable, you can use `define: { 'process.env.`<wbr>`NODE_ENV': '"production"' }` for example to statically replace them.
:::

## Options avancées du chemin de base

::: warning
Cette fonctionnalité est expérimentale; l’API pourra très bien changer dans une future version mineure sans suivre le SemVer. Fixez la version mineure de Vite si vous l’utilisez.
:::

For advanced use cases, the deployed assets and public files may be in different paths, for example to use different cache strategies.
A user may choose to deploy in three different paths:

- Les fichiers HTML d’entrée générés (qui peuvent aussi être traités par le rendu côté serveur)
- Les ressources hachées générées (JS, CSS, et les autres types de fichier comme les images)
- Les [fichiers publics](assets.md#le-repertoire-public) copiés

A single static [base](#public-base-path) isn't enough in these scenarios. Vite provides experimental support for advanced base options during build, using `experimental.renderBuiltUrl`.

```ts
experimental: {
  renderBuiltUrl(filename: string, { hostType }: { hostType: 'js' | 'css' | 'html' }) {
    if (hostType === 'js') {
      return { runtime: `window.__toCdnUrl(${JSON.stringify(filename)})` }
    } else {
      return { relative: true }
    }
  }
}
```

If the hashed assets and public files aren't deployed together, options for each group can be defined independently using asset `type` included in the second `context` param given to the function.

```ts
experimental: {
  renderBuiltUrl(filename: string, { hostId, hostType, type }: { hostId: string, hostType: 'js' | 'css' | 'html', type: 'public' | 'asset' }) {
    if (type === 'public') {
      return 'https://www.domain.com/' + filename
    }
    else if (path.extname(hostId) === '.js') {
      return { runtime: `window.__assetsPath(${JSON.stringify(filename)})` }
    }
    else {
      return 'https://cdn.domain.com/assets/' + filename
    }
  }
}
```
