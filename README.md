# DiD-Trade

Here is the Repository for *Difference-in-Differences* model for COMEXT trade data, used in analysis of EU Economic Sanctions on Russia.

This repository contains the code and resources used in the master's thesis titled:

**"Empirical analysis of the effectiveness of European Union economic sanctions against the Russian Federation in 2022–2024"**

---

## Project Overview

This study investigates the impact of EU-imposed economic sanctions on trade with Russia. Using the **Difference-in-Differences (DiD)** methodology and trade data from **Eurostat Comext**, the analysis focuses on the direct effects of sanctions on trade flows.

Code was compiled using **R** and **STATA**. Data comes from the public **Eurostat COMEXT** database.

---

## 1. Getting The Data

### Data Description

This project uses monthly international trade data from **Eurostat Comext** (2012–2024), covering both intra-EU and extra-EU trade flows. The dataset is structured by **Harmonized System (HS)** product codes and includes trade value, reporting and partner countries, trade type, and product classification.

To assess the impact of EU sanctions on Russia, the dataset was enriched with:
- **Sanctioned goods lists** and timelines from EUR-Lex, the EU’s legal database.
- **Control variables** such as GDP per capita and population from the **World Bank** and **IMF**.

The final dataset contains over **650 million monthly observations**, each uniquely identifying trade flows by country pair, product, and time. Due to size, the database is split into 15 files in `.parquet` format.

---

### Data Access & Processing Workflow

1. **Connecting to Eurostat Comext**  
   Data is accessed via the Eurostat Comext dissemination API, which provides monthly trade files in `.7z` format.

2. **Downloading in Batches**  
   An R script loops through all months from 2010 to 2024, downloading each file individually using the `httr` package.

3. **Extracting & Reading Data**  
   Files are extracted using the `archive` package and read into R using `readr::read_delim()`.

4. **Merging & Optimizing**  
   All monthly datasets are merged using `dplyr::bind_rows()` and saved in efficient `.parquet` format using the `arrow` package.

5. **Cleaning & Structuring**  
   Data is cleaned and filtered using `dplyr`, `tidyr`, and `stringr` to ensure consistency and prepare it for econometric analysis.

---

## 2. Sourcing The Data

Due to the large size of the full trade database (over 650 million records), data is not processed all at once. Instead, subsets are extracted on demand for specific econometric analyses. This modular approach ensures efficient computation and memory usage.

For example, when analyzing monthly intra-EU trade flows, only the relevant subset of data is pulled from the full database, cleaned, and saved as a dedicated `.dta` file containing only the necessary observations.

### Monthly Data

Monthly data is extracted from segmented `.parquet` files, each representing a specific year. These files are grouped by:
- Reporting country (`DECLARANT_ISO`)
- Partner country (`PARTNER_ISO`)
- Trade flow direction (`FLOW`)
- Month (`PERIOD`)
- Product (`HS CODE`)

The data is aggregated to produce a panel of monthly bilateral trade values. This panel is then saved in both `.parquet` and `.dta` formats for use in STATA-based econometric models.

### Yearly Data

For broader analyses, such as long-term trends or yearly comparisons, the same segmented files are aggregated by year. The process involves:
- Summing trade values across all months within each year
- Grouping by country pairs and trade flow
- Exporting the results as yearly `.dta` files

This structure allows flexible access to both high-frequency (monthly) and low-frequency (yearly) trade data.

---

## 3. Estimation

The core of the analysis relies on **Difference-in-Differences (DiD)** and **event study** models to estimate the causal impact of EU sanctions on trade with Russia and potential diversion partners. Estimations are performed using **STATA**, with data pre-processed in **R**.

### Econometric Strategy

The estimation strategy includes:

- **Baseline DiD models** comparing trade flows with Russia before and after the imposition of sanctions.
- **Extended DiD models** with control variables such as GDP, population, and trade openness.
- **Event study models** to capture the dynamic effects of sanctions over time.
- **Country-specific DiD regressions** to assess heterogeneous treatment effects across EU member states.
- **Product-level analysis** using HS codes to evaluate the impact on sanctioned vs. non-sanctioned goods.

### Model Implementation

#### Panel Setup

All models are estimated on **panel data** structured by:
- Country pairs (`DECLARANT_ISO`, `PARTNER_ISO`)
- Time (`Year` or `Month`)
- Product (`HS Code`, for product-level models)

Panel identifiers are created using `egen group()` and declared with `xtset`.

#### Baseline DiD

```stata
xtdidregress (ln_TOTAL_VALUE ln_DECLARANT_GDP ln_PARTNER_GDP ln_DECLARANT_POPULATION ln_PARTNER_POPULATION) ///
              (TREATMENT), group(PAIR_ID) time(Year)
```

Where `TREATMENT` is defined as trade with Russia after February 2022.

#### Country-Specific DiD

A looped estimation is performed for each partner country to assess the **Average Treatment Effect on the Treated (ATET)** individually. Results are exported and visualized.

#### Event Study

To explore the **temporal dynamics** of the treatment effect, an event study framework is implemented using `reghdfe`:

```stata
reghdfe ln_TOTAL_VALUE pre* post*, absorb(PAIR_ID DATE) cluster(PAIR_ID)
```

Where `pre*` and `post*` are dummy variables for each month before and after the sanctions.

### Output

- **Regression tables** are saved in `.txt` and `.csv` formats using `esttab`.
- **Event study plots** are exported as `.jpg` files showing the evolution of trade effects over time.
- **Final datasets** are saved in `.dta` and `.parquet` formats for reproducibility.
