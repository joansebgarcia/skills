# Changesets (`@changesets/cli`): investigación para versionado y releases en este repo

> Fecha de la investigación: 2026-10-12.
> Fuentes: únicamente primarias (sitio oficial https://changesets.dev, repo https://github.com/changesets/changesets, repo https://github.com/changesets/action y el código fuente de los paquetes). Context7 (`/changesets/changesets`, `/changesets/action`) se usó para contrastar y devolvió el mismo contenido que el repo oficial.
> Snapshots revisados: `changesets/changesets@c73949b` (main, 2026-09-22) y `changesets/action@615034b` (main, 2026-10-02).

## 0. Hallazgos clave antes de empezar

- **Changesets v3 es la versión actual.** `npm view` (2026-10-12): `@changesets/cli@3.0.3`, `@changesets/changelog-github@1.0.1`, `@changesets/changelog-git@1.0.0`, `@changesets/config@4.0.1`. La v3 salió el 11-08-2026 (https://changesets.dev/blog/announcing-changesets-v3). Los tags más recientes de la Action son `v2.1.x` (`git ls-remote https://github.com/changesets/action`); la v2 de la Action es la compatible con Changesets v3 (https://github.com/changesets/action/blob/main/README.md).
- **La carpeta `docs/` del repo de changesets está desactualizada**: cada archivo empieza con "This documentation is outdated. View the up-to-date version at https://changesets.dev/..." (https://github.com/changesets/changesets/blob/main/docs/config-file-options.md). Por ejemplo, `docs/` todavía dice que `baseBranch` es `"master"` por defecto y documenta la opción `prettier`, que en v3 fue reemplazada por `format`. La fuente válida es `site/` (publicada en changesets.dev), y eso es lo que se cita aquí.
- **Requisitos v3**: Node.js `^22.11 || ^24 || >=26`; pnpm `>=10.0.0`, npm `>=10.9.0` o yarn `>=4.5.2`. Todos los paquetes son solo ESM (https://changesets.dev/guide/getting-started#install-requirements, https://changesets.dev/guide/migration#all-packages-are-now-es-modules).

### Estado actual de este repo (`/Users/sebastiandotdev/dev/skills`)

- Un único commit (`Initial commit`) en `main`; remote `origin` = `https://github.com/joansebgarcia/skills.git`.
- Archivos: `README.md` ("My Agents Skills. Straight from my .agents directory."), `package.json` (sin trackear) y una carpeta `skills/` vacía. No hay `.github/workflows/`, lockfile ni campo `"workspaces"` → **es un repo de un solo paquete**, no un monorepo.
- `package.json` (actualizado el 2026-10-12): `"name": "@joansebgarcia/skills"`, `"version": "1.0.0"`, `"type": "commonjs"`, `"main": "index.js"` (no existe), **sin `"private": true`** y sin `devEngines` → **el gestor de paquetes es npm** (generado con `npm init`). Entorno local: npm 11.17.0 y Node 24.14.1, que cumplen los requisitos de v3.
- `"type": "commonjs"` no afecta: la restricción "solo ESM" de v3 es de los paquetes de Changesets, no del proyecto que los usa (https://changesets.dev/guide/migration#all-packages-are-now-es-modules).
- **Nombre en npm**: `skills` sin scope está ocupado (`skills@1.7.2`, mantenedores `rauchg`, `quuu`), pero **`@joansebgarcia/skills` está libre** (`npm view @joansebgarcia/skills` → `E404`, 2026-10-12). Al ser un paquete con scope, publicarlo como público requiere `access: "public"` (ver sección 2).

---

## 1. Instalación y `changeset init`

Instalación (npm) e inicialización (https://changesets.dev/guide/getting-started#setting-up):

```bash
npm install --save-dev @changesets/cli
npx changeset init
```

Qué hace `init` (verificado en el código, https://github.com/changesets/changesets/blob/main/packages/cli/src/commands/init/index.ts):

- Si `.changeset/config.json` ya existe, no hace nada.
- Crea `.changeset/` y genera `config.json` **mediante preguntas interactivas** (https://changesets.dev/guide/cli#init):
  1. ¿Usar la integración de GitHub para changelogs? → si sí, pide `org/repo` y escribe `"changelog": ["@changesets/changelog-github", { "repo": "org/repo" }]`.
  2. ¿Commitear automáticamente changesets y bumps? → `commit`.
  3. ¿Publicar paquetes públicos o privados por defecto? → `access: "public" | "restricted"`.
  4. ¿Rama base? → `baseBranch` (por defecto `main`).
- Copia `.changeset/README.md` (una guía breve que enlaza a https://changesets.dev y al FAQ; fuente: https://github.com/changesets/changesets/blob/main/packages/cli/default-files/README.md).
- El `config.json` escrito incluye `"$schema": "https://unpkg.com/@changesets/config@<versión>/schema.json"` más `baseBranch`, `access`, `format`, `changelog`, `commit`, `ignore`, `fixed`, `linked`, `updateInternalDependencies` (https://github.com/changesets/changesets/blob/main/packages/config/src/defaults.ts).

---

## 2. Opciones de `.changeset/config.json`

Config por defecto documentada (https://changesets.dev/guide/config):

```json
{
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "restricted",
  "baseBranch": "main",
  "changedFilePatterns": ["**"],
  "format": "auto",
  "privatePackages": { "version": false, "tag": false },
  "updateInternalDependencies": "patch",
  "ignore": [],
  "bumpVersionsWithWorkspaceProtocolOnly": false
}
```

| Opción                                              | Tipo / default                                                                                                                                                                                 | Qué hace                                                                                                                                                                                                                                                                      | Recomendado para este repo                                                                                                                                                                      |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `changelog`                                         | `false \| string \| [string, opts]`, default `"@changesets/cli/changelog"` (alias de `@changesets/changelog-git`, añade enlaces a commits) (https://changesets.dev/guide/config#changelog)     | Generador de CHANGELOG. `false` desactiva changelogs.                                                                                                                                                                                                                         | `["@changesets/changelog-github", { "repo": "joansebgarcia/skills" }]` si se quiere enlazar PRs/autores; si no, dejar el default. Ver sección 3.                                                |
| `commit`                                            | `boolean \| string \| [string, opts]`, default `false` (https://changesets.dev/guide/config#commit)                                                                                            | Si `add` y `version` hacen commit automático. `true` = `["@changesets/cli/commit", { "skipCI": "version" }]`.                                                                                                                                                                 | `false` (la Action hace el commit del PR de versión). Ojo: `skipCI` puede añadir `[skip ci]` y bloquear workflows (https://changesets.dev/guide/automating#run-github-actions-for-version-prs). |
| `fixed`                                             | `string[][]`, default `[]`, solo monorepos, admite picomatch (https://changesets.dev/guide/config#fixed)                                                                                       | Grupos que siempre se bumpean y publican juntos con la misma versión.                                                                                                                                                                                                         | `[]` (un solo paquete).                                                                                                                                                                         |
| `linked`                                            | `string[][]`, default `[]`, solo monorepos (https://changesets.dev/guide/config#linked)                                                                                                        | Grupos que "comparten" la versión más alta, pero solo se bumpean los que tienen cambios.                                                                                                                                                                                      | `[]`.                                                                                                                                                                                           |
| `access`                                            | `"restricted" \| "public"`, default `"restricted"` (https://changesets.dev/guide/config#access)                                                                                                | Acceso al publicar en npm. Se puede sobrescribir por paquete con `publishConfig.access`. En v3 ya no se acepta `"private"` (https://changesets.dev/guide/migration#replace-access-private-with-restricted).                                                                   | Irrelevante si el paquete es `private: true`. Si se publica con scope: `"public"`.                                                                                                              |
| `baseBranch`                                        | `string`, default `"main"` (cambió en v3; https://changesets.dev/guide/migration#set-basebranch-if-your-default-branch-is-not-main)                                                            | Rama contra la que se detectan cambios; `--since` la sobrescribe.                                                                                                                                                                                                             | `"main"` (explícito por claridad).                                                                                                                                                              |
| `changedFilePatterns`                               | `string[]`, default `["**"]` (https://changesets.dev/guide/config#changedfilepatterns)                                                                                                         | Qué archivos cambiados marcan un paquete como cambiado.                                                                                                                                                                                                                       | Opcional: `["skills/**"]` si solo las skills deben considerarse cambios versionables.                                                                                                           |
| `format`                                            | `"auto" \| "prettier" \| "oxfmt" \| "deno" \| "dprint" \| false`, default `"auto"` (https://changesets.dev/guide/config#format)                                                                | Formateador de changesets y changelogs; reemplaza a `prettier` de v2 (https://changesets.dev/guide/migration#replace-prettier-with-format).                                                                                                                                   | `"auto"`.                                                                                                                                                                                       |
| `privatePackages`                                   | `{ version?, tag? } \| boolean`, default `{ "version": false, "tag": false }` (https://changesets.dev/guide/config#privatepackages)                                                            | Si se versionan (`version`) y etiquetan en git (`tag`, durante `publish`) los paquetes `private: true`. `true` equivale a ambos en `true`. **Cambió en v3**: en v2 se versionaban por defecto (https://changesets.dev/guide/migration#opt-in-to-versioning-private-packages). | **`{ "version": true, "tag": true }`** si el repo no se publica en npm (ver sección 4).                                                                                                         |
| `updateInternalDependencies`                        | `"patch" \| "minor"`, default `"patch"`, solo monorepos (https://changesets.dev/guide/config#updateinternaldependencies)                                                                       | Cuándo actualizar el rango de dependencias internas.                                                                                                                                                                                                                          | Default (no aplica).                                                                                                                                                                            |
| `ignore`                                            | `string[]`, default `[]`, solo monorepos, picomatch (https://changesets.dev/guide/config#ignore)                                                                                               | Excluye paquetes temporalmente de la publicación. Para no publicar nunca, usar `"private": true`.                                                                                                                                                                             | `[]`.                                                                                                                                                                                           |
| `bumpVersionsWithWorkspaceProtocolOnly`             | `boolean`, default `false`, solo monorepos (https://changesets.dev/guide/config#bumpversionswithworkspaceprotocolonly)                                                                         | Solo bumpea rangos con `workspace:`.                                                                                                                                                                                                                                          | Default.                                                                                                                                                                                        |
| `snapshot.useCalculatedVersion`                     | `boolean`, default `false` (https://changesets.dev/guide/config#snapshot-usecalculatedversion)                                                                                                 | En snapshots usa la versión calculada en vez de `0.0.0`. Reemplaza al experimental `useCalculatedVersionForSnapshots` de v2.                                                                                                                                                  | No definir.                                                                                                                                                                                     |
| `snapshot.prereleaseTemplate`                       | `string`, default `"{tag}-{datetime}"` (o `"{datetime}"` sin tag) (https://changesets.dev/guide/config#snapshot-prereleasetemplate)                                                            | Sufijo de snapshots. Placeholders: `{tag}`, `{commit}`, `{commit-short}`, `{timestamp}`, `{datetime}`.                                                                                                                                                                        | No definir.                                                                                                                                                                                     |
| `___experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH` | `updateInternalDependents: "out-of-range" \| "always"`, `onlyUpdatePeerDependentsWhenOutOfRange: boolean` (https://changesets.dev/guide/config#experimentalunsafeoptions-will-change-in-patch) | Opciones experimentales que pueden cambiar en un patch.                                                                                                                                                                                                                       | No usar.                                                                                                                                                                                        |

Nota: la opción `prettier` solo existe en la doc antigua (https://github.com/changesets/changesets/blob/main/docs/config-file-options.md); en v3 se usa `format`.

---

## 3. Flujo de trabajo y comandos

Ciclo general: (1) añadir changesets con cada cambio, (2) `version` cuando se quiera hacer release, revisar, (3) `publish` (https://github.com/changesets/changesets/blob/main/docs/intro-to-using-changesets.md, https://changesets.dev/guide/versioning-and-publishing).

### Qué es un changeset

Un archivo Markdown con frontmatter YAML: el frontmatter indica paquetes y tipo de bump semver (`major` | `minor` | `patch`), y el cuerpo es el resumen que va al CHANGELOG (https://changesets.dev/guide/getting-started#what-is-a-changeset):

```md
---
"@joansebgarcia/skills": minor
---

Añade la skill `research`.
```

No todo cambio necesita changeset; la doc **no recomienda** bloquear contribuciones por no tenerlo (https://changesets.dev/guide/getting-started#usage).

### Comandos (https://changesets.dev/guide/cli; flags verificados en https://github.com/changesets/changesets/blob/main/packages/cli/src/cli.ts)

- **`changeset` / `changeset add`**: interactivo; pregunta paquetes, bump por paquete y resumen, y escribe el archivo en `.changeset/`. Flags: `--empty` (changeset vacío, sin release), `--open` (abrir en el editor), `--since <ref>`, `-m/--message <text>`, `--major/--minor/--patch <pkg>` (no interactivo).
- **`changeset version`**: consume los changesets, actualiza versiones y dependencias, y escribe los CHANGELOG. Flags: `--ignore <pkg>`, `--snapshot [name]`, `--snapshot-prerelease-template <tpl>`. **En v3 sale con código 1 si no hay changesets** (https://changesets.dev/guide/migration#handle-version-command-returning-exit-code-1-when-there-is-nothing-to-release).
- **`changeset publish`**: para cada paquete cuya versión no esté en npm, ejecuta el publish del gestor detectado y crea tags git (`pkg@X.Y.Z` en monorepos, **`vX.Y.Z` en repos de un solo paquete**). Flags: `--otp`, `--tag <dist-tag>`, `--from-pack-dir <dir>`, `--no-git-tag`. Hay que hacer `git push --follow-tags` después. Asume que el último commit es el de versión. Aviso: puede publicar paquetes nuevos accidentalmente; usar `"private": true` en lo que no deba publicarse (https://changesets.dev/guide/cli#publish).
- **`changeset status`**: muestra los changesets existentes; sale con 1 si hay paquetes cambiados sin changesets. Flags: `--since`, `-v`, `-o/--output <file>` (JSON) (https://changesets.dev/guide/cli#status).
- **`changeset publish-plan`**, **`changeset pack --out-dir <dir>`**: nuevos en v3 para separar build/pack/publish en CI (https://changesets.dev/guide/cli#publish-plan, https://changesets.dev/guide/cli#pack).
- **`changeset git-tag`**: crea tags sin publicar (antes `tag`; renombrado en v3, `tag` sigue como alias en el código) (https://changesets.dev/guide/migration#rename-tag-command-to-git-tag).
- **`changeset pre enter <tag>` / `pre exit`**: ver sección 5.
- **`changeset init`**: ver sección 1.

### Cómo se generan las entradas del CHANGELOG

Durante `version`, el generador configurado en `changelog` llama a `getReleaseLine(changeset, type, opts)` por cada changeset y a `getDependencyReleaseLine` para bumps de dependencias; reciben el resumen, los releases y el hash del commit que introdujo el changeset (https://changesets.dev/guide/customize-changelog-format).

- Default `@changesets/cli/changelog` = `@changesets/changelog-git`: añade enlaces a commits (https://changesets.dev/guide/config#changelog).
- **`@changesets/changelog-github`** (https://github.com/changesets/changesets/blob/main/packages/changelog-github/README.md): enlaza PR, commit y autor (`Thanks @user!`). Requiere un `GITHUB_TOKEN` con permisos `read:user` y `repo:status` (en Actions como secret; en local vía `.env`). Opciones: `repo` (default `process.env.GITHUB_REPOSITORY`, obligatorio si se ejecuta en local), `disableThanks` (default `false`) y `template` (experimental; tokens `{summary}`, `{ref}`, `{pull}`, `{commit}`, `{authors}`). Hay que instalarlo como devDependency.

---

## 4. Paquete único vs monorepo, y paquetes privados

- Changesets funciona en monorepos y en repos de un solo paquete (https://changesets.dev/guide/getting-started#what-is-changesets). Las opciones `fixed`, `linked`, `ignore`, `updateInternalDependencies` y `bumpVersionsWithWorkspaceProtocolOnly` solo aplican en monorepos (https://changesets.dev/guide/config).
- Diferencia visible: formato de tags git, `vX.Y.Z` en paquete único frente a `nombre@X.Y.Z` en monorepo (https://changesets.dev/guide/cli#publish).
- **Versionar sin publicar en npm** ("Beyond npm", https://changesets.dev/guide/beyond-npm): el proyecto necesita un `package.json` con al menos `name`, `private: true` y `version`; luego `privatePackages.version: true` para versionar y `privatePackages.tag: true` para crear tags git en `changeset publish`. Esto permite el flujo "solo tags git" en CI (https://changesets.dev/guide/automating#publish-git-tags-only).
- Para que un paquete no se publique nunca: `"private": true` (no `ignore`, que es temporal) (https://changesets.dev/guide/config#ignore).

**Aplicación a este repo:** es un repo de un solo paquete. Si las skills se consumen desde GitHub (copiando a `.agents/`) y no desde npm, lo adecuado es `"private": true` + `privatePackages: { version: true, tag: true }`.

---

## 5. Prereleases y snapshots

### Prereleases (https://changesets.dev/guide/prereleases)

- La doc advierte que son complejas y que los errores pueden dejar estados difíciles de arreglar.
- `npx changeset pre enter beta` crea `.changeset/pre.json`. Luego `version` y `publish` como siempre: las versiones serán `X.Y.Z-beta.N` y se publican con el dist-tag `beta`. Para cambiar de etapa (alpha → beta → rc), editar `"tag"` en `pre.json`.
- En v3, tras `version` los changesets se mueven a `.changeset/pre/` (no se borran) y se usan en el changelog estable; se pueden editar o borrar antes de salir (https://changesets.dev/guide/migration#versioned-changesets-are-moved-to-changeset-pre).
- `npx changeset pre exit` marca la intención de salir; el siguiente `version` + `publish` genera la versión estable con el dist-tag `latest`.
- Si se hace en otra rama (p. ej. `next`), cambiar `baseBranch` y habilitar el workflow de CI para esa rama, y revertirlo antes de mergear.
- Un paquete **nuevo** publicado en modo pre también recibe `latest` (npm exige que exista).
- No se puede entrar en modo pre solo para algunos paquetes.

### Snapshots (https://changesets.dev/guide/snapshot-releases)

- `npx changeset version --snapshot [nombre]` → versiones `0.0.0-{nombre}-{datetime}` (base `0.0.0` salvo que se use `snapshot.useCalculatedVersion`).
- `npx changeset publish --tag <nombre> --no-git-tag`. **Usar siempre `--tag`** para no mover `latest`.
- No hacer push de los cambios de un snapshot a ninguna rama que se vaya a mergear.
- Alternativa que menciona la doc: pkg.pr.new.

Ninguna de las dos aplica a este repo si no se publica en npm.

---

## 6. Automatización en CI con `changesets/action@v2`

### Piezas (https://github.com/changesets/action/blob/main/README.md)

- `changesets/action`: todo en uno (select-mode + version + publish). Inputs: `github-token`, `publish-script`, `version-script`, `commit-message`/`pr-title` (default `Version Packages`), `pr-draft`, `pr-base-branch`, `create-github-releases` (default `true`), `push-git-tags` (default `true`), `push-with-git-cli` (default `false`; por defecto se usa la API de GitHub y los commits/tags quedan firmados), `cwd`. Outputs: `published`, `published-packages`, `has-changesets`, `pr-number`.
- Subacciones: `select-mode` (devuelve `version` | `publish` | `none`), `version` (crea/actualiza el PR "Version Packages"), `pack`, `publish` (`create-github-releases` default `true`), `pr-status` y `pr-comment` (https://github.com/changesets/action/blob/main/select-mode/README.md, https://github.com/changesets/action/blob/main/version/README.md, https://github.com/changesets/action/blob/main/publish/README.md).
- Lógica recomendada: si hay changesets, versionar y crear/actualizar el PR; si no los hay y hay algo publicable, publicar; si no, no hacer nada (https://changesets.dev/guide/automating#how-do-i-run-the-version-and-publish-commands).

### Permisos y configuración

- Permisos del job: `contents: write` (commit de versión, releases), `pull-requests: write` (crear el PR) e `id-token: write` solo si se usa trusted publishing (https://github.com/changesets/action/blob/main/README.md#requirements).
- En el repo, **Settings > Actions > General > "Allow GitHub Actions to create and approve pull requests"** debe estar activado; si no, aparecen errores como `GitHub Actions is not permitted to create or approve pull requests` (https://changesets.dev/guide/automating#prerequisites).
- Recomendaciones de la doc: `permissions: {}` a nivel de workflow con permisos por job, `concurrency` por rama, `persist-credentials: false` en checkout, `package-manager-cache: false` en setup-node (para evitar cache poisoning) (snippets en https://github.com/changesets/changesets/tree/main/site/guide/_snippets).
- Los PRs creados con el `GITHUB_TOKEN` no disparan otros workflows; para que corra CI en el PR de versión hay que usar un PAT o un token de GitHub App (`actions/create-github-app-token`) pasado en `github-token` (https://changesets.dev/guide/automating#run-github-actions-for-version-prs). Poner `GITHUB_TOKEN` como variable de entorno no configura la Action; hay que pasarlo por el input (https://github.com/changesets/action/blob/main/README.md#usage).

### Autenticación con npm

- **Trusted Publishing (recomendado)**: la doc lo recomienda; dice que Staged Publishing aún no funciona con Changesets. `id-token: write` solo en el job que publica (https://changesets.dev/guide/automating#trusted-publishing). Para un paquete nuevo hay que publicarlo antes a mano (stub `0.0.0`), configurar trusted publishing en npm y luego crear un changeset para la versión inicial (https://changesets.dev/guide/versioning-and-publishing#trusted-publishing). Se puede usar un `environment` con revisores obligatorios para aprobar la publicación.
- **Por token (`NPM_TOKEN`) ya no se recomienda**: los tokens granulares caducan como mucho a los 90 días y los tokens con bypass de 2FA están en deprecación. Solo se mantiene para registries sin trusted publishing. Uso: secret `NPM_TOKEN`, `actions/setup-node` con `registry-url`, y `NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}` (https://changesets.dev/guide/automating#token-based-publishing).
- La documentación de Changesets no menciona "provenance" explícitamente; lo que documenta es trusted publishing.

### Ejemplo oficial (trusted publishing, resumido)

Archivo completo: https://github.com/changesets/changesets/blob/main/site/guide/_snippets/automating-trusted-publishing.yaml. Jobs: `select-mode` (contents: read) → `version` (contents + pull-requests: write) **o** `pack` (contents: read, hace build) → `publish` (contents: write, id-token: write) usando `changesets/action/publish@v2` con `pack-dir-artifact-id`.

Detalles de los snippets oficiales que hay que tener en cuenta:

- Usan `run: npm build`, que no es un comando válido de npm (sería `npm run build`); adaptarlo.
- `automating-version-only.yaml` usa `changesets/action@v2` con `NODE_AUTH_TOKEN`, lo que no encaja con un flujo de "solo versionar" (parece copiado del ejemplo con token). Para ese caso usar `changesets/action/version@v2`.

### Comprobar si los PRs traen changesets

- No bloqueante: Changeset bot (https://github.com/apps/changeset-bot) o `pr-status` + `pr-comment`. Si se usa `pull_request_target`, no ejecutar código del fork (https://changesets.dev/guide/automating#non-blocking).
- Bloqueante (no recomendado): `npx changeset status --since main` en CI; `changeset --empty` para PRs sin release (https://changesets.dev/guide/automating#blocking).

---

## 7. Configuración recomendada para ESTE repo, paso a paso

Supuesto principal (**hay que confirmarlo**): las skills se distribuyen desde GitHub y no se publican en npm. Así se obtienen versiones semver, `CHANGELOG.md`, tags `vX.Y.Z` y GitHub Releases sin publicar en npm. Al final hay una variante por si se publica.

### Paso 1: ajustar `package.json` (no modificado en esta investigación)

```jsonc
{
  "name": "@joansebgarcia/skills",
  "version": "1.0.0", // o bajar a 0.x; el primer changeset decide el bump
  "private": true, // imprescindible: evita publicar en npm por accidente
  "scripts": {
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "changeset publish", // con private:true + privatePackages.tag solo crea el tag git
  },
}
```

Quitar `"main": "index.js"` (el archivo no existe) y el script `test` de relleno. El resto de campos de `npm init` (`repository`, `bugs`, `homepage`, `license`…) puede quedarse. Justificación de `private` + scripts: https://changesets.dev/guide/beyond-npm, https://changesets.dev/guide/cli#publish.

### Paso 2: instalar e inicializar

```bash
npm install --save-dev @changesets/cli @changesets/changelog-github
npx changeset init
```

Commitear `package-lock.json` (la CI usa `npm ci`, que lo exige).

### Paso 3: `.changeset/config.json`

```json
{
  "$schema": "https://unpkg.com/@changesets/config@4.0.1/schema.json",
  "baseBranch": "main",
  "access": "restricted",
  "format": "auto",
  "changelog": ["@changesets/changelog-github", { "repo": "joansebgarcia/skills" }],
  "commit": false,
  "ignore": [],
  "fixed": [],
  "linked": [],
  "updateInternalDependencies": "patch",
  "privatePackages": { "version": true, "tag": true }
}
```

Opcional: `"changedFilePatterns": ["skills/**"]` para que solo los cambios en skills cuenten. Si no se quiere el token para `changelog-github` en local, usar `"changelog": "@changesets/cli/changelog"`.

### Paso 4: activar el permiso en GitHub

Settings > Actions > General > "Allow GitHub Actions to create and approve pull requests" (https://changesets.dev/guide/automating#prerequisites).

### Paso 5: `.github/workflows/release.yml` (basado en el snippet oficial "Publish Git Tags Only")

Base: https://github.com/changesets/changesets/blob/main/site/guide/_snippets/automating-publish-git-tags-only.yaml, que ya usa npm. Cambios respecto al snippet: `npm ci` en vez de `npm install` (instalación reproducible a partir del lockfile), `package-manager-cache: false` también en `select-mode`, y se quita el paso `npm build` (no es un comando válido y aquí no hay nada que compilar).

```yaml
name: Release

on:
  push:
    branches:
      - main

permissions: {}

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}

jobs:
  select-mode:
    runs-on: ubuntu-latest
    outputs:
      mode: ${{ steps.select-mode.outputs.mode }}
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false
      - uses: actions/setup-node@v7
        with:
          node-version: 24
          package-manager-cache: false
      - run: npm ci
      - id: select-mode
        uses: changesets/action/select-mode@v2

  version:
    if: needs.select-mode.outputs.mode == 'version'
    needs: select-mode
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v7
        with:
          persist-credentials: false
      - uses: actions/setup-node@v7
        with:
          node-version: 24
          package-manager-cache: false
      - run: npm ci
      - name: Version packages (abre/actualiza el PR "Version Packages")
        uses: changesets/action/version@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # lo usa @changesets/changelog-github

  publish:
    if: needs.select-mode.outputs.mode == 'publish'
    needs: select-mode
    runs-on: ubuntu-latest
    permissions:
      contents: write # tags + GitHub Releases
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-node@v7
        with:
          node-version: 24
          package-manager-cache: false
      - run: npm ci
      - name: Crear tag vX.Y.Z y GitHub Release (sin npm)
        uses: changesets/action/publish@v2
```

Nota: la variable `GITHUB_TOKEN` del paso `version` es para el generador `@changesets/changelog-github` (https://github.com/changesets/changesets/blob/main/packages/changelog-github/README.md). La Action recibe su token por el input `github-token`, que por defecto es `github.token`. No he verificado contra el código de la Action si el entorno del paso llega al proceso `changeset version`. Si los enlaces a PRs/autores salen vacíos, revisar ese punto.

### Paso 6: uso diario

1. En cada PR con cambios en skills: `npx changeset` (o `npm run changeset`) → elegir `patch`/`minor`/`major` y escribir el resumen → commitear el `.changeset/*.md`.
2. Al mergear en `main`, la Action abre o actualiza el PR **"Version Packages"** (bump de `package.json` y `CHANGELOG.md`).
3. Al mergear ese PR, el workflow entra en modo `publish`: crea el tag `vX.Y.Z` y el GitHub Release. No publica en npm porque el paquete es `private`.
4. Opcional: instalar el Changeset bot para recordar changesets sin bloquear PRs.

### Variante: si se publica en npm

- El nombre `@joansebgarcia/skills` ya tiene scope y está libre en npm. Quitar `"private": true`, definir `"files"` (p. ej. `["skills"]`) y `"access": "public"` en `.changeset/config.json` (o `publishConfig.access: "public"`); sin eso, un paquete con scope se publica como restringido (https://changesets.dev/guide/versioning-and-publishing#publishing-for-the-first-time, https://changesets.dev/guide/config#access).
- Publicar un stub `0.0.0` a mano, configurar Trusted Publishing en npmjs.com apuntando a `joansebgarcia/skills` y al archivo `release.yml`, y crear un changeset `minor` como "Initial release" (https://changesets.dev/guide/versioning-and-publishing#trusted-publishing).
- Workflow: el snippet de trusted publishing (select-mode → version | pack → publish con `id-token: write` solo en `publish`), con los mismos ajustes de npm que arriba (`npm ci`, y `npm run build` solo si hay build) (https://changesets.dev/guide/automating#trusted-publishing). Sin `NPM_TOKEN`.
- `privatePackages` deja de importar; se puede volver al default.

---

## Preguntas abiertas

1. ¿Se va a publicar en npm o solo se distribuye desde GitHub? Decide entre la configuración del paso 7 y la variante.
2. ¿Versión inicial: mantener `1.0.0` o empezar en `0.x`?
3. ¿Changelog con enlaces a PRs y autores (`changelog-github`, requiere token) o solo commits (default)?
4. ¿Una sola versión para todo el repo o versiones por skill? Lo segundo implica convertir el repo en un monorepo con npm workspaces (campo `"workspaces"` en el `package.json` raíz y un `package.json` por skill), con tags `nombre@X.Y.Z` y opciones como `fixed`/`linked`.
5. ¿Hace falta que la CI corra sobre el PR "Version Packages"? Si sí, hay que usar un token de GitHub App o un PAT.
