# E-Commerce Data I/O, Web Scraping, API & SQL Analytics Pipeline

## 📌 Project Overview

This project is an end-to-end **E-Commerce Data Engineering and Analytics Pipeline** built with Python, Pandas, BeautifulSoup, Regex, REST API concepts, SQLite, and SQL.

The pipeline demonstrates how raw product data can be:

1. Extracted from HTML web pages
2. Cleaned using Regex and Pandas
3. Enriched using API-based exchange-rate data
4. Cleaned and validated numerically
5. Stored in a relational SQLite database
6. Processed using SQL queries and JOIN operations
7. Aggregated into business analytics
8. Visualized for reporting

---

## 🎯 Project Objective

The main objective is to build a complete data pipeline that combines **Data I/O, Web Scraping, API Data Extraction, Text Preprocessing, Database Storage, SQL Processing, and Business Analytics**.

This project combines four major modules:

* **Module 1:** Web Scraping & Text Preprocessing
* **Module 2:** API Extraction & Numerical Data Cleaning
* **Module 3:** SQLite Database & Relational Schema
* **Module 4:** SQL Validation, JOINs & Analytics

---

# 🏗️ Project Architecture

```text
                    E-COMMERCE DATA PIPELINE
                              │
                              ▼
                   ┌─────────────────────┐
                   │   Web Scraping      │
                   │    BeautifulSoup    │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Text Preprocessing  │
                   │      Regex          │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ API Data Enrichment  │
                   │  Exchange Rates     │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Numerical Cleaning  │
                   │ & Missing Values    │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │   SQLite Database   │
                   │ Products + Orders   │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ SQL Validation      │
                   │ LEFT JOIN           │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ SQL Analytics       │
                   │ INNER JOIN          │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Visualization      │
                   │ Revenue Analytics  │
                   └─────────────────────┘
```

---

# 📚 Modules Covered

## Module 1 — Web Scraping & Text Preprocessing

### Technologies

* Python
* BeautifulSoup
* Regex
* Pandas

### Tasks

The first module uses a simulated HTML product page.

Product information extracted:

* Product name
* Product description
* Product price
* Stock status

BeautifulSoup is used to parse the HTML structure.

Regex is then used to clean the extracted data.

### Text preprocessing examples

```python
clean_title = re.sub(
    r"[^a-zA-Z0-9\s]",
    "",
    raw_title
)
```

Whitespace normalization:

```python
clean_desc = re.sub(
    r"\s+",
    " ",
    raw_desc
)
```

Price conversion:

```python
price_val = float(
    re.sub(
        r"[^\d.]",
        "",
        raw_price
    )
)
```

### Output

```text
data/raw_web_scraped_products.csv
```

---

# Module 2 — API Extraction & Numerical Data Cleaning

## Technologies

* Python
* JSON
* Pandas
* Regex

The second module enriches the product dataset with exchange-rate information.

Example API data:

```json
{
    "base": "USD",
    "rates": {
        "USD": 1.0,
        "EUR": 0.92,
        "GBP": 0.78,
        "INR": 83.15
    }
}
```

### Currency Conversion

USD prices are converted to EUR.

```python
df_scraped["price_eur"] = (
    df_scraped["price_usd"] * fx_rate_eur
).round(2)
```

### Stock Quantity Extraction

The stock quantity is extracted using Regex.

Example:

```text
In Stock (15)
```

becomes:

```text
15
```

For:

```text
Out of Stock
```

the missing quantity is replaced with:

```text
0
```

### Numerical Validation

The pipeline validates that product prices are greater than zero.

```python
assert (
    df_scraped["price_usd"] > 0
).all()
```

### Output

```text
data/api_exchange_rates.json
```

---

# Module 3 — SQLite Database & Schema Design

## Database

```text
SQLite
```

The cleaned data is stored in a relational database.

Database file:

```text
data/ecommerce_pipeline.db
```

## Tables

### Products Table

```text
products
```

Columns:

| Column         | Description         |
| -------------- | ------------------- |
| product_id     | Primary key         |
| product_name   | Product name        |
| description    | Product description |
| price_usd      | Product price       |
| stock_quantity | Available stock     |

### Orders Table

```text
orders
```

Columns:

| Column          | Description     |
| --------------- | --------------- |
| order_id        | Primary key     |
| product_id      | Foreign key     |
| order_date      | Date of order   |
| quantity_sold   | Quantity sold   |
| customer_region | Customer region |

---

# 🔗 Database Relationship

```text
PRODUCTS
-----------------------
PK product_id
product_name
description
price_usd
stock_quantity
        │
        │
        │ 1 : MANY
        │
        ▼
ORDERS
-----------------------
PK order_id
FK product_id
order_date
quantity_sold
customer_region
```

