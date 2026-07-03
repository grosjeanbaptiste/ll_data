# ll_data

Packs de seed pour l'app **LL** (apprentissage de langues).

## Contenu

- `manifest.json` — liste des packs disponibles avec métadonnées (id, version,
  taille, paire de langues, URL de download).
- `packs/*.jsonl.gz` — un fichier par pack. Chaque ligne est un `WireTrio`
  JSON conforme au schéma de `ll-ingest` (`SeedRecord`).

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
- **GitHub raw** pour les petits packs (< 1 MB compressé) directement depuis
  `main` : `https://raw.githubusercontent.com/grosjeanbaptiste/ll_data/main/packs/<nom>.jsonl.gz`
- **GitHub Releases** pour les gros packs (> 1 MB) en tant qu'assets attachés
  à une release versionnée.

Le `manifest.json` distant (raw) sert d'index : c'est ce que la crate
`ll-packs` (dans `ll_app/`) lit en premier au lancement.

## Ajouter un pack

1. Générer le JSONL.gz via `cargo run -p ll-cli -- tatoeba` puis `seed export`
   dans `ll_app/`.
2. Le déposer ici sous `packs/pack-<langues>-<source>-v<n>.jsonl.gz`.
3. Mettre à jour `manifest.json` (id unique, version monotone, sha256
   recommandé).
4. Commit + tag `data-v<n>` + push. Si gros pack : créer une release et y
   attacher le `.jsonl.gz`.

## Licence

CC-BY 4.0 (voir [`LICENSE`](./LICENSE)).
