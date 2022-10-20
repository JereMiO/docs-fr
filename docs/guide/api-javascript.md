# API JavaScript

Les APIs JavaScript de Vite sont complétement typées, et il est recommandé d'utiliser TypeScript ou d'activer la vérification des types JS dans VSCode pour profiter d'IntelliSense et de la validation.

## `createServer`

**Signature de type :**

```ts
async function createServer(inlineConfig?: InlineConfig): Promise<ViteDevServer>
```

**Exemple d'utilisation:**

```js
import { fileURLToPath } from 'url'
import { createServer } from 'vite'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

;(async () => {
  const server = await createServer({
    // any valid user config options, plus `mode` and `configFile`
    configFile: false,
    root: __dirname,
    server: {
      port: 1337
    }
  })
  await server.listen()

  server.printUrls()
})()
```

::: tip NOTE
When using `createServer` and `build` in the same Node.js process, both functions rely on `process.env.`<wbr>`NODE_ENV` to work properly, which also depends on the `mode` config option. To prevent conflicting behavior, set `process.env.`<wbr>`NODE_ENV` or the `mode` of the two APIs to `development`. Otherwise, you can spawn a child process to run the APIs separately.
:::

## `InlineConfig`

L'interface `InlineConfig` étend `UserConfig` avec des propriétés supplémentaires:

- `configFile`: spécifie le fichier de configuration à utiliser. S'il n'est pas fourni, Vite essaiera de le résoudre depuis la racine projet. Définissez-la à `false` pour désactiver la résolution automatique.
- `envFile`: définissez-la à `false` pour désactiver la prise en charge des fichiers `.env`.

## `ResolvedConfig`

The `ResolvedConfig` interface has all the same properties of a `UserConfig`, except most properties are resolved and non-undefined. It also contains utilities like:

- `config.assetsInclude`: A function to check if an `id` is considered an asset.
- `config.logger`: Vite's internal logger object.

## `ViteDevServer`

```ts
interface ViteDevServer {
  /**
   * L'objet de configuration Vite résolu.
   */
  config: ResolvedConfig
  /**
   * Une instance d'application connect,
   * - peut être utilisée pour attacher des middlewares custom au serveur de
   *   développement.
   * - peut aussi être utilisée comme fonction handler d'un serveur HTTP custom
   *   ou comme middleware d'un framework Node.js de style connect.
   *
   * https://github.com/senchalabs/connect#use-middleware
   */
  middlewares: Connect.Server
  /**
   * Instance native du serveur HTTP Node.
   * Sera null en mode middleware.
   */
  httpServer: http.Server | Http2SecureServer | null
  /**
   * Instance du watcher Chokidar.
   * https://github.com/paulmillr/chokidar#api
   */
  watcher: FSWatcher
  /**
   * Serveur web socket avec une méthode `send(payload)`.
   */
  ws: WebSocketServer
  /**
   * Conteneur de plugin Rollup qui peut lancer des hooks de plugins pour un
   * fichier donné.
   */
  pluginContainer: PluginContainer
  /**
   * Le graphe des modules stockant les relations d'import, les correspondances
   * entre URLs et fichiers, et l'état du remplacement des modules à la volée.
   */
  moduleGraph: ModuleGraph
  /**
   * The resolved urls Vite prints on the CLI. null in middleware mode or
   * before `server.listen` is called.
   */
  resolvedUrls: ResolvedServerUrls | null
  /**
   * Programmatically resolve, load and transform a URL and get the result
   * without going through the http request pipeline.
   */
  transformRequest(
    url: string,
    options?: TransformOptions
  ): Promise<TransformResult | null>
  /**
   * Appliquer les transformations du HTML internes à Vite ainsi que les
   * transformations HTML des plugins.
   */
  transformIndexHtml(url: string, html: string): Promise<string>
  /**
   * Charger une URL donnée en tant que module instancié pour le rendu côté
   * serveur.
   */
  ssrLoadModule(
    url: string,
    options?: { isolated?: boolean }
  ): Promise<Record<string, any>>
  /**
   * Corriger la stacktrace des erreurs en rendu côté serveur.
   */
  ssrFixStacktrace(e: Error): void
  /**
   * Triggers HMR for a module in the module graph. You can use the `server.moduleGraph`
   * API to retrieve the module to be reloaded. If `hmr` is false, this is a no-op.
   */
  reloadModule(module: ModuleNode): Promise<void>
  /**
   * Démarrer le serveur.
   */
  listen(port?: number, isRestart?: boolean): Promise<ViteDevServer>
  /**
   * Relancer le serveur.
   *
   * @param forceOptimize - force l'optimisateur à refaire le bundling, à la
   * façon du signal --force de l'interface en ligne de commande
   */
  restart(forceOptimize?: boolean): Promise<void>
  /**
   * Arrête le serveur.
   */
  close(): Promise<void>
}
```

