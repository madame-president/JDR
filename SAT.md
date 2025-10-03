# Standard for Address Transparency (SAT-1): institutional-grade reporting for Bitcoin addresses

## Abstract

The Standard for Address Transparency *(SAT-1 for short)* is a new format that defines the structure and key components of a Bitcoin address statement. SAT-1 was built inspired by traditional financial practices. My goal is to introduce a clean, structured way to present Bitcoin address details, balances and historical transactions in an auditable format. SAT-1 ensures that Bitcoin holdings can be audited with the same precision as traditional financial statements. This will bridge the wide gap between Bitcoin's technical transparency and institutional-grade financial reporting needs.

## Introduction

The Bitcoin network contains hunderds of gigabytes of unstructured data. The data is public and transparent, but there is no standarized method for reporting this information in a financial context. Without a clear framework, reporting on Bitcoin addresses often becomes vulnerable to misinterpretation and inconsinsitency. The method I propose to solve this problem is the standard of address transparancy.

## SAT-1 components

SAT-1 reports are divided into two sections:

1. Address Summary
2. Transaction History

### Address Summary contains:

| Field                        | Description                                                                                   |
| :--------------------------- | :-------------------------------------------------------------------------------------------- |
| **Timestamp**                | The exact time when the report was generated in UTC.                                          |
| **Market Price (USD)**       | The market price of Bitcoin at the time when the report is generated.                         |
| **Address**                  | The Bitcoin address being audited.                                                            |
| **Address Type**             | The classification of the address (`p2pkh`, `p2sh`, `bech32`, `taproot`).                     |
| **First Seen Receiving**     | The timestamp when the address was first seen receiving Bitcoin.                              |
| **Last Seen Receiving**      | The most recent timestamp the address received Bitcoin.                                       |
| **First Seen Spending**      | The timestamp when the address was first seen spending Bitcoin (if any).                      |
| **Last Seen Spending**       | The most recent timestamp the address spent Bitcoin (if any).                                 |
| **Period Transaction Count** | The number of transactions involving the address during the reporting period.                 |
| **Unspent Output Count**     | The current number of UTXOs associated with the address.                                      |
| **Current Balance (BTC)**    | The current Bitcoin balance of the address.                                                   |
| **Current Balance (USD)**    | The USD equivalent of the current balance.                                                    |
| **Total Received (BTC)**     | The total Bitcoin ever received by the address during the reporting period.                   |
| **Total Received (USD)**     | The USD equivalent of the total amount received with the value at time of transaction.        |
| **Total Sent (BTC)**         | The total Bitcoin ever sent by the address during the reporting period.                       |
| **Total Sent (USD)**         | The USD equivalent of the total amount sent with the value at time of transaction.            |
| **Period Start**             | The starting date for the reporting period.                                                   |
| **Period End**               | The ending date for the reporting period.                                                     |

#### Each data field included helps with:

- Generating Bitoin reports at specific date ranges.
- Calculating the address P/L.
- Measuring overall Bitcoin volume transacted within the address.
- Converting Bitcoin to fiat.

### Transaction history contains:

| Field                | Description                                                                    |
| :------------------- | :----------------------------------------------------------------------------- |
| **Block Height**     | The block in which the transaction was confirmed.                              |
| **Transaction Time** | The time the transaction was confirmed in UTC.                                 |
| **Txid**             | The transaction hash.                                                          |
| **Amount (BTC)**     | The amount of Bitcoin transacted.                                              |
| **Direction**        | Whether the Bitcoin was received or sent.                                      |

#### Each data field included helps with (additional):

- Verifying the transactions have been confirmed.
- Categorizing each transaction as received or sent, to mimic the double-entry accounting system.

## Why does this matter?

SAT-1 goes beyond a report formatting. It is the first attempt at the institutional level to reconcile Bitcoin's technical transparency and traditional financial accountability. The standardized fields ensure clear comparisons across addresses. It is designed to satisfy internal audits, external audits, tax preparation and compliance needs. The values are recorded in UTC for universal time accuracy. And most importantly, every data point is backed by on-chain evidence.

As Bitcoin adoption continues amoung institutions and wealth funds, auditable Bitcoin address reporting will become a requirement. SAT-1 establishes a strong foundation for this inevitable future.

## Implementation of code using SAT-1

