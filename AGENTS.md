# Brief IA — ll_data

Repo de **données** pour LL : packs de seed (vocabulaire + phrases) que l'app
télécharge à l'install.

## Stack

Aucun code — uniquement des fichiers de données :
- `manifest.json` à la racine (schéma v2 + `concept_packs`)
- `packs/curated/concept/ll-N-v<n>.jsonl.gz` — un pack **concept** par palier
- documentation

Les data sont consommées par la crate `ll-packs` (dans
`ll_app/crates/ll-packs/`) qui les fetch et les pousse dans la DB locale
via le use case `import_seed`.

## Layout des packs

```
packs/
  curated/
    concept/       # un pack par palier, toutes les langues
      ll-1-v0.jsonl.gz … ll-7-v0.jsonl.gz
```

Chaque ligne d'un pack concept est un `SourceRecord` : un concept et ses
realizations dans les 6 langues (fr, en, nl, es, de, it). **Un palier
couvre donc toutes les paires** — l'app dérive les 30 paires orientées
des realizations. Les packs sont listés dans `concept_packs` du manifest
et produits par `ll_lab/scripts/concepts/rebuild_pack_v0.py`.

**Legacy (retiré 2026-10-01)** : l'ancien format « un pack par paire et par
palier » (`packs/curated/ll-1..4/<src>-<dst>-v7.jsonl.gz`, 60 packs) n'est
plus lu par l'app depuis le passage concept-only. Il était généré depuis
les mêmes YAML de `ll_lab/concepts/`, sans les corrections ultérieures
(92 % identique aux packs concept, le reste = erreurs corrigées depuis).
Les fichiers sont supprimés ; `packs` reste présent mais vide dans le
manifest car les versions installées de l'app exigent ce champ. Le
générateur `build_packs.py` est désactivé. Les sections de schéma
ci-dessous décrivent ce format historique.

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
