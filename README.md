# DiD-Trade
Here is the Repository for *Difference in difference* model for COMEXT trade data, used in analysis of EU Economic Sanctions on Russia

This repository contains the code and resources used in the master's thesis titled:

**"Empirical analysis of the effectiveness of European Union economic sanctions against the Russian Federation in 2022–2024"**

## 📘 Project Overview

This study investigates the impact of EU-imposed economic sanctions on trade with Russia. Using the **Difference-in-Differences (DiD)** methodology and trade data from **Eurostat Comext**, the analysis focuses on the direct effects of sanctions on trade flows.

Code was compiled using R and STATA. Data comes from public Eurostat COMEXT database.


## 1. Getting The Data

### 📊 Data Description

This project uses monthly international trade data from **Eurostat Comext** (2012–2024), covering both intra-EU and extra-EU trade flows. The dataset is structured by **Harmonized System (HS)** product codes and includes trade value, reporting and partner countries, trade type, and product classification.

To assess the impact of EU sanctions on Russia, the dataset was enriched with:
- **Sanctioned goods lists** and timelines from EUR-Lex, the EU’s legal database.
- **Control variables** such as GDP per capita and population from the **World Bank** and **IMF**.

The final dataset contains over **650 million monthly observations**, each uniquely identifying trade flows by country pair, product, and time.

---

### 🔄 Data Access & Processing Workflow

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
