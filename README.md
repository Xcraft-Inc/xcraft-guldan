# 📘 Documentation du module xcraft-guldan

# Gul'dan

> Gul'dan is a powerful orcish warlock in the Warcraft universe
> who made a pact with the demonic Burning Legion, corrupting the
> orcish Horde and playing a pivotal role in their invasion of
> Azeroth. His pursuit of power led to the transformation of Draenor
> into Outland, and he remains a recurring villain associated
> with fel magic and manipulation in the Warcraft lore.

## Aperçu

xcraft-guldan est un utilitaire en ligne de commande pour l'écosystème Xcraft, conçu pour analyser les dépendances entre projets Visual Studio (.vcxproj, .sln, etc.). Il permet de visualiser l'arborescence des projets et leurs interdépendances, ce qui est particulièrement utile pour comprendre la structure de grands projets.

## Structure du module

Le module est structuré comme un outil en ligne de commande (CLI) avec les caractéristiques suivantes:

- Point d'entrée via le binaire `bin/guldan`
- Utilisation de `cheerio` pour l'analyse XML des fichiers de projet
- Utilisation de `vs-parse` pour l'analyse de fichiers de solution Visual Studio (.sln)
- Interface de ligne de commande construite avec `yargs`

## Fonctionnement global

xcraft-guldan analyse récursivement les fichiers de projet Visual Studio pour identifier toutes les dépendances. Il parcourt les références de projet, les importations et autres liens entre fichiers pour construire une carte complète des dépendances.

Le processus fonctionne comme suit:

1. Analyse du fichier de projet initial (fourni en argument)
2. Identification du répertoire racine du bundle
3. Extraction des propriétés du projet pour résoudre les variables
4. Analyse récursive de toutes les références et importations
5. Affichage des résultats selon les options spécifiées

## Exemples d'utilisation

```bash
# Afficher toutes les dépendances d'un projet
guldan path/to/myProject.vcxproj

# Afficher les dépendances manquantes
guldan -m path/to/myProject.vcxproj

# Afficher les chemins complets des projets
guldan -p path/to/myProject.vcxproj

# Afficher uniquement les noms de répertoires
guldan -d path/to/myProject.vcxproj

# Afficher l'aide
guldan -h
```

## Interactions avec d'autres modules

Ce module est principalement un outil autonome qui n'interagit pas directement avec d'autres modules Xcraft pendant son exécution. Il est conçu pour analyser la structure des projets qui peuvent inclure des modules Xcraft.

## Détails des sources

### `bin/guldan`

Ce fichier est le point d'entrée de l'application CLI. Il contient la logique principale pour:

- Analyser les fichiers de projet Visual Studio (.vcxproj, .sln, etc.)
- Résoudre les chemins relatifs et les variables
- Extraire les références entre projets
- Afficher les résultats selon différentes options

Les fonctions principales sont:

- `resolveDependencies`: Analyse récursivement un fichier de projet pour trouver toutes ses dépendances
- `resolveImported`: Traite les fichiers importés et ajoute leurs dépendances
- `normalizePath`: Normalise les chemins de fichiers en remplaçant les variables et en ajustant les séparateurs
- `resolveImportedFilePath`: Résout le chemin complet d'un fichier importé

L'outil prend en charge plusieurs types de références:

- Références de projet dans les fichiers .vcxproj
- Importations dans les groupes d'importation
- Importations directes au niveau du projet
- Projets importés dans les groupes d'éléments

### `.eslintrc.js`

Ce fichier configure ESLint pour le projet avec les caractéristiques suivantes:

- Support pour ES2022
- Support pour JSX et React
- Règles de documentation JSDoc
- Configuration spécifique pour ignorer certaines variables non utilisées dans la destructuration

### `package.json`

Le fichier package.json définit:

- Le nom du module: `xcraft-guldan`
- Version: `1.0.1`
- Point d'entrée binaire: `bin/guldan`
- Dépendances principales:
  - `cheerio`: Pour l'analyse de contenu XML
  - `vs-parse`: Pour l'analyse de fichiers Visual Studio
  - `yargs`: Pour la création d'interfaces en ligne de commande
- Dépendances de développement liées à l'écosystème Xcraft et à la mise en forme du code

_Cette documentation a été mise à jour automatiquement._