# Data Contract Template

How to define the interface between data pipelines (notebooks/CSV) and visualization (HTML/JS).

## What Is a Data Contract

A data contract is a JSON file that specifies:
1. What fields the HTML page expects
2. What type and format each field has
3. Where the source data comes from (which CSV, which columns)
4. How to transform source → target format

The contract eliminates manual data copying. A script reads source CSVs, applies the contract's mapping, and outputs standardized JSON that the HTML consumes directly.

## Contract File Format

Place contracts in `src/data/contracts/page-X.X.json`:

```json
{
  "page": "1.2",
  "title": "Primary Color Overview",
  "description": "Mirror bar chart comparing two data sources by color",
  "sort": { "field": "srcAPr", "order": "descending" },
  "fields": {
    "name": {
      "type": "string",
      "description": "Color English name",
      "example": "Earth"
    },
    "hex": {
      "type": "string",
      "description": "Hex color code",
      "example": "#DBCCB7"
    },
    "srcAPr": {
      "type": "number",
      "description": "Source A Look Presence %",
      "precision": 2,
      "example": 54.88
    },
    "srcAGrowth": {
      "type": "number",
      "description": "Source A YoY Growth %",
      "precision": 2,
      "example": 11.89
    },
    "srcBPr": {
      "type": "number",
      "description": "Source B Look Presence %",
      "precision": 2,
      "example": 63.35
    },
    "srcBGrowth": {
      "type": "number",
      "description": "Source B YoY Growth %",
      "precision": 2,
      "example": 10.13
    }
  },
  "sources": [
    {
      "id": "srcA",
      "path": "runway/output/data/01_primary_color.csv",
      "filter": { "column": "gender", "value": "men" },
      "mapping": {
        "name": "color_name_en",
        "hex": "color_hex",
        "srcAPr": "PR_CY",
        "srcAGrowth": "PR_Growth"
      }
    },
    {
      "id": "srcB",
      "path": "rednote/output/data/03_primary_PR_yoy.csv",
      "filter": { "column": "gender", "value": "women" },
      "mapping": {
        "srcBPr": "CY_PR_pct",
        "srcBGrowth": "YoY_pct"
      }
    }
  ],
  "join": {
    "on": "hex",
    "type": "full_outer"
  }
}
```

## Conversion Script Template

```python
#!/usr/bin/env python3
"""
build_data.py — Convert source CSVs to page JSON per data contracts.

Usage: python build_data.py
"""

import csv
import json
from pathlib import Path

CONTRACTS_DIR = Path("src/data/contracts")
DATA_DIR = Path("src/data")
PROJECT_ROOT = Path(".")

def read_csv(path):
    with open(path, encoding="utf-8") as f:
        return list(csv.DictReader(f))

def apply_filter(rows, filter_spec):
    if not filter_spec:
        return rows
    col = filter_spec["column"]
    val = filter_spec["value"]
    return [r for r in rows if r.get(col) == val]

def safe_float(val, precision=2):
    try:
        return round(float(val), precision)
    except (ValueError, TypeError):
        return 0.0

def process_contract(contract_path):
    contract = json.loads(contract_path.read_text(encoding="utf-8"))
    page_id = contract["page"]
    fields = contract["fields"]

    source_data = {}
    for source in contract["sources"]:
        csv_path = PROJECT_ROOT / source["path"]
        if not csv_path.exists():
            print(f"  ⚠️  Source not found: {csv_path}")
            continue
        rows = read_csv(csv_path)
        rows = apply_filter(rows, source.get("filter"))
        source_data[source["id"]] = {
            "rows": rows,
            "mapping": source["mapping"]
        }

    join_key = contract.get("join", {}).get("on", "hex")
    all_keys = set()
    keyed_data = {}

    for src_id, src in source_data.items():
        mapping = src["mapping"]
        key_col = None
        for target_field, source_col in mapping.items():
            if target_field == join_key or target_field == "name":
                continue
        for row in src["rows"]:
            key_val = None
            for target_field, source_col in mapping.items():
                if target_field in ("name", join_key):
                    key_val = row.get(source_col, "")
                    break
            if key_val is None:
                for possible_key in [join_key, "hex", "name"]:
                    if possible_key in mapping:
                        key_val = row.get(mapping[possible_key], "")
                        break
            if not key_val:
                continue

            all_keys.add(key_val)
            if key_val not in keyed_data:
                keyed_data[key_val] = {}

            for target_field, source_col in mapping.items():
                val = row.get(source_col, "")
                field_spec = fields.get(target_field, {})
                if field_spec.get("type") == "number":
                    val = safe_float(val, field_spec.get("precision", 2))
                keyed_data[key_val][target_field] = val

    result = list(keyed_data.values())

    sort_spec = contract.get("sort")
    if sort_spec:
        reverse = sort_spec.get("order", "ascending") == "descending"
        sort_field = sort_spec["field"]
        result.sort(key=lambda x: x.get(sort_field, 0), reverse=reverse)

    missing_fields = []
    for item in result:
        for field_name in fields:
            if field_name not in item:
                missing_fields.append(f"{field_name} missing in item {item}")

    if missing_fields:
        print(f"  ⚠️  Missing fields in page {page_id}:")
        for mf in missing_fields[:5]:
            print(f"      {mf}")

    output_path = DATA_DIR / f"page-{page_id}.json"
    output_path.write_text(
        json.dumps(result, indent=2, ensure_ascii=False),
        encoding="utf-8"
    )
    print(f"  ✅ page-{page_id}.json — {len(result)} records")

def main():
    print("Building page data from contracts...\n")
    DATA_DIR.mkdir(exist_ok=True)

    contracts = sorted(CONTRACTS_DIR.glob("page-*.json"))
    if not contracts:
        print("No contracts found in", CONTRACTS_DIR)
        return

    for cp in contracts:
        print(f"Processing {cp.name}...")
        process_contract(cp)

    print(f"\nDone. {len(contracts)} contract(s) processed.")

if __name__ == "__main__":
    main()
```

## Gallery / Nested Data Contracts

For pages with nested data structures (e.g., color gallery with gender → season → cohort → colors), use a nested contract:

```json
{
  "page": "2.3",
  "title": "Color Gallery",
  "structure": "nested",
  "hierarchy": ["gender", "season", "cohort"],
  "leaf_fields": {
    "h": { "type": "string", "description": "Hex code" },
    "n": { "type": "string", "description": "Color Chinese name" },
    "p": { "type": "number", "description": "Look Presence %", "precision": 2 },
    "pc": { "type": "string", "description": "Primary color hex" },
    "pn": { "type": "string", "description": "Primary color English name" }
  }
}
```

For nested structures, the conversion script may need custom logic per page. The contract still serves as documentation of the expected output format.

## Validation Checklist

After running `build_data.py`:

- [ ] Each page has a corresponding JSON file in `src/data/`
- [ ] Field names match the contract exactly
- [ ] Number fields have correct precision
- [ ] Sort order matches contract specification
- [ ] No empty/null values where data is expected
- [ ] `git diff` shows meaningful data changes (not formatting noise)