## `build`

**Signature de type:**

```ts
async function build(
  inlineConfig?: InlineConfig
): Promise<RollupOutput | RollupOutput[]>
```

**Exemple d'utilisation:**

```js
import path from 'path'
import { fileURLToPath } from 'url'
import { build } from 'vite'

const __dirname = fileURLToPath(new URL('.', import.meta.url))

;(async () => {
  await build({
    root: path.resolve(__dirname, './project'),
    base: '/foo/',
    build: {
      rollupOptions: {
        // ...
      }
    }
  })
})()
```

## `preview`

**Signature de type:**

```ts
async function preview(inlineConfig?: InlineConfig): Promise<PreviewServer>
```

**Exemple d'utilisation:**

```js
const { preview } = require('vite')

;(async () => {
  const previewServer = await preview({
    // n'importe quelles options de configuration valides, ainsi que `mode` et
    // `configFile`
    preview: {
      port: 8080,
      open: true
    }
  })

  previewServer.printUrls()
})()
```

## `resolveConfig`

**Signature de type:**

```ts
async function resolveConfig(
  inlineConfig: InlineConfig,
  command: 'build' | 'serve',
  defaultMode = 'development'
): Promise<ResolvedConfig>
```

The `command` value is `serve` in dev (in the cli `vite`, `vite dev`, and `vite serve` are aliases).

## `mergeConfig`

**Type Signature:**

```ts
function mergeConfig(
  defaults: Record<string, any>,
  overrides: Record<string, any>,
  isRoot = true
): Record<string, any>
```

Deeply merge two Vite configs. `isRoot` represents the level within the Vite config which is being merged. For example, set `false` if you're merging two `build` options.

## `searchForWorkspaceRoot`

**Type Signature:**

```ts
function searchForWorkspaceRoot(
  current: string,
  root = searchForPackageRoot(current)
): string
```

**Related:** [server.fs.allow](/config/server-options.md#server-fs-allow)

Search for the root of the potential workspace if it meets the following conditions, otherwise it would fallback to `root`:

- contains `workspaces` field in `package.json`
- contains one of the following file
  - `lerna.json`
  - `pnpm-workspace.yaml`

## `loadEnv`

**Type Signature:**

```ts
function loadEnv(
  mode: string,
  envDir: string,
  prefixes: string | string[] = 'VITE_'
): Record<string, string>
```

**Related:** [`.env` Files](./env-and-mode.md#env-files)

Load `.env` files within the `envDir`. By default, only env variables prefixed with `VITE_` are loaded, unless `prefixes` is changed.

## `normalizePath`

**Type Signature:**

```ts
function normalizePath(id: string): string
```

**Related:** [Path Normalization](./api-plugin.md#path-normalization)

Normalizes a path to interoperate between Vite plugins.

## `transformWithEsbuild`

**Signature de type:**

```ts
async function transformWithEsbuild(
  code: string,
  filename: string,
  options?: EsbuildTransformOptions,
  inMap?: object
): Promise<ESBuildTransformResult>
```

Transform JavaScript or TypeScript with esbuild. Useful for plugins that prefer matching Vite's internal esbuild transform.

## `loadConfigFromFile`

**Type Signature:**

```ts
async function loadConfigFromFile(
  configEnv: ConfigEnv,
  configFile?: string,
  configRoot: string = process.cwd(),
  logLevel?: LogLevel
): Promise<{
  path: string
  config: UserConfig
  dependencies: string[]
} | null>
```

Load a Vite config file manually with esbuild.
