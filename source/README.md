# `source/` — les archives de source à publier

Ce dossier est celui que la LGPL regarde. Il contient **les archives de
source** et **le manifeste** qui les décrit.

L'obligation vient de la licence, pas des magasins : la LGPL 2.1 §6(d) demande
l'accès au source *depuis le même endroit* que le binaire, et la checklist
officielle de FFmpeg (`ffmpeg.org/legal.html`) demande que le source
corresponde **exactement** au binaire distribué, et que la façon dont FFmpeg a
été compilé soit écrite dans un fichier texte à la racine du source.

Les archives sont produites par **`apps/flutter/tool/prepare-source-offer.sh`**,
qui est l'implémentation de référence. Ce document décrit ce que ce script
produit, et ce que la LGPL exige de ce qu'il produit.

```sh
apps/flutter/tool/prepare-source-offer.sh --check          # état, sans télécharger
apps/flutter/tool/prepare-source-offer.sh                  # produit les archives
apps/flutter/tool/prepare-source-offer.sh --only android   # une seule variante
```

Par défaut, le script écrit dans `dist/source/`. C'est ce dossier qu'il faut
verser dans `site/source/`.

---

## 1. Les archives

Le script produit **trois archives**, une par dépôt de compilation — donc une
par famille de binaires réellement distribués :

| `plateforme` | Libellé | mpv | FFmpeg | Dépôt de compilation |
|---|---|---|---|---|
| `darwin` | macOS et iOS | 0.36.0 (`v0.36.0`) | 6.0 (`n6.0`) | `media-kit/libmpv-darwin-build` @ `v0.6.0` |
| `android` | Android (arm64-v8a, armeabi-v7a, x86, x86_64) | commit `78d43740f52db817d98bcf24fb30a76ab6fa13ff` | commit `ea3d24bbe3c58b171e55fe2151fc7ffaca3ab3d2` | `media-kit/libmpv-android-video-build` @ `v1.1.7` |
| `windows` | Windows (x86_64) | commit de dev `0f78584` (21/10/2024) | commit `ea3d24bbe3c58b171e55fe2151fc7ffaca3ab3d2` | `media-kit/libmpv-win32-video-cmake` @ `20241021` |

L'archive `darwin` couvre **macOS et iOS** : le même dépôt de compilation
produit les deux xcframeworks, avec le même source de mpv et de FFmpeg. Les
deux applications y puisent.

> **Il y a trois archives, et non quatre.** Une archive par **dépôt de
> compilation** — donc par famille de binaires réellement distribués — et non
> par plateforme: macOS et iOS sortent du même dépôt, avec la même source,
> et les séparer ne donnerait au lecteur qu'une archive qui ne compile pas
> seule. `docs/deploiement.md` §10.3 décrit désormais les trois mêmes
> archives que le script produit.

### Nom de fichier

Le script nomme ses archives :

```
infinite-source-<plateforme>-<AAAA-MM-JJ>.tar.gz
```

où `<plateforme>` est `darwin`, `android` ou `windows`, et `<AAAA-MM-JJ>` est
la date de compilation (UTC). Exemple : `infinite-source-darwin-2026-10-25.tar.gz`.

**La date dans le nom pose un problème d'URL** : la LGPL 2.1 §6(d) demande que
le source soit accessible *depuis le même endroit* que le binaire, et un lien
qui change à chaque recompilation n'est pas un lien. La solution retenue :

**les archives sont publiées sous un nom stable.**

```
infinite-source-darwin.tar.gz
infinite-source-android.tar.gz
infinite-source-windows.tar.gz
```

C'est `tools/publish-site.sh` qui fait la conversion : il copie chaque
archive datée sous son nom stable, et il met le manifeste en accord — le
champ `nom` désigne alors le fichier **réellement publié**, et le champ
`nom_original` conserve le nom daté d'origine, pour la traçabilité.

Seul le nom stable est publié. Publier les deux ferait grossir le dépôt
public de 45 Mo à chaque version, sans que cela apporte quoi que ce soit au
lecteur.

