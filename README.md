# 📘 xcraft-guldan

> Gul'dan is a powerful orcish warlock in the Warcraft universe who made a pact with the demonic Burning Legion, corrupting the orcish Horde and playing a pivotal role in their invasion of Azeroth. His pursuit of power led to the transformation of Draenor into Outland, and he remains a recurring villain associated with fel magic and manipulation in the Warcraft lore.

## Aperçu

`xcraft-guldan` est un utilitaire en ligne de commande (CLI) pour l'écosystème Xcraft, conçu pour analyser récursivement les dépendances entre projets Visual Studio (`.vcxproj`, `.sln`, `.props`, `.targets`, `.msbuildproj`, etc.). Il permet de cartographier l'arborescence des projets et leurs interdépendances, ce qui est particulièrement utile pour comprendre et auditer la structure de grands projets C++/C# multi-modules.

## Sommaire

- [Structure du module](#structure-du-module)
- [Fonctionnement global](#fonctionnement-global)
- [Exemples d'utilisation](#exemples-dutilisation)
- [Détails des sources](#détails-des-sources)
- [Licence](#licence)

## Structure du module

Le module est structuré comme un outil CLI minimal avec les composants suivants :

- **`bin/guldan`** — Point d'entrée exécutable contenant l'intégralité de la logique d'analyse.
- **`.eslintrc.js`** — Configuration ESLint pour le développement du module.
- **`package.json`** — Manifeste du module avec ses dépendances.

Dépendances principales :

- [`cheerio`](https://cheerio.js.org/) — Analyse et parcours de contenu XML des fichiers de projet MSBuild.
- [`vs-parse`](https://www.npmjs.com/package/vs-parse) — Analyse des fichiers de solution Visual Studio (`.sln`).
- [`yargs`](https://yargs.js.org/) — Construction de l'interface en ligne de commande.

## Fonctionnement global

`xcraft-guldan` réalise une analyse statique récursive de fichiers de projet MSBuild. À partir d'un fichier d'entrée, il parcourt toutes les références et importations pour construire un graphe complet des dépendances.

### Étapes de traitement

1. **Détermination du `bundleDir`** — L'outil identifie le répertoire racine du bundle en remontant l'arborescence à la recherche d'un répertoire `zou` (deux ou trois niveaux au-dessus du projet fourni), sinon il retombe sur le répertoire du projet lui-même.
2. **Analyse du fichier racine** — Le fichier de projet fourni en argument est lu et parsé via `cheerio` en mode XML.
3. **Extraction des propriétés MSBuild** — Les balises `<PropertyGroup>` sont extraites pour permettre la résolution des variables (`$(BundleDir)`, `$(ZouDir)`, `$(V)`, etc.).
4. **Résolution récursive des dépendances** — Les quatre types de références suivants sont explorés :
   - `<ProjectReference Include="...">` dans les `<ItemGroup>` (fichiers `.vcxproj`)
   - `<Import Project="...">` dans les `<ImportGroup>` (fichiers `.props`, `.targets`)
   - `<Import Project="...">` directement sous `<Project>`
   - `<ImportProject Include="...">` dans les `<ItemGroup>` (fichiers `.msbuildproj`, `.sln`)
5. **Normalisation des chemins** — Les variables MSBuild sont substituées et les séparateurs de chemin sont adaptés à la plateforme courante.
6. **Affichage des résultats** — Les dépendances résolues sont affichées selon les options passées en argument, et les fichiers manquants (si demandé) sont listés sur `stderr`.

Les fichiers de solution (`.sln`) sont traités par un chemin dédié : `vs-parse` extrait la liste des projets référencés (avec l'option `deepParse`), puis chacun est résolu récursivement comme les autres imports.

### Résolution des variables de chemin

| Variable MSBuild | Résolution                                      |
| ----------------- | ------------------------------------------------ |
| `$(BundleDir)`    | Répertoire racine du bundle (auto-détecté)        |
| `$(ZouDir)`       | `<bundleDir>/zou/`                                |
| `$(V)`            | Séparateur de chemin OS (`/` ou `\`)              |
| Autres `$(VAR)`   | Valeur extraite des `<PropertyGroup>` du projet   |

Un registre `dependencies` (indexé par chemin de fichier absolu) empêche de retraiter deux fois le même fichier et protège ainsi contre les cycles d'imports.

## Exemples d'utilisation

```bash
# Afficher toutes les dépendances d'un projet (chemins relatifs au bundle)
guldan path/to/myProject.vcxproj

# Afficher les projets dont les fichiers sont introuvables
guldan -m path/to/myProject.vcxproj

# Afficher les chemins complets absolus
guldan -p path/to/myProject.vcxproj

# Afficher uniquement les noms de répertoires de premier niveau
guldan -d path/to/myProject.vcxproj

# Combiner les options
guldan -m -d path/to/myProject.sln

# Afficher l'aide
guldan -h
```

### Options disponibles

| Option           | Alias | Description                                                                        |
| ----------------- | ----- | ----------------------------------------------------------------------------------- |
| `--show-missing`  | `-m`  | Affiche sur `stderr` les fichiers de projet référencés mais introuvables            |
| `--show-full`     | `-p`  | Affiche les chemins complets absolus plutôt que les chemins relatifs au bundle      |
| `--dirname-only`  | `-d`  | Affiche uniquement le nom du répertoire de premier niveau (sans le chemin complet)  |
| `--help`          | `-h`  | Affiche l'aide                                                                       |

## Détails des sources

### `bin/guldan`

Point d'entrée de l'application CLI. Ce fichier unique contient l'intégralité de la logique d'analyse et d'affichage.

#### Méthodes publiques

- **`resolveDependencies(projectFile)`** — Fonction principale d'analyse récursive. Lit le fichier de projet, extrait les propriétés MSBuild, puis parcourt les quatre types de références supportés. Si le fichier est absent ou est un répertoire, il est enregistré dans `missing` (si l'option `-m` est active), et la fonction s'arrête. Évite les cycles grâce au registre `dependencies` qui court-circuite les fichiers déjà traités. Traite spécifiquement les fichiers `.sln` via `vs-parse`.
- **`resolveImported(importedFile, projectFile)`** — Résout le chemin d'un fichier importé relativement au fichier courant, puis déclenche `resolveDependencies` sur ce fichier.
- **`normalizePath(filePath, currentFile)`** — Normalise un chemin MSBuild en substituant les variables (`$(BundleDir)`, `$(ZouDir)`, `$(V)` et toute variable présente dans les `<PropertyGroup>`), et en adaptant les séparateurs de chemin à la plateforme.
- **`resolveImportedFilePath(importedFile, currentFile)`** — Combine `path.dirname(currentFile)` et le résultat de `normalizePath` pour obtenir le chemin absolu d'un fichier importé via `path.resolve`.

### `.eslintrc.js`

Configuration ESLint pour le développement du module. Points notables :

- Support ES2022, JSX/React et environnements `browser`, `node` et `mocha`.
- Plugins `react`, `babel` et `jsdoc` activés.
- La règle `no-unused-vars` tolère les variables ignorées dans la destructuration d'arrays (préfixe `^_`) et ignore les arguments de fonctions.
- `react/display-name` désactivé.

## Licence

Ce module est distribué sous [licence MIT](./LICENSE).

---

_Ce contenu a été généré par IA_