# Fashion Brand Pricing & Returns Analysis

## Background

This project analyzes real fashion product data from four mainstream brands sold on Myntra — GAP, Tommy Hilfiger, Levis, and U.S. Polo Assn. — combining Python, SQL, an AI-assisted natural-language query layer, and Power BI into one end-to-end pipeline, reflecting how real analysts increasingly work across tools.

**Note on H&M:** H&M was originally targeted for this analysis but is not present in this dataset, which reflects reality — H&M operates through its own standalone stores/app in India rather than through Myntra.

## Important note on return data

No public dataset exists with real return data tied to named fashion brands (companies do not publicly release this). To enable return-pattern analysis, a `simulated_returned` column was generated using documented, transparent assumptions:
- A 10% base return chance for any product
- +10% if the item is in the top 25% most expensive (above ₹1,999)
- +8% if the product name contains a fit-sensitive keyword (jeans, dress, fit, skinny)
- Capped at 35%

**This column is clearly synthetic and does not reflect real return behavior from these brands.** It is used only to demonstrate return-pattern analysis technique, and is never presented as a real-world finding about these brands.

## Dataset

- **Source:** Fashion Clothing Products Catalog (Myntra), Kaggle
- **Size:** 408 real products across 4 brands, after filtering from a ~12,000 product catalog
- **Columns:** Product name, brand, gender, price, number of images, description, primary color, plus the engineered `product_type`, `Price_Tier`, and simulated return columns

## Tools

Python (pandas), SQL (SQLite), Google Gemini API (AI-assisted SQL generation), Power BI (Power Query, DAX)

## Pipeline

1. **Python:** Loaded and cleaned the raw catalog, filtered to 4 target brands, handled missing color values, engineered a `product_type` column
2. **SQL:** Loaded cleaned data into SQLite; wrote aggregation queries to analyze pricing patterns
3. **AI-assisted queries:** Used the Gemini API to generate SQL from plain-English questions, with every query manually verified before running (see below)
4. **Power BI:** Built a dashboard with KPI cards, a brand pricing chart, price-tier composition, a Top 3 most-expensive-products-per-brand table (using RANKX), and a price-vs-images correlation analysis

## Key findings

- **Levis has the highest average price (₹3,442)** — more than double GAP, Tommy Hilfiger (~₹1,400 each), and over 4x U.S. Polo Assn. (₹762).
- **Levis' premium price is not driven by jeans** — its jeans are actually cheaper than Tommy Hilfiger's. The premium instead comes from other categories (~₹3,520 average), including its top 3 most expensive individual items: two boots and a jacket.
- **Across all four brands, the single most expensive item was never the brand's own signature category** — GAP's top item was a hoodie, Levis' was a boot, Tommy Hilfiger's was a dress, and U.S. Polo Assn.'s was a sneaker. This suggests premium pricing in this catalog consistently comes from non-core categories rather than each brand's flagship line.
- **Levis has the highest proportion of Premium-tier products (~40%)**, consistent with its higher average price.
- **Price and number of product images show a moderate positive correlation (0.52)** — more expensive items tend to have more images, though the relationship isn't strict.
- **Average price can be a misleading signal on its own.** U.S. Polo Assn. has the lowest average price (₹762) but the *largest* proportional gap between average and median (+54%) of any brand — larger than Levis (+15%), Tommy Hilfiger (+12%), or GAP (+8%). Its typical product sits around ₹495, with a smaller number of higher-priced items (up to ₹2,999) pulling the average upward. This shows average price alone can misrepresent a brand's typical price point without checking the median.
- **Gender-based pricing varies by brand, with no consistent direction.** Levis prices Women's items ~27% higher than Men's (₹4,332 vs ₹3,400) — the largest gender gap observed — while GAP shows the opposite pattern, pricing Men's items ~10% higher than Women's (₹1,550 vs ₹1,414). U.S. Polo Assn. had no Women's products in this dataset, so no comparison was possible for that brand.

## AI-assisted SQL: verification examples

Every AI-generated query was checked against the schema and logic before running — not accepted blindly:

1. **Correct but incomplete:** Asked which brand had the highest average price. The AI's query was logically correct but only returned the brand name, not the actual price — a human-written version was used instead to show the full value.
2. **Fully verified:** Asked for Tommy Hilfiger's return percentage. The AI's result (12.82%) matched an earlier manual SQL finding (12.8%) exactly, confirming correctness.
3. **Correct edge case:** Asked how many H&M products exist. The AI correctly returned 0 rather than hallucinating a plausible-sounding but fake number.
4. **Caught a real logic error:** Asked for the average price of items "likely to be returned." The AI used the `return_possibility` probability column with a condition (`= 1`) that could never be true, since that column holds decimal values (e.g. 0.28), not a 0/1 outcome. The correct column, `simulated_returned`, was used instead after catching this.

## Files

- `01_fashion_brand_analysis.ipynb` — Python cleaning, SQL analysis, and AI query layer
- `fashion_brand_dashboard.pbix` — Power BI dashboard