La date de compilation reste dans le champ `date` du manifeste, et dans
l'archive elle-même (`README.md` et `LIGNE-DE-COMPILATION.txt`).

### Ce que contient chaque archive

```
infinite-source-<plateforme>-<date>/
├── README.md                  la commande exacte pour reproduire le binaire
├── LIGNE-DE-COMPILATION.txt   les commandes de compilation, telles qu'écrites
│                              dans la recette épinglée
├── SHA256SUMS                 les sommes SHA-256 des deux sources
├── mpv/                       le source de mpv, à la version exacte compilée
├── ffmpeg/                    le source de FFmpeg, à la version exacte compilée
├── media-kit-build/           la recette de compilation complète : les
│                              scripts, le fichier de verrouillage, les
│                              correctifs media_kit
└── LICENSES/
    ├── mpv-COPYING-LGPL-2.1.txt
    ├── ffmpeg-LICENSE.md
    ├── ffmpeg-COPYING.LGPLv3.txt
    ├── ffmpeg-COPYING.GPLv3.txt
    └── recette-media-kit-LICENSE.txt
```

Pourquoi c'est comme ça :

* **`LIGNE-DE-COMPILATION.txt` est à la racine de l'archive.** C'est
  l'exigence de FFmpeg : *« explain how FFmpeg was compiled, in a text file in
  the root of the source »*. Le script ne le reconstitue pas : il extrait les
  commandes **littéralement** des scripts de la recette épinglée
  (`scripts/mpv/build.sh`, `scripts/ffmpeg/build.sh`,
  `buildscripts/flavors/default.sh`, `packages/mpv.cmake`,
  `packages/ffmpeg.cmake`, selon la plateforme).
* **`media-kit-build/` est la recette entière**, pas seulement les correctifs.
  Les correctifs media_kit en font partie : sans eux, le source ne
  correspond pas au binaire. C'est le point le plus souvent vérifié, et c'est
  pourquoi le dépôt de compilation est épinglé par release et pas par
  « dernière version ».
* **`LICENSES/` contient les textes de licence** : LGPL 2.1 de mpv, LGPL v3
  et GPL v3 de FFmpeg, le `LICENSE.md` de FFmpeg, et la licence (MIT) de la
  recette de compilation. La LGPL 3 §4(b) exige que le texte de la GPL
  accompagne celui de la LGPL.
* **Le README de l'archive** donne la commande exacte, les versions, et dit
  ce que la LGPL demande et ce que l'archive y répond.

### Ce qui ne doit pas se trouver dans une archive ni dans le manifeste

* Un chemin absolu de machine (`/Users/…`, `/home/…`), un nom d'utilisateur, un
  nom de poste.
* Un jeton, une clé, une adresse de compte, une URL signée.
* Le contenu du dépôt privé du projet.

Les archives et le manifeste sont publics : ils doivent pouvoir être publiés
tels quels. Le script, lui, ne lit jamais `~/.infinite-credentials`.

---

## 2. Où les placer

Les fichiers `.tar.gz` et le fichier `manifest.json` vont **dans ce
dossier-ci**, c'est-à-dire `site/source/`, à côté de ce `README.md` :

```
site/source/
├── README.md          ce document
├── manifest.json      le manifeste
├── infinite-source-darwin.tar.gz
├── infinite-source-android.tar.gz
└── infinite-source-windows.tar.gz
```

Les liens de la page `site/source.html` pointent vers ce dossier, en chemins
relatifs : `source/infinite-source-….tar.gz`.

**C'est `tools/publish-site.sh` qui y verse le tout** : il prend les archives
datées dans `dist/source/`, les copie sous leur nom stable, met le manifeste
en accord, et publie le site dans le dépôt public
`ya5inq/infinite-media-player-site`, dont GitHub Pages sert le contenu.

**GitHub Pages ne sert que ce qui est versionné dans la branche publiée.** Les
archives sont donc commitées dans cette branche, sous leur nom stable.