The `product_id` column connects the `products` and `orders` tables.

The database also uses:

* Primary Keys
* Foreign Keys
* CHECK constraints
* Parameterized SQL INSERT statements

---

# Module 4 — SQL Data Processing & Analytics

The fourth module performs SQL-based validation and business analytics.

## SQL Validation

A `LEFT JOIN` is used to check products and their associated orders.

```sql
SELECT
    p.product_id,
    p.product_name,
    COUNT(o.order_id) AS total_orders,
    COALESCE(
        SUM(o.quantity_sold),
        0
    ) AS units_sold
FROM products p
LEFT JOIN orders o
    ON p.product_id = o.product_id
GROUP BY
    p.product_id,
    p.product_name;
```

This provides:

* Total orders
* Total units sold
* Product-level validation

---

# 💰 Revenue Analytics

An `INNER JOIN` combines products and orders.

```sql
SELECT
    p.product_name,
    o.customer_region,
    SUM(o.quantity_sold) AS total_quantity,
    ROUND(
        SUM(
            o.quantity_sold * p.price_usd
        ),
        2
    ) AS total_revenue_usd
FROM orders o
INNER JOIN products p
    ON o.product_id = p.product_id
GROUP BY
    p.product_name,
    o.customer_region
ORDER BY
    total_revenue_usd DESC;
```

### Business Metrics

The query calculates:

* Product-level sales
* Regional sales
* Quantity sold
* Total revenue

Revenue is calculated as:

```text
Quantity Sold × Product Price
```

---

# 📊 Visualization

The SQL analytics results are visualized using:

* Matplotlib
* Seaborn

The generated visualization shows:

```text
Total Revenue by Product and Customer Region
```

Output:

```text
visuals/sql_join_analytics.png
```

---

# 📁 Project Structure

```text
e-commerce-data-pipeline-analytics/
│
├── README.md
├── LICENSE
│
├── data/
│   ├── raw_web_scraped_products.csv
│   ├── api_exchange_rates.json
│   └── ecommerce_pipeline.db
│
├── notebooks/
│   └── data_io_sql_pipeline.ipynb
│
└── visuals/
    ├── sql_join_analytics.png
    └── database_schema.png
```

---

# ⚙️ Installation

## 1. Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/e-commerce-data-pipeline-analytics.git
```

Navigate into the project:

```bash
cd e-commerce-data-pipeline-analytics
```

---

# 2. Create Virtual Environment

Windows PowerShell:

```powershell
py -m venv .venv
```

Activate:

```powershell
.\.venv\Scripts\Activate.ps1
```

---

# 3. Install Dependencies

```powershell
pip install pandas numpy beautifulsoup4 requests matplotlib seaborn openpyxl jupyter
```

---

# ▶️ How to Run

Open the project in VS Code.

Open:

```text
notebooks/data_io_sql_pipeline.ipynb
```

Select the project's Python environment:

```text
.venv
```

Then execute the notebook cells sequentially or select:

```text
Run All
```

---

# 📦 Generated Outputs

After successful execution, the pipeline generates:

```text
data/
├── raw_web_scraped_products.csv
├── api_exchange_rates.json
└── ecommerce_pipeline.db
```

and:

```text
visuals/
└── sql_join_analytics.png
```

---

# 🛠️ Technologies Used

| Category             | Technology                 |
| -------------------- | -------------------------- |
| Programming          | Python                     |
| Data Processing      | Pandas                     |
| Numerical Processing | NumPy                      |
| Web Scraping         | BeautifulSoup              |
| Text Processing      | Regex                      |
| API / JSON           | Requests / JSON            |
| Database             | SQLite                     |
| Query Language       | SQL                        |
| Visualization        | Matplotlib                 |
| Visualization        | Seaborn                    |
| Version Control      | Git                        |
| Repository           | GitHub                     |
| Development          | VS Code / Jupyter Notebook |

---

# 🎓 Skills Demonstrated

This project demonstrates practical knowledge of:

* Python programming
* Data extraction
* Web scraping
* HTML parsing
* Regex
* Text preprocessing
* API data handling
* JSON processing
* Numerical cleaning
* Missing-value handling
* Data validation
* Relational database design
* SQLite
* SQL JOINs
* SQL aggregation
* Business analytics
* Data visualization
* Git
* GitHub
* Project documentation

---

# 🚀 Future Improvements

Possible future improvements include:

* Replace simulated HTML with a real permitted web source
* Replace mock API data with a live API
* Add automated data pipeline scheduling
* Add unit tests
* Add logging
* Add data quality checks
* Add Docker support
* Add CI/CD
* Deploy the pipeline to the cloud
* Add a dashboard for business analytics

---

# 👨‍💻 Author

**Aviraj Ananda Sapate**

Data Analyst → MLOps / Data Engineering
