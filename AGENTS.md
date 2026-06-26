# Brief IA — ll_data

Repo de **données** pour LL : packs de seed (vocabulaire + phrases) que l'app
télécharge à l'install.

## Stack

Aucun code — uniquement des fichiers de données :
- `manifest.json` à la racine (schéma v2)
- `packs/<source>/<level>/<pair>-v<n>.jsonl.gz`
- documentation

Les data sont consommées par la crate `ll-packs` (dans
`ll_app/crates/ll-packs/`) qui les fetch et les pousse dans la DB locale
via le use case `import_seed`.

## Layout des packs

```
packs/
  <source>/        # curated | llm | tatoeba
    <level>/       # ll-1 | ll-2 | … | ll-7 | none
      <src>-<dst>-v<n>.jsonl.gz
```

Exemples :
- `packs/curated/ll-1/fr-en-v1.jsonl.gz` — concepts ll-1 curés main, fr→en
- `packs/llm/ll-1/fr-en-v1.jsonl.gz` — générée via Claude CLI (ll_lab/scripts/llm/)
- `packs/tatoeba/none/fr-en-v1.jsonl.gz` — co-occurrence Tatoeba, sans palier

`level = "none"` est réservé aux sources qui n'ont pas d'unité conceptuelle
attachée à un palier LL (typiquement Tatoeba — freq-rank pur).

Les paliers `ll-N` sont définis par la **Norme LL** documentée dans
`ll_lab/concepts/SPEC.md` : chaque palier est un set fixé de concepts,
livré intégralement dans toutes les langues couvertes. Mappings
indicatifs : `ll-1 ≈ CEFR A1 ≈ JLPT N5 ≈ HSK 1-2`, etc.

## Schéma `manifest.json` (v2)

```json
{
  "version": 2,
  "packs": [
    {
      "id": "curated-ll-1-fr-en-v1",
      "source": "curated",
      "level": "ll-1",
      "src_lang": "fr",
      "dst_lang": "en",
      "version": 1,
      "size_bytes": 1064,
      "sha256": "<sha256 du .jsonl.gz>",
      "url": "https://raw.githubusercontent.com/grosjeanbaptiste/ll_data/main/packs/curated/ll-1/fr-en-v1.jsonl.gz",
      "license": "CC-BY 4.0 (LL curated)"
    }
  ]
}
```

- `version` (racine) = version du **format manifest** (v2 depuis 2026-06-26).
- `id` = `<source>-<level>-<src>-<dst>-v<n>` (slug stable, unique).
- `source` ∈ {`curated`, `llm`, `tatoeba`}.
- `level` ∈ {`ll-1`, `ll-2`, …, `ll-7`, `none`} — paliers de la **Norme LL**
  (voir `ll_lab/concepts/SPEC.md`).
- `version` (pack) = version du pack lui-même (pour invalidation de cache).

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