Le dossier `source/` ne sert à rien tant que les archives n'y sont pas : la
page `source.html` ne serait pas conforme à l'obligation de la LGPL.

---

## 3. Le manifeste : `manifest.json`

Un seul fichier JSON, à la racine de ce dossier. Il décrit les archives : ce
qu'il y a dedans, à quelle version, avec quelles sommes de contrôle, et
comment le binaire a été compilé.

Le manifeste est écrit par le script (`prepare-source-offer.sh`, section
« Manifeste »). Le format est donc **celui du script** — le décrire autrement
donnerait une spécification que rien ne produit.

### Règles de forme

* JSON strict, UTF-8, deux espaces d'indentation, une seule clé racine.
* Les clés sont en français, et le reste aussi : c'est un document destiné à
  être lu par l'éditeur et par un juriste, pas par une machine qui parle
  anglais.
* Les sommes SHA-256 sont en hexadécimal minuscule, 64 caractères.
* Les dates sont en `AAAA-MM-JJ` ; `genere_le` est en ISO 8601 UTC, terminé
  par `Z`.
* `ligne_de_compilation` est une **ligne unique** ; `commande_de_reproduction`
  est la commande complète, **réduite à une seule ligne** (les retours à la
  ligne sont remplacés par des espaces).
* Un lecteur doit pouvoir valider le fichier avec un simple
  `json.load` : le script le fait lui-même avant de terminer.

### Exemple

Les valeurs marquées `⟨…⟩` sont des exemples : elles sont à remplacer par les
valeurs réelles.

```json
{
  "projet": "Infinite Media Player",
  "objet": "Archives de source exigées par la LGPL 2.1 §6 et la LGPL v3 §4",
  "genere_le": "2026-10-25T00:00:00Z",
  "generateur": "apps/flutter/tool/prepare-source-offer.sh",
  "licences": {
    "libmpv": "LGPL 2.1 or later",
    "ffmpeg": "LGPL v3 or later"
  },
  "archives": [
    {
      "plateforme": "darwin",
      "libelle": "macOS et iOS",
      "nom": "infinite-source-darwin-2026-10-25.tar.gz",
      "taille_octets": 12345678,
      "sha256": "⟨sha256 de l'archive⟩",
      "date": "2026-10-25",
      "depot_de_compilation": {
        "url": "https://github.com/media-kit/libmpv-darwin-build",
        "release": "v0.6.0",
        "tarball": "https://github.com/media-kit/libmpv-darwin-build/archive/refs/tags/v0.6.0.tar.gz",
        "sha256": "⟨sha256 du dépôt de compilation⟩",
        "fichier_de_verrouillage": "downloads.lock"
      },
      "sources": [
        {
          "composant": "mpv",
          "version": "0.36.0",
          "commit": "⟨commit, ou vide si la version est étiquetée⟩",
          "url": "https://github.com/mpv-player/mpv/archive/⟨archive⟩.tar.gz",
          "sha256": "⟨sha256 du source mpv⟩"
        },
        {
          "composant": "ffmpeg",
          "version": "6.0",
          "commit": "⟨commit, ou vide si la version est étiquetée⟩",
          "url": "https://github.com/FFmpeg/FFmpeg/archive/⟨archive⟩.tar.gz",
          "sha256": "⟨sha256 du source FFmpeg⟩"
        }
      ],
      "ligne_de_compilation": "⟨ligne unique : les commandes de compilation⟩",
      "commande_de_reproduction": "⟨commande complète, sur une seule ligne⟩"
    }
  ]
}
```

L'exemple montre **une seule** archive pour rester lisible ; le fichier réel
en contient **trois** : `darwin`, `android`, `windows`, dans cet ordre.

### Les champs, un par un

#### Racine

