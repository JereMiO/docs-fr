---
sidebar: false
---

# Announcing Vite 2.0

<p style="text-align:center">
  <img src="/logo.svg" style="height:200px">
</p>

Nous sommes très heureux d’annoncer aujourd’hui la sortie officielle de Vite 2.0!

Vite (French word for "fast", pronounced `/vit/`) is a new kind of build tool for frontend web development. Think a pre-configured dev server + bundler combo, but leaner and faster. It leverages browser's [native ES modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) support and tools written in compile-to-native languages like [esbuild](https://esbuild.github.io/) to deliver a snappy and modern development experience.

To get a sense of how fast Vite is, check out [this video comparison](https://twitter.com/amasad/status/1355379680275128321) of booting up a React application on Repl.it using Vite vs. `create-react-app` (CRA).

If you've never heard of Vite before and would love to learn more about it, check out [the rationale behind the project](https://vitejs.dev/guide/why.html). If you are interested in how Vite differs from other similar tools, check out the [comparisons](https://vitejs.dev/guide/comparisons.html).
Si vous n’aviez jamais entendu parler de Vite jusqu’ici et que vous voudriez en savoir plus, nous avons une page expliquant les [raisons qui motivent le projet](/guide/why.html). Si vous vous demandez en quoi Vite diffère des outils similaires, nous avons aussi une page de [comparaisons](/guide/comparisons.html).

## What's New in 2.0

Since we decided to completely refactor the internals before 1.0 got out of RC, this is in fact the first stable release of Vite. That said, Vite 2.0 brings about many big improvements over its previous incarnation:

### Framework Agnostic Core

The original idea of Vite started as a [hacky prototype that serves Vue single-file components over native ESM](https://github.com/vuejs/vue-dev-server). Vite 1 was a continuation of that idea with HMR implemented on top.

Vite 2.0 takes what we learned along the way and is redesigned from scratch with a more robust internal architecture. It is now completely framework agnostic, and all framework-specific support is delegated to plugins. There are now [official templates for Vue, React, Preact, Lit Element](https://github.com/vitejs/vite/tree/main/packages/create-vite), and ongoing community efforts for Svelte integration.

### New Plugin Format and API

Inspired by [WMR](https://github.com/preactjs/wmr), the new plugin system extends Rollup's plugin interface and is [compatible with many Rollup plugins](https://vite-rollup-plugins.patak.dev/) out of the box. Plugins can use Rollup-compatible hooks, with additional Vite-specific hooks and properties to adjust Vite-only behavior (e.g. differentiating dev vs. build or custom handling of HMR).

L’[API programmatique](/guide/api-javascript.html) a également été beaucoup améliorée pour faciliter l’apparition d’outils ou de frameworks de plus haut niveau par-dessus Vite.

### esbuild Powered Dep Pre-Bundling

Puisque Vite est un serveur de développement basé sur les modules ES natifs, il pré-bundle les dépendances pour réduire le nombre de requêtes et gérer les conversions de CommonJS en des modules ES. Auparavant, Vite effectuait cette opération à l’aide de Rollup, et pour cette 2.0 il utilise désormais `esbuild`, ce qui permet une diminution entre 10 et 100 fois de la durée de cette phase. À titre d’exemple, démarrer à froid une application de test avec de grosses dépendances comme React Material UI prenait auparavant 28 secondes sur un Macbook Pro à processeur M1, et cela prend désormais environ 1,5 secondes. Attendez-vous à des améliorations de cet ordre-là si votre setup actuel utilise un bundler traditionnel.

### First-class CSS Support

Vite treats CSS as a first-class citizen of the module graph and supports the following out of the box:

- **Modification par le résolveur** : les chemins en `@import` ou `url()` dans le CSS sont modifiés à l’aide du résolveur de Vite afin de respecter les alias et les dépendances npm.
- **Réécriture de la base des URLs** : les chemis en `url()` voient leur base automatiquement réécrite, peu importe où se trouve le fichier importé.
- **Fractionnement (_code splitting_) du CSS** : un morceau (_chunk_) en JS émet également le CSS correspondant, qui sera automatiquement chargé en parallèle de ce dernier.

### Server-Side Rendering (SSR) Support

Vite 2.0 est livré avec le [support expérimental du rendu côté serveur](/guide/ssr.html). Vite fournit des APIs afin de charger et de mettre à jour efficacement le code source sous forme de modules ES en Node.js pendant le développement (presque comme du rafraîchissement de modules à la volée côté serveur), et externalise automatiquement les dépendances compatibles avec CommonJS pour rendre la compilation plus rapide. Le serveur de production peut être complètement découplé de Vite, et le même setup peut être facilement adapté pour permettre le pré-rendu / la génération statique (_SSG_).

Vite SSR is provided as a low-level feature and we are expecting to see higher level frameworks leveraging it under the hood.

### Opt-in Legacy Browser Support

Vite targets modern browsers with native ESM support by default, but you can also opt-in to support legacy browsers via the official [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy). The plugin automatically generates dual modern/legacy bundles, and delivers the right bundle based on browser feature detection, ensuring more efficient code in modern browsers that support them.

## Give it a Try!

That was a lot of features, but getting started with Vite is simple! You can spin up a Vite-powered app literally in a minute, starting with the following command (make sure you have Node.js >=12):

```bash
npm init @vitejs/app
```

Ensuite, vous pouvez suivre [ce guide](/guide/) pour voir ce que Vite a à proposer. Vous pouvez également voir le code source sur [GitHub](https://github.com/vitejs/vite), suivre les mises à jour sur [Twitter](https://twitter.com/vite_js), ou venir discuter avec d’autres utilisateurs de Vite sur le [serveur Discord](http://chat.vitejs.dev/).
