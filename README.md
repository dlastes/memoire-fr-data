# memoire-fr-data

Bases nominatives "Morts pour la France" compilées en SQLite, prêtes à être téléchargées
à la demande par une application mobile offline-first (pas de backend). Ce dépôt ne
contient que des données — aucun code applicatif.

Consommateur : [`morts-pour-la-france`](https://github.com/dlastes/morts-pour-la-france)
(app Expo/React Native), distribution via [jsDelivr](https://www.jsdelivr.com/) sur ce dépôt.

## Source et licence

Données extraites de [Mémoire des Hommes](https://www.memoiredeshommes.defense.gouv.fr/conflits-operations/telechargement-des-bases)
(ministère des Armées et des Anciens Combattants), licence ouverte Etalab — réutilisation
libre y compris commerciale, à condition de mentionner la source :

> Site *Mémoire des hommes*, données téléchargées sur
> https://www.memoiredeshommes.defense.gouv.fr/conflits-operations/telechargement-des-bases,
> mise à jour du 21 janvier 2026.

**Dernière compilation de ce dépôt : 25 juillet 2026.** Les fichiers ci-dessous sont un
sous-ensemble de champs des bases officielles (voir schéma plus bas), pas une copie brute.

## Découpage des fichiers — limite réelle : jsDelivr, pas GitHub

Chaque base est découpée en fichiers **de moins de 20 Mo**. Ce n'est pas la limite de
GitHub (100 Mo sans Git LFS) mais celle du **CDN jsDelivr** qui sert ces fichiers à l'app :
`cdn.jsdelivr.net` renvoie une erreur 403 *"File size exceeded the configured limit of 20 MB"*
au-delà — vérifié par un vrai fetch, pas une supposition. Un premier découpage visant
uniquement la limite GitHub (fichiers de 25-70 Mo) a été poussé puis rejeté par ce test et
corrigé.

`manifest.json` liste, pour chaque base, ses fichiers avec le nombre de lignes et le
premier/dernier nom (tri alphabétique) qu'ils couvrent — l'app l'utilise pour router une
recherche par nom vers le ou les bons fichiers sans avoir à tous les télécharger.

| Base | Fichiers | Lignes | Poids total |
|---|---:|---:|---:|
| `1418-00.sqlite3` … `1418-23.sqlite3` (24 fichiers) | 24 | 1 417 220 | 354,7 Mo |
| `3945-00.sqlite3` … `3945-03.sqlite3` (4 fichiers) | 4 | 215 108 | 50,3 Mo |
| `indochine.sqlite3` | 1 | 39 327 | 9,8 Mo |
| `coree.sqlite3` | 1 | 289 | 0,1 Mo |
| `algerie-maroc-tunisie.sqlite3` | 1 | 26 008 | 4,1 Mo |
| `opex-militaires.sqlite3` | 1 | 696 | 0,2 Mo |
| `opex-theatres.sqlite3` | 1 | 20 281 | 4,2 Mo |
| **Total** | **33** | **1 718 929** | **415,7 Mo** |

Toutes les bases sauf 14-18 et 39-45 tiennent sous 20 Mo en un seul fichier ; ces deux-là
sont découpées par tranche alphabétique de nom (~60-64k lignes / ~15 Mo par tranche).

## Schéma

```sql
CREATE TABLE personnes (
    id INTEGER PRIMARY KEY,
    conflit_id INTEGER,      -- FK -> conflits.id
    nom TEXT,
    prenom TEXT,
    naissance_date TEXT,
    naissance_lieu TEXT,     -- "Commune, Département" ou "Commune, Pays"
    deces_date TEXT,
    deces_lieu TEXT,
    grade TEXT,
    unite TEXT,
    mention_id INTEGER,      -- FK -> mentions.id
    ark TEXT,                -- identifiant de la fiche officielle, vide si non fourni par la source
    recherche TEXT           -- nom+prénom en minuscules, accents retirés (recherche LIKE)
);
CREATE TABLE conflits (id INTEGER PRIMARY KEY, libelle TEXT);
CREATE TABLE mentions (id INTEGER PRIMARY KEY, libelle TEXT);
```

`ark` reconstruit l'URL de la fiche officielle via
`https://www.memoiredeshommes.sga.defense.gouv.fr/fr/ark:/40699/<ark>`.
**Absent pour les bases Indochine, Corée et Algérie/Maroc/Tunisie** : la source ne fournit
pas d'identifiant de fiche pour ces trois conflits (contrairement à 14-18, 39-45 et OPEX) —
ce n'est pas un oubli de compilation, la colonne n'existe pas dans les CSV d'origine.

## Accès via jsDelivr

```
https://cdn.jsdelivr.net/gh/dlastes/memoire-fr-data@main/manifest.json
https://cdn.jsdelivr.net/gh/dlastes/memoire-fr-data@main/1418-00.sqlite3
https://cdn.jsdelivr.net/gh/dlastes/memoire-fr-data@main/3945-00.sqlite3
https://cdn.jsdelivr.net/gh/dlastes/memoire-fr-data@main/indochine.sqlite3
...
```

`@main` suit la branche par défaut (le CDN met en cache ~24h côté jsDelivr — utiliser un
tag de release au lieu de `@main` si une purge de cache immédiate est nécessaire après une
mise à jour des données).

## Régénérer ces fichiers

Voir `scripts/exploration/build_reduced_db.py` dans le dépôt applicatif
[`morts-pour-la-france`](https://github.com/dlastes/morts-pour-la-france).
