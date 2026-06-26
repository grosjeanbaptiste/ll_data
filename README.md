# ll_data

Packs de seed pour l'app **LL** (apprentissage de langues).

## Contenu

- `manifest.json` — liste des packs disponibles avec métadonnées (id, source,
  level, paire de langues, version, taille, sha256, URL, licence).
- `packs/<source>/<level>/<pair>-v<n>.jsonl.gz` — un fichier par pack,
  classé par provenance (curated/llm/tatoeba) puis palier LL
  (ll-1/ll-2/…/ll-7/none). Chaque ligne est un `WireTrio` JSON conforme
  au schéma de `ll-ingest` (`SeedRecord`). Les paliers `ll-N` sont
  définis par la **Norme LL** dans `ll_lab/concepts/SPEC.md`.

## Format `WireTrio`

```json
{
  "src": {"text": "chat", "lang": "fr", "rank": 42},
  "dst": {"text": "cat",  "lang": "en", "rank": 38},
  "src_sent": "Le chat dort.",
  "dst_sent": "The cat sleeps."
}
```

## Distribution

Les packs sont distribués via :
- **GitHub raw** pour les petits packs (< 1 MB compressé) :
  `https://raw.githubusercontent.com/grosjeanbaptiste/ll_data/main/packs/<source>/<level>/<pair>-v<n>.jsonl.gz`
- **GitHub Releases** pour les gros packs (> 1 MB) en tant qu'assets attachés
  à une release versionnée.

Le `manifest.json` distant (raw) sert d'index : c'est ce que la crate
`ll-packs` (dans `ll_app/`) lit en premier au lancement.

## Ajouter un pack

Les packs sont générés depuis le repo voisin `ll_lab/` (scripts curated /
LLM / Tatoeba) et atterrissent ici sous `packs/<source>/<level>/`. Le
script générateur patche aussi `manifest.json` (id, source, level, paire,
taille, sha256, URL).

1. Lancer le générateur dans `ll_lab/` (cf. son AGENTS.md).
2. Vérifier le diff sur `manifest.json` + le placement des nouveaux fichiers.
3. Commit + push depuis `ll_data/`. Si gros pack : créer une release et y
   attacher le `.jsonl.gz` (l'URL du manifest pointe alors vers l'asset).

## Licence

CC-BY 4.0 (voir [`LICENSE`](./LICENSE)).
