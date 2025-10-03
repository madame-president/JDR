# Client X

## Client X uses Koinly (premium) for monthly reconciliations of transactions. They are currently experiencing the following pains:

1. Koinly is great for basic bookkeeping and retail investors, not corporations with 100k + UTXOs

2. Slow imports

3. Cloud limitations since Koinly is a SaaS and cannot efficiently query intitutional-scale datasets

## Proposed improvement:

The idea of pulling data using API calls is correct, but Koinly is not the best tool for your use case. I suggest an institutional infrastructure provider like Blockchair.com, and a purchase plan of 500,000 API requests for $500 USD, which are set to expire within one year of purchase.

After the API purchased is completed, the following documentation suggests a more efficient design system built using blockchair.

1. Local Caching Database (PostgreSQL / MongoDB)

**Store fetched data for repeatable queries.**

```
**Organize by:**

address

transaction_id

UTXO_status (spent/unspent)

timestamp

value (sats/BTC)

```
2. Data Processing & Normalization

**ETL (Extract, Transform, Load) Pipeline**

Run scheduled jobs to normalize data into audit-standard fields (date, amount, counterparty, wallet group).

Apply business rules: classify wallets into custody, treasury, hot, cold.

De-duplicate transactions across multiple addresses.

3. Reporting & Analytics

**Financial Metrics Dashboard**

Total BTC under custody.

Net inflows/outflows (daily, monthly, quarterly).

Address distribution (concentration in top wallets).

Reserve ratios (assets vs liabilities, once liability data is integrated).

Audit-Approved Reports (PDF/CSV)

Address-level ledger export.

Historical reconciliations at specific cutoff dates.

Proof-of-Reserves templates (assets ≥ liabilities evidence).

Liabilities Reconciliation Module (future expansion)

Import customer balance data from exchange DB.

Compare aggregated liabilities vs UTXO balances.

Produce solvency snapshots.

**This page was generated using artificial intelligence for demo purposes**