| Champ | Type | Rôle |
|---|---|---|
| `projet` | chaîne | `Infinite Media Player`. |
| `objet` | chaîne | Ce que le manifeste contient, en une phrase. |
| `genere_le` | chaîne | Horodatage de génération, ISO 8601 UTC, terminé par `Z`. |
| `generateur` | chaîne | Le script qui a produit le fichier. |
| `licences` | objet | Les licences des bibliothèques livrées, en clair : `libmpv` et `ffmpeg`. C'est l'information qui dit, d'un coup d'œil, que la pile livrée est bien en LGPL. |
| `archives` | tableau | Les archives. `plateforme` est unique dans le tableau. |

#### Une archive

| Champ | Type | Rôle |
|---|---|---|
| `plateforme` | chaîne | `darwin`, `android` ou `windows`. Identifiant de la variante ; ne pas le changer. |
| `libelle` | chaîne | Le nom lisible : « macOS et iOS », « Android (arm64-v8a, armeabi-v7a, x86, x86_64) », « Windows (x86_64) ». |
| `nom` | chaîne | Le nom du fichier de l'archive, **tel qu'il est publié** : le nom stable. |
| `nom_original` | chaîne | Le nom daté que le script a donné au fichier. Absent tant que la conversion en nom stable n'a pas eu lieu. |
| `taille_octets` | entier | La taille de l'archive en octets, telle que publiée. |
| `sha256` | chaîne | La somme SHA-256 du fichier de l'archive. C'est elle que l'utilisateur vérifie. |
| `date` | chaîne | La date de compilation, `AAAA-MM-JJ`. |
| `depot_de_compilation` | objet | Le dépôt media-kit d'où vient la recette, et la release épinglée. |
| `sources` | tableau | Les deux sources livrées : mpv et FFmpeg. |
| `ligne_de_compilation` | chaîne | La ligne de compilation, sur une seule ligne. |
| `commande_de_reproduction` | chaîne | La commande complète qui reproduit le binaire, sur une seule ligne. |

#### Le dépôt de compilation

| Champ | Type | Rôle |
|---|---|---|
| `url` | chaîne | L'URL du dépôt media-kit. |
| `release` | chaîne | La release épinglée (`v0.6.0`, `v1.1.7`, `20241021`). |
| `tarball` | chaîne | L'URL de l'archive de cette release. |
| `sha256` | chaîne | La somme SHA-256 du dépôt téléchargé. |
| `fichier_de_verrouillage` | chaîne | Le fichier du dépôt qui épingle les versions (`downloads.lock`, `buildscripts/include/depinfo.sh`, `packages/ffmpeg.cmake`). |

#### Une source

| Champ | Type | Rôle |
|---|---|---|
| `composant` | chaîne | `mpv` ou `ffmpeg`. |
| `version` | chaîne | La version, telle qu'elle est annoncée par le projet. |
| `commit` | chaîne | Le commit **exact** du source mis dans l'archive, sur 40 caractères. Vide quand la version est étiquetée : c'est alors la version qui fait foi, et elle est vérifiée par la somme. |
| `url` | chaîne | L'URL exacte du tarball downloaded. |
| `sha256` | chaîne | La somme SHA-256 du tarball tel qu'il a été téléchargé. |

#### La compilation

| Champ | Type | Rôle |
|---|---|---|
| `ligne_de_compilation` | chaîne | La ligne de compilation, en une ligne. Elle est extraite de la recette, pas reconstituée. |
| `commande_de_reproduction` | chaîne | La commande exacte, sur une ligne. |

### Pourquoi ces champs

* `archives[].sha256` : c'est ce qui rend l'archive vérifiable. Sans somme,
  l'utilisateur ne peut pas savoir s'il a bien le source qui correspond à son
  binaire.
* `sources[].commit` : c'est ce qui garantit que le source est **exactement**
  celui qui a été compilé. La version seule ne suffit pas — surtout pour
  `android` et `windows`, où la version d'mpv est un commit de développement.
* `sources[].sha256` : le source téléchargé est vérifié avant d'être extrait.
  Une somme qui ne correspond pas est une erreur fatale, jamais un « on
  continue quand même ».
