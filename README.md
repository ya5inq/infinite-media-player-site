# Infinite Media Player — le site public

Ce dépôt contient le site public d'**Infinite Media Player** : la politique
de confidentialité, les mentions légales, les conditions d'utilisation, les
licences des bibliothèques livrées, et l'offre de source exigée par la LGPL.

Il est **public**, et c'est voulu : les magasins — Google Play, Amazon
Appstore, Microsoft Store, App Store — vérifient l'adresse de la politique
de confidentialité, et ils doivent pouvoir l'ouvrir. Le **code** de
l'application, lui, n'est pas public, et n'a pas à l'être.

| Page | Ce qu'elle est |
|---|---|
| [`confidentialite.html`](confidentialite.html) | la politique de confidentialité, en français et en anglais |
| [`mentions-legales.html`](mentions-legales.html) | les mentions légales françaises — l'article 1-1 de la loi n° 2004-575 |
| [`conditions.html`](conditions.html) | les conditions d'utilisation, en français et en anglais |
| [`licences.html`](licences.html) | les licences des bibliothèques livrées, avec leur texte intégral |
| [`source.html`](source.html) | l'offre de source LGPL 2.1 §6, et les archives |
| [`support.html`](support.html) | le contact |
| [`telecharger.html`](telecharger.html) | les fichiers d'installation pour les télévisions |

## L'offre de source — la LGPL

Les bibliothèques livrées — mpv, FFmpeg, libass, dav1d, media_kit — sont
sous **LGPL**. La LGPL 2.1 §6 demande que le source soit accessible
**depuis le même endroit** que le binaire : il est donc ici, dans
[`source/`](source/).

| Fichier | Ce que c'est |
|---|---|
| [`source/infinite-source-darwin.tar.gz`](source/infinite-source-darwin.tar.gz) | le source et la ligne de compilation de macOS et iOS |
| [`source/infinite-source-android.tar.gz`](source/infinite-source-android.tar.gz) | le source et la ligne de compilation d'Android et Android TV |
| [`source/infinite-source-windows.tar.gz`](source/infinite-source-windows.tar.gz) | le source et la ligne de compilation de Windows |
| [`source/manifest.json`](source/manifest.json) | le manifeste : versions, sommes SHA-256, commandes de compilation |
| [`source/README.md`](source/README.md) | la spécification du dossier et du manifeste |

**Vérifier une archive** — la somme annoncée par le manifeste doit être
identique à celle que vous calculez :

```sh
shasum -a 256 source/infinite-source-darwin.tar.gz
```

La ligne de compilation exacte de chaque plateforme est écrite **à la racine**
de chaque archive, dans `LIGNE-DE-COMPILATION.txt` : c'est ce que la
checklist FFmpeg exige, et c'est ce qui permet de recompiler le binaire
publié.

## Les fichiers d'installation

Le dépôt porte aussi les fichiers d'installation pour Android TV, **à la
racine**, sous des noms courts : sur une télévision, l'adresse se tape à la
télécommande, et chaque caractère compte.

| Fichier | Pour qui |
|---|---|
| [`arm64.apk`](arm64.apk) | Android TV, Fire TV 64 bits (`arm64-v8a`) |
| [`armv7.apk`](armv7.apk) | Fire TV 32 bits (`armeabi-v7a`) |

Les sommes de contrôle sont dans [`SHA256SUMS`](SHA256SUMS).

Sur la télévision : application **Downloader** (Amazon Appstore) → *Enter URL* →
`ya5inq.github.io/infinite-media-player-site/arm64.apk` → *Download* → puis ouvrir
le fichier téléchargé et installer.

La page complète est ici : [`telecharger.html`](telecharger.html)

## Ce que ce dépôt ne contient pas

Le code de l'application. Il n'est pas sous licence libre, il n'a pas à
l'être, et il n'est pas ici.

## L'éditeur

**ya5inq** — éditeur indépendant, une personne physique, sans société.
Ses coordonnées complètes sont dans les [mentions légales](mentions-legales.html),
et son contact est dans la page [support](support.html).
