# ETL_GDP

A small Python ETL (Extract–Transform–Load) pipeline that scrapes country-level nominal GDP data from an archived Wikipedia page, cleans and converts it, and loads the results into both a CSV file and a SQLite database — with each stage logged to a text file.

## What it does

`etl_project_gdp.py` runs a full ETL pipeline in one script, built from five reusable functions:

| Function | Purpose |
|---|---|
| `extract(url, table_attribs)` | Fetches the source page with `requests`, parses the relevant HTML table with `BeautifulSoup`, and builds a DataFrame of `Country` / `GDP_USD_millions` |
| `transform(df)` | Converts the GDP figures from millions to billions (rounded to 2 decimal places) and renames the column to `GDP_USD_billions` |
| `load_to_csv(df, csv_path)` | Writes the transformed DataFrame to a CSV file |
| `load_to_db(df, sql_connection, table_name)` | Writes the transformed DataFrame to a table in a SQLite database (replacing it if it already exists) |
| `run_query(query_statement, sql_connection)` | Runs a SQL query against the database and prints the result |
| `log_progress(message)` | Appends a timestamped line to a log file after each pipeline stage |

The script runs end to end: extract → transform → save to CSV → save to database → run a sample query (`GDP_USD_billions >= 100`) → close the connection, logging progress at every step.

## Data source

Data is scraped from a Wayback Machine snapshot of the "List of countries by GDP (nominal)" Wikipedia page:

```
https://web.archive.org/web/20230902185326/https://en.wikipedia.org/wiki/List_of_countries_by_GDP_%28nominal%29
```

Using an archived snapshot keeps the table structure stable for the pipeline, even if the live page changes.

## Project structure

```
ETL_GDP/
├── etl_project_gdp.py       # The ETL pipeline (extract, transform, load, query, log)
├── Countries_by_GDP.csv     # Output: transformed data as CSV
├── World_Economies.db       # Output: transformed data loaded into SQLite (table: Countries_by_GDP)
├── etl_project_log.txt      # Output: timestamped log of each pipeline stage
└── README.md
```

## Requirements

- Python 3.8+
- [requests](https://pypi.org/project/requests/)
- [beautifulsoup4](https://pypi.org/project/beautifulsoup4/)
- [pandas](https://pypi.org/project/pandas/)
- [numpy](https://pypi.org/project/numpy/)

`sqlite3` and `datetime` are part of the Python standard library.

```bash
pip install requests beautifulsoup4 pandas numpy
```

## Usage

```bash
git clone https://github.com/Dajoe312/ETL_GDP.git
cd ETL_GDP
python etl_project_gdp.py
```

Running the script will:
1. Scrape and parse the GDP table from the archived page.
2. Convert GDP values to billions of USD.
3. Overwrite `Countries_by_GDP.csv` with the latest data.
4. Overwrite the `Countries_by_GDP` table in `World_Economies.db`.
5. Print the countries with GDP ≥ 100 billion USD.
6. Append a fresh set of timestamped entries to `etl_project_log.txt`.

## Inspecting the results

```bash
sqlite3 World_Economies.db
sqlite> SELECT * FROM Countries_by_GDP WHERE GDP_USD_billions >= 100;
```

Or with `pandas`:

```python
import pandas as pd
df = pd.read_csv("Countries_by_GDP.csv")
print(df.head())
```

## Possible improvements

- Add error handling for network requests and for cases where the page's table structure changes.
- Make the query and output paths configurable via command-line arguments instead of hardcoded values.
- Add a `requirements.txt` for easier environment setup.

## License

No license specified.