* `depot_de_compilation` : c'est la **preuve** que le source correspond au
  binaire. La release est épinglée, et son fichier de verrouillage est dans
  l'archive.
* `licences` : dit d'un coup d'œil que la pile livrée est en LGPL, et non en
  GPL.
* `ligne_de_compilation` : l'exigence de FFmpeg, texte, à la racine du
  source.

---

## 4. Produire le dossier

L'ordre des opérations du script, tel qu'il est attendu :

1. Télécharger le dépôt de compilation **épinglé** (`--release`), et vérifier
   sa somme.
2. En déduire les versions de mpv et de FFmpeg **depuis le fichier de
   verrouillage** du dépôt — pas depuis une table codée en dur. C'est ce qui
   fait que le source publié correspond au binaire livré.
3. Télécharger les deux sources, en vérifier les sommes (les versions
   étiquetées ont une somme connue, inscrite dans le verrouillage ; les
   versions épinglées par commit ont une somme calculée au premier passage et
   **enregistrée** dans `apps/flutter/tool/source-offer.lock`, puis vérifiée
   aux passages suivants).
4. Extraire les sources, assembler l'archive, y déposer la recette, les
   textes de licence, `LIGNE-DE-COMPILATION.txt`, `SHA256SUMS` et le README.
5. Calculer la somme et la taille de l'archive **une fois l'archive écrite**.
6. Écrire `manifest.json`.

Le point 5 est un piège classique : la somme se calcule sur le fichier final.
Si l'archive est recompressée ou retouchée après, la somme change, et le
manifeste doit être régénéré.

### Vérifier ce qui a été produit

```sh
cd site/source

# 1. Les archives sont là
ls -l infinite-source-*.tar.gz

# 2. La somme annoncée est la bonne
shasum -a 256 infinite-source-darwin-2026-10-25.tar.gz

# 3. L'archive s'ouvre, et contient ce qu'il doit
tar tzf infinite-source-darwin-2026-10-25.tar.gz \
  | grep -E 'LIGNE-DE-COMPILATION|LICENSES/'

# 4. Le manifeste est du JSON valide
python3 -m json.tool manifest.json > /dev/null && echo OK

# 5. Les sommes du manifeste correspondent aux fichiers
python3 - <<'PY'
import hashlib, json, pathlib
m = json.loads(pathlib.Path("manifest.json").read_text())
for a in m["archives"]:
    h = hashlib.sha256(pathlib.Path(a["nom"]).read_bytes()).hexdigest()
    print("OK " if h == a["sha256"] else "DIFF", a["plateforme"], a["nom"])
PY
```

---

## 5. Avant de publier

- [ ] Les archives sont présentes dans ce dossier, sous un nom stable (ou
  jointes à une release d'URL stable).
- [ ] Le `sha256` de chaque archive dans `manifest.json` correspond au fichier
  réellement publié.
- [ ] Chaque archive contient : `mpv/`, `ffmpeg/`, `media-kit-build/`,
  `LICENSES/`, `LIGNE-DE-COMPILATION.txt`, `SHA256SUMS` et `README.md`.
- [ ] `LIGNE-DE-COMPILATION.txt` contient les commandes **littérales** de la
  recette épinglée, et non une reconstitution.
- [ ] Aucune archive ne contient de binaire compilé, de `.git`, de chemin
  absolu, ni de secret.
- [ ] `manifest.json` passe `python3 -m json.tool`.
- [ ] La mention de FFmpeg — *« This software uses libraries from the FFmpeg
  project under the LGPLv2.1 and LGPLv3. »* — est présente sur la page où
  l'application se télécharge (`site/index.html` et `site/source.html`).
- [ ] L'adresse de contact de l'offre de source écrite (§6(c)) est renseignée
  dans `site/source.html`.
- [ ] `site/source.html` et `site/licences.html` pointent vers des fichiers qui
  existent vraiment.
- [ ] Le nombre d'archives est le même dans `docs/deploiement.md` §10.3, dans
  le script et sur la page.
