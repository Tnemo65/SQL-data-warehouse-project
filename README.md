# SQL Data Warehouse Project

## Overview

This project implements a modern three-tier data warehouse architecture using SQL Server, following the Bronze-Silver-Gold medallion architecture pattern. The system processes data from multiple sources (CRM and ERP systems) through progressive layers of refinement to create analytics-ready datasets.

## Architecture

### Data Flow
```
Raw Data (CSV/Excel) → Bronze Layer → Silver Layer → Gold Layer → Analytics/BI
```

### Layer Descriptions

- **Bronze Layer**: Raw data ingestion from source files with minimal processing
- **Silver Layer**: Cleansed, validated, and standardized data with business rules applied
- **Gold Layer**: Business-ready star schema with dimension and fact views for analytics

## Database Schema

### Bronze Layer Tables
- `bronze.crm_cust_info` - Customer information from CRM
- `bronze.crm_prd_info` - Product information from CRM
- `bronze.crm_sales_details` - Sales transaction details
- `bronze.erp_loc_a101` - Location data from ERP
- `bronze.erp_cust_az12` - Customer demographics from ERP
- `bronze.erp_px_cat_g1v2` - Product category information

### Silver Layer Tables
- `silver.crm_cust_info` - Cleansed customer data
- `silver.crm_prd_info` - Standardized product data
- `silver.crm_sales_details` - Validated sales transactions
- `silver.erp_loc_a101` - Normalized location data
- `silver.erp_cust_az12` - Validated customer demographics
- `silver.erp_px_cat_g1v2` - Clean product categories

### Gold Layer Views
- `gold.dim_customers` - Customer dimension with integrated CRM/ERP data
- `gold.dim_products` - Product dimension with category information
- `gold.fact_sales` - Sales fact table with foreign keys to dimensions

## Data Transformations

### Silver Layer Transformations
- **Customer Data**: Deduplication, name trimming, gender/marital status standardization
- **Product Data**: Category ID extraction, price validation, product line mapping
- **Sales Data**: Date format conversion, price/quantity validation, sales amount recalculation
- **Location Data**: Country code normalization
- **Demographics**: Birthdate validation, gender standardization

### Gold Layer Features
- **Star Schema**: Optimized for analytics with fact and dimension tables
- **Surrogate Keys**: Auto-generated keys for dimension tables
- **Data Integration**: Combines CRM and ERP data sources
- **Business Logic**: Applies complex business rules and calculations

## Usage Examples

### Query Customer Sales
```sql
SELECT 
    c.first_name,
    c.last_name,
    c.country,
    SUM(f.sales_amount) as total_sales
FROM gold.fact_sales f
JOIN gold.dim_customers c ON f.customer_key = c.customer_key
GROUP BY c.first_name, c.last_name, c.country
ORDER BY total_sales DESC;
```

### Product Performance Analysis
```sql
SELECT 
    p.product_name,
    p.category,
    p.product_line,
    COUNT(f.order_number) as order_count,
    SUM(f.quantity) as total_quantity,
    AVG(f.price) as avg_price
FROM gold.fact_sales f
JOIN gold.dim_products p ON f.product_key = p.product_key
GROUP BY p.product_name, p.category, p.product_line
ORDER BY total_quantity DESC;
```
