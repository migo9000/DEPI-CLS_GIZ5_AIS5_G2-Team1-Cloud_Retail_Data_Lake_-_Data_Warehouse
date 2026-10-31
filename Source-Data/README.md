# AdventureWorks Sales: Source Simulation

The Excel workbook plays the role of a **source system**. The snippet below turns it into JSON files, one per order month, laid out like batches landing in the data lake's `raw` zone.

Requires `pandas` and `openpyxl`.

## Run

```python
import pandas as pd
from pathlib import Path

SRC = "AdventureWorks_Sales_Exercise.xlsx"   # path to the workbook
OUT = Path("raw/sales")                      # stands in for the lake's raw zone

# Postal codes must stay text, or leading zeros are lost
df = pd.read_excel(SRC, dtype={"customer_postal_code": str, "reseller_postal_code": str})

# Keep dates as plain YYYY-MM-DD text (blank ship dates stay null)
for col in ["order_date", "due_date", "ship_date"]:
    df[col] = pd.to_datetime(df[col]).dt.strftime("%Y-%m-%d")

# The source "emits" one JSON Lines file per order month
df["batch_month"] = df["order_date"].str[:7]
for month, part in df.groupby("batch_month"):
    folder = OUT / f"batch_month={month}"
    folder.mkdir(parents=True, exist_ok=True)
    part.drop(columns="batch_month").to_json(
        folder / f"sales_{month}.json", orient="records", lines=True, force_ascii=False)
```

## Output

```
raw/sales/
├── batch_month=2017-07/sales_2017-07.json
├── batch_month=2017-08/sales_2017-08.json
└── ...                                     (36 files, 121,253 records)
```

## Notes

- Each file is **JSON Lines**: one record per line.
- Blank Excel cells become `null`.
- Read it back in Spark with `spark.read.json("raw/sales/")`; Spark adds `batch_month` from the folder name.
- Next step: copy `raw/` into the ADLS `raw` container (Storage Explorer or `azcopy`).
