# Data Dictionary

## RAW Layer Tables

### 1. ORDERS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| order_id          | INT       | Unique identifier for each order       | Primary key                             |
| customer_id       | INT       | Identifier for the customer             | Foreign key referencing customers       |
| order_date        | DATETIME  | Date and time when the order was placed| Set when the order is created           |
| total_amount      | DECIMAL   | Total amount for the order             | Sum of all items in the order          |
| status            | VARCHAR   | Current status of the order            | E.g., 'Pending', 'Completed', 'Cancelled'|

### 2. PAYMENTS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| payment_id        | INT       | Unique identifier for each payment     | Primary key                             |
| order_id          | INT       | Identifier for the related order       | Foreign key referencing orders          |
| payment_date      | DATETIME  | Date and time when the payment was made| Set when the payment is completed       |
| amount            | DECIMAL   | Payment amount                         | Should match the total amount in ORDERS |
| payment_method    | VARCHAR   | Method of payment                       | E.g., 'Credit Card', 'PayPal'          |

### 3. MARKETING_SPEND
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| spend_id          | INT       | Unique identifier for marketing record  | Primary key                             |
| campaign_id       | INT       | Identifier for the marketing campaign    | Foreign key references campaign details  |
| spend_date        | DATETIME  | Date of the marketing spend             | Set when the spend is recorded          |
| amount            | DECIMAL   | Amount spent on marketing               | Total spend for the campaign            |

## STAGING Layer Tables

### 1. STG_ORDERS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| stg_order_id      | INT       | Unique identifier for staged orders    | Primary Key                             |
| raw_order_id      | INT       | Original order ID from RAW layer        | Reference to RAW ORDERS                 |
| load_date         | DATETIME  | Date and time when data was loaded     | Set when the data is staged             |

### 2. STG_PAYMENTS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| stg_payment_id    | INT       | Unique identifier for staged payments   | Primary Key                             |
| raw_payment_id    | INT       | Original payment ID from RAW layer      | Reference to RAW PAYMENTS               |
| load_date         | DATETIME  | Date and time when data was loaded     | Set when the data is staged             |

### 3. STG_MARKETING_SPEND
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| stg_spend_id      | INT       | Unique identifier for staged spend      | Primary Key                             |
| raw_spend_id      | INT       | Original spend ID from RAW layer        | Reference to RAW MARKETING_SPEND        |
| load_date         | DATETIME  | Date and time when data was loaded     | Set when data is staged                 |

## MART Layer Tables

### 1. FCT_ORDERS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| fct_order_id      | INT       | Unique identifier for fact order       | Primary key                             |
| order_id          | INT       | Identifier for the original order      | Foreign key referencing RAW ORDERS      |
| total_amount      | DECIMAL   | Total amount for the order             | Aggregated value                        |
| customer_id       | INT       | Identifier for the customer             | Foreign key referencing customers       |

### 2. FCT_PAYMENTS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| fct_payment_id    | INT       | Unique identifier for fact payment     | Primary key                             |
| payment_id        | INT       | Identifier for the original payment    | Foreign key referencing RAW PAYMENTS    |
| amount            | DECIMAL   | Payment amount                         | Stored value                            |

### 3. FCT_MARKETING_METRICS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| fct_metric_id     | INT       | Unique identifier for marketing metrics | Primary key                             |
| campaign_id       | INT       | Identifier for the marketing campaign   | Foreign key referencing marketing data   |
| total_spend       | DECIMAL   | Total spend for the campaign            | Sum of spends                           |
| performance_metric | DECIMAL   | Metric to evaluate campaign performance | Dynamic based on business rules         |

### 4. DIM_CUSTOMERS
| Column Name       | Data Type | Description                             | Business Logic                           |
|-------------------|-----------|-----------------------------------------|-----------------------------------------|
| customer_id       | INT       | Unique identifier for customers        | Primary key                             |
| customer_name     | VARCHAR   | Name of the customer                   | Derived from orders                     |
| join_date         | DATETIME  | Date the customer joined               | Used for segmentation                   |
| total_spends       | DECIMAL   | Total spending of the customer        | Running total from FCT_PAYMENTS        |