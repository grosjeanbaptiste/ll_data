# Brief IA — ll_data

Repo de **données** pour LL : packs de seed (vocabulaire + phrases) que l'app
télécharge à l'install.

## Stack

Aucun code — uniquement des fichiers de données :
- `manifest.json` à la racine
- `packs/*.jsonl.gz`
- documentation

Les data sont consommées par la crate `ll-packs` (dans
`ll_app/crates/ll-packs/`) qui les fetch et les pousse dans la DB locale
via le use case `import_seed`.

## Schéma `manifest.json`

```json
{
  "version": 1,
  "packs": [
    {
      "id": "tatoeba-fr-en-v1",
      "version": 1,
      "src_lang": "fr",
      "dst_lang": "en",
      "size_bytes": 5242880,
      "sha256": "<sha256 du .jsonl.gz>",
      "url": "https://raw.githubusercontent.com/grosjeanbaptiste/ll_data/main/packs/tatoeba-fr-en-v1.jsonl.gz",
      "license": "CC-BY-2.0 (Tatoeba)"
    }
  ]
}
```

`version` au niveau racine = version du **format manifest** (pour gérer
l'évolution du schéma). `version` au niveau pack = version du pack lui-même
(pour le invalidation de cache côté client).

## Schéma d'une ligne JSONL (`WireTrio`)

```json
{
  "src": {"text": "chat", "lang": "fr", "rank": 42},
  "dst": {"text": "cat",  "lang": "en", "rank": 38},
  "src_sent": "Le chat dort.",
  "dst_sent": "The cat sleeps."
}
```

`rank` = freq_rank (1 = mot le plus fréquent). Doit être cohérent à
l'intérieur d'un pack.

## Conventions

- **Pas de PII** dans les packs (anonymisation des phrases si extraites
  d'utilisateurs).
- **Licence par pack** documentée dans le manifest (Tatoeba = CC-BY 2.0,
  UDHR = domaine public, etc.).
- **Pas d'historique des gros fichiers dans git** : si un `.jsonl.gz` dépasse
  ~1 MB, créer une release et attacher le fichier comme asset, ne pas le
  committer. Le manifest pointe alors vers l'URL release.

## Quand intervenir

Modifications légitimes :
- Ajouter un pack (cf. README)
- Corriger le manifest (URL cassée, sha256 erroné)
- Documenter un nouveau format

Modifications interdites depuis ce repo :
- Pas de code Rust ici
- Pas de logique applicative
