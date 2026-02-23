# E-Commerce ETL Pipeline

A production-grade ELT (Extract-Load-Transform) pipeline for a mid-size B2C e-commerce business, built with **dbt** and **Snowflake**.

## Overview

This pipeline ingests transactional data (orders, payments, marketing spend) into Snowflake and transforms it into analytics-ready datasets following a layered architecture:

```
RAW → STAGING → MART
```

### Key Features

- **Layered Architecture**: RAW → STAGING → MART for clear data lineage
- **dbt Transformations**: Minimal, readable, interview-safe models
- **Scalable Design**: Built to handle millions of records
- **Snowflake-Native**: Leverages Snowflake's compute and storage
- **Portfolio-Grade**: Production practices without over-engineering

## Quick Start

### Prerequisites

- Snowflake account with admin access
- dbt installed (`pip install dbt-snowflake`)
- Python 3.9+
- Git

### Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/loki-cloud0/ecommerce-etl-pipeline.git
   cd ecommerce-etl-pipeline
   ```

2. **Configure Snowflake connection**
   ```bash
   cp profiles.yml ~/.dbt/profiles.yml
   # Edit profiles.yml with your Snowflake credentials
   ```

3. **Create RAW schema and load sample data**
   ```bash
   bash scripts/setup.sh
   bash scripts/load_seeds.sh
   ```

4. **Run dbt**
   ```bash
   dbt deps
   dbt run
   dbt test
   ```

## Data Architecture

### Source Systems

| System | Table | Frequency | Records |
|--------|-------|-----------|---------|
| Orders System | `RAW.ORDERS` | Daily | ~10K-100K/day |
| Payment Processor | `RAW.PAYMENTS` | Daily | ~10K-100K/day |
| Marketing Platform | `RAW.MARKETING_SPEND` | Daily | ~1K-10K/day |

### Transformation Layers

**RAW**: Unmodified source data loaded as-is
**STAGING**: Cleaned, deduplicated, type-cast data with business logic applied
**MART**: Fact and dimension tables ready for analysis

### Key Models

#### Staging Layer
- `stg_orders`: Cleaned order transactions
- `stg_payments`: Payment reconciliation data
- `stg_marketing_spend`: Marketing campaign performance

#### Mart Layer
- `fct_orders`: Fact table for order analysis
- `fct_payments`: Payment success metrics
- `dim_customers`: Customer dimension
- `fct_marketing_metrics`: Marketing ROI and attribution

## Documentation

- [Architecture Guide](./ARCHITECTURE.md) - Detailed design decisions
- [Data Dictionary](./DATA_DICTIONARY.md) - Column definitions
- [Setup Guide](./docs/SETUP_GUIDE.md) - Step-by-step configuration
- [Workflow](./docs/WORKFLOW.md) - Daily pipeline operations

## Project Structure

```
models/          → dbt transformation models
├── staging/     → Intermediate cleansing layer
├── mart/        → Analytics-ready fact/dimension tables
└── schema.yml   → dbt YAML configurations and tests

data/            → Sample CSV data for testing
scripts/         → Shell scripts for setup and seeding
docs/            → Additional documentation
```

## Running the Pipeline

### Full Pipeline Run
```bash
dbt run
dbt test
```

### Run Specific Models
```bash
dbt run --select stg_orders
dbt run --select fct_orders
```

### Lineage & Documentation
```bash
dbt docs generate
dbt docs serve  # Opens browser documentation
```

## Project Metrics

After running the pipeline, query the MART tables:

```sql
-- Daily order revenue
SELECT DATE, SUM(ORDER_TOTAL) as daily_revenue
FROM MART.FCT_ORDERS
GROUP BY DATE
ORDER BY DATE DESC;

-- Payment success rate
SELECT 
    PAYMENT_STATUS,
    COUNT(*) as transaction_count,
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER (), 2) as pct
FROM MART.FCT_PAYMENTS
GROUP BY PAYMENT_STATUS;

-- Marketing spend efficiency
SELECT 
    CAMPAIGN_NAME,
    TOTAL_SPEND,
    ATTRIBUTED_REVENUE,
    ROUND(ATTRIBUTED_REVENUE / TOTAL_SPEND, 2) as roi
FROM MART.FCT_MARKETING_METRICS
ORDER BY ROI DESC;
```

## Development Workflow

1. **Create a feature branch** for model changes
2. **Write models** in `/models/staging/` or `/models/mart/`
3. **Define tests** in `schema.yml`
4. **Run locally**: `dbt run && dbt test`
5. **Submit PR** for code review
6. **Deploy** after approval

## Testing

dbt includes built-in tests. Run all tests:

```bash
dbt test
```

Run specific test:
```bash
dbt test --select stg_orders
```

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## Performance Considerations

- **Incremental Models**: Staging models use full refreshes (designed for daily batch)
- **Clustering**: Fact tables clustered on date columns for query optimization
- **Materialization**: Staging as views, Mart as tables
- **Scalability**: Tested up to 100M+ records

**Last Updated**: 2026-02-23 12:44:39  
**Team**: Data Engineering