```
import requests
import pandas as pd

# -------- config --------
ADDRESS = "YOUR_BITCOIN_ADDRESS"
START_DATE = "2024-01-01"
END_DATE = "2024-12-31"
BLOCKCHAIR_API_KEY = "YOUR_BLOCKCHAIR_API_KEY"
TIMEOUT_SEC = 15

# -------- helpers --------
def satsBtc(s) -> float:
    try:
        return float(s) / 100_000_000
    except Exception:
        return 0.0

# -------- module 1: get data --------
def getData(address: str, startDate: str, endDate: str, apiKey: str):
    base = "https://api.blockchair.com/bitcoin/dashboards/address/"
    url = f"{base}{address}?transaction_details=true&q=time({startDate}..{endDate})&key={apiKey}"
    try:
        r = requests.get(url, timeout=TIMEOUT_SEC)
        r.raise_for_status()
        return r.json()
    except requests.RequestException as e:
        print(f"request err: {e}")
        return None

# -------- module 2: process -> dicts --------
def makeSummary(data: dict, address: str):
    try:
        a = data["data"][address]["address"]
        p = data["data"][address]["period"]
        c = data["context"]

        def clean(v, placeholder="Not yet spent"):
            return placeholder if v in (None, "2000-01-01 00:00:00") else v

        return {
            "Timestamp": c["cache"]["since"],
            "Market Price (USD)": float(c["market_price_usd"]),
            "Address": address,
            "Address Type": str(a["type"]),
            "First Seen Receiving": str(clean(a["first_seen_receiving"])),
            "Last Seen Receiving": str(clean(a["last_seen_receiving"])),
            "First Seen Spending": str(clean(a["first_seen_spending"])),
            "Last Seen Spending": str(clean(a["last_seen_spending"])),
            "Period Transaction Count": int(p["transaction_count"]),
            "Unspent Output Count": int(a["unspent_output_count"]),
            "Current Balance (BTC)": satsBtc(a["balance"]),
            "Current Balance (USD)": float(a["balance_usd"]),
            "Total Received (BTC)": satsBtc(a["received"]),
            "Total Received (USD)": float(a["received_usd"]),
            "Total Sent (BTC)": satsBtc(a["spent"]),
            "Total Sent (USD)": float(a["spent_usd"]),
            "Period Start": str(p["period_start"]),
            "Period End": str(p["period_end"]),
        }
    except KeyError as e:
        print(f"data field missing: {e}")
        return None

def makeTx(data: dict, address: str) -> pd.DataFrame:
    try:
        txs = data["data"][address].get("transactions", []) or []
        rows = []
        for t in txs:
            chg = t.get("balance_change", 0)
            rows.append({
                "Block Height": t.get("block_id"),
                "Transaction Time": t.get("time"),
                "Txid": t.get("hash"),
                "Amount": satsBtc(chg),
                "Direction": "Received" if (chg or 0) > 0 else "Sent",
            })
        return pd.DataFrame(rows, columns=["Block Height","Transaction Time","Txid","Amount","Direction"])
    except KeyError as e:
        print(f"tx parse err: {e}")
        return pd.DataFrame(columns=["Block Height","Transaction Time","Txid","Amount","Direction"])

# -------- module 3: dataframes --------
def makeFrames(summary: dict, txDf: pd.DataFrame):
    sumDf = pd.DataFrame(summary.items(), columns=["Field", "Value"])
    return sumDf, txDf

# -------- module 4: save --------
def saveExcel(sumDf: pd.DataFrame, txDf: pd.DataFrame, address: str) -> str:
    name = f"verified_sat_report_{address}.xlsx"
    with pd.ExcelWriter(name, engine="openpyxl") as w:
        sumDf.to_excel(w, sheet_name="Summary", index=False)
        txDf.to_excel(w, sheet_name="Transactions", index=False)
    return name

# -------- run --------
def run() -> None:
    if not ADDRESS or not BLOCKCHAIR_API_KEY:
        print("set ADDRESS and BLOCKCHAIR_API_KEY constants")
        return
    data = getData(ADDRESS, START_DATE, END_DATE, BLOCKCHAIR_API_KEY)
    if not data:
        print("no data returned")
        return
    summary = makeSummary(data, ADDRESS)
    if not summary:
        print("summary unavailable")
        return
    txDf = makeTx(data, ADDRESS)
    sumDf, txDf = makeFrames(summary, txDf)
    out = saveExcel(sumDf, txDf, ADDRESS)
    print(f"saved: {out}")

# auto-run for `python satReport.py`
run()
```
