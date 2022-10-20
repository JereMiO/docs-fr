# HMR API

:::tip Note
Il s'agit ici de l'API de rafraîchissement des modules à la volée cliente. Pour gérer les rafraîchissements à l'aide d'un plugin, voir [handleHotUpdate](./api-plugin#handlehotupdate).

L'API de remplacement des modules est surtout faite pour les développeurs de frameworks ou d'outils. En tant qu'utilisateur final, le rafraîchissement est normalement géré pour vous dans les templates de démarrage spécifiques aux frameworks.
:::

Vite expose son API de remplacement des modules à la volée à l'aide de l'objet spécial `import.meta.hot`:

```ts
interface ImportMeta {
  readonly hot?: ViteHotContext
}

type ModuleNamespace = Record<string, any> & {
  [Symbol.toStringTag]: 'Module'
}

interface ViteHotContext {
  readonly data: any

  accept(): void
  accept(cb: (mod: ModuleNamespace | undefined) => void): void
  accept(dep: string, cb: (mod: ModuleNamespace | undefined) => void): void
  accept(
    deps: readonly string[],
    cb: (mods: Array<ModuleNamespace | undefined>) => void
  ): void

  dispose(cb: (data: any) => void): void
  decline(): void
  invalidate(): void

  // `InferCustomEventPayload` provides types for built-in Vite events
  on<T extends string>(
    event: T,
    cb: (payload: InferCustomEventPayload<T>) => void
  ): void
  send<T extends string>(event: T, data?: InferCustomEventPayload<T>): void
}
```

## Gardes conditionnels requis

D'abord, assurez-vous de mettre en place des gardes à l'utilisation de l'API de rafraîchissement à la volée avec un bloc conditionnel afin que le code inutile puisse être éliminé en production (_tree-shaking_):

```js
if (import.meta.hot) {
  // code de rafraîchissement à la volée
}
```

## `hot.accept(cb)`

Pour qu'un module s'auto-accepte, utilisez `import.meta.hot.accept` avec un callback qui reçoit le module mis à jour:

```js
export const count = 1

if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    if (newModule) {
      // newModule is undefined when SyntaxError happened
      console.log('updated: count is now ', newModule.count)
    }
  })
}
```

Un module qui «accepte» les rafraîchissements à la volée est une **frontière de rafraîchissement à la volée (_HMR boundary_)**.

Notez que le rafraîchissement de Vite ne remplace pas vraiment le module importé à l'origine: si un module frontière de rafraîchissement ré-exporte un import d'une dépendance, alors il est responsable de la mise à jour de ces ré-exports (et ces exports doivent utiliser `let`). Les importeurs en amont du module frontière ne seront pas notifiés du changement.

Cette implémentation simplifiée du remplacement de modules à la volée est suffisante pour la plupart des cas d'usage en développement, et nous permet d'éviter la coûteuse génération des modules proxy.

## `hot.accept(deps, cb)`

Un module peut aussi accepter une mise à jour d'une dépendance directe sans se recharger:

```js
import { foo } from './foo.js'

foo()

if (import.meta.hot) {
  import.meta.hot.accept('./foo.js', (newFoo) => {
    // le callback reçoit le module './foo.js' mis à jour
    newFoo.foo()
  })

  // la méthode peut aussi accepter une liste de modules de dépendances:
  import.meta.hot.accept(
    ['./foo.js', './bar.js'],
    ([newFooModule, newBarModule]) => {
      // le callback reçoit les modules mis à jour dans un Array
    }
  )
}
```

## `hot.dispose(cb)`

Un module auto-accepté ou un module qui s'attend à être accepté par d'autres peut utiliser `hot.dispose` pour nettoyer les effets secondaires persistants créés par sa copie mise à jour:

```js
function setupSideEffect() {}

setupSideEffect()

if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // nettoyage des effets secondaires
  })
}
```

## `hot.data`

L'objet `import.meta.hot.data` est persisté entre les différentes instances d'un même module mis à jour. Il peut être utilisé pour passer des informations depuis une version précédente du module vers la suivante.

## `hot.decline()`

Appeler `import.meta.hot.decline()` indique que ce module n'est pas remplaçable à la volée, et que le navigateur doit recharger complétement la page si ce module est rencontré pendant la propagation des mises à jour de modules.

## `hot.invalidate()`

A self-accepting module may realize during runtime that it can't handle a HMR update, and so the update needs to be forcefully propagated to importers. By calling `import.meta.hot.invalidate()`, the HMR server will invalidate the importers of the caller, as if the caller wasn't self-accepting.

Note that you should always call `import.meta.hot.accept` even if you plan to call `invalidate` immediately afterwards, or else the HMR client won't listen for future changes to the self-accepting module. To communicate your intent clearly, we recommend calling `invalidate` within the `accept` callback like so:

```js
import.meta.hot.accept((module) => {
  // You may use the new module instance to decide whether to invalidate.
  if (cannotHandleUpdate(module)) {
    import.meta.hot.invalidate()
  }
})
```

## `hot.on(event, cb)`

Écoute un évènement de remplacement à la volée.

Les évènements de remplacement suivants sont émis par Vite automatiquement:

- `'vite:beforeUpdate'` quand une mise à jour est sur le point d'être appliquée (par exemple lorsqu'un module va être remplacé)
- `'vite:beforeFullReload'` quand un rechargement complet est sur le point d'avoir lieu
- `'vite:beforePrune'` quand des modules qui ne sont plus nécessaires sont sur le point d'être élagués
- `'vite:error'` quand une erreur survient (par exemple une erreur de syntaxe)

Des évènements de remplacement à la volée custom peuvent aussi être émis par les plugins. Voir [handleHotUpdate](./api-plugin#handlehotupdate) pour plus de détails.

## `hot.send(event, data)`

Send custom events back to Vite's dev server.

If called before connected, the data will be buffered and sent once the connection is established.

See [Client-server Communication](/guide/api-plugin.html#client-server-communication) for more details.
