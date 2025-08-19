flowchart TD
    %% ========== SOURCES ==========
    A[("Source\nmaster_customer_data\n(CSV/Parquet upload to DBFS)")]
    
    %% ========== INGESTION ==========
    subgraph S1[Ingestion & Storage]
      B[Ingestion Notebook (PySpark)\nread -> write Delta]
      C[(Bronze Delta\nraw.master_customer_data)]
      A --> B --> C
    end

    %% ========== CLEANING ==========
    subgraph S2[Cleaning & Standardization]
      D[Cleaning Notebook (PySpark)\n• Standardize casing/columns\n• Fix typos: 'sucess'→'success'\n• Impute: signup_date=order_date\n• Impute: product_category='Unknown'\n• Impute: payment_status='Pending'\n• Cap product_price: ₹500–₹100,000\n• Filter invalid/null FKs]
      E[(Silver Delta\nclean.master_customer_data)]
      C --> D --> E
    end

    %% ========== MODELING ==========
    subgraph S3[Snowflake Modeling (PySpark SQL)]
      F[Modeling Notebook\nSplit & Normalize]
      G[(customers)]
      H[(orders)]
      I[(products)]
      J[(payments)]
      E --> F
      F --> G
      F --> H
      F --> I
      F --> J
    end

    %% ========== GOLD / MARTS ==========
    subgraph S4[Gold Layer / Marts]
      K[(Gold Delta Marts)\nAOV by region • Monthly revenue\nTop categories • High-value customers\nPayment method split]
      G --> K
      H --> K
      I --> K
      J --> K
    end

    %% ========== CONSUMPTION ==========
    subgraph S5[Consumption]
      L[Databricks SQL Warehouse]
      M[Databricks SQL Dashboard\nKPIs: Total Revenue • Orders by Region • Payment Split • Category Leaderboard • HVC Heatmap]
      K --> L --> M
    end

    %% ========== ORCHESTRATION & GOVERNANCE ==========
    subgraph S6[Orchestration & Quality]
      O[Databricks Jobs Scheduler\nRuns Notebooks on a cadence]
      Q[Data Quality Checks\n(Python asserts/SQL tests)]
    end
    O -.-> B
    O -.-> D
    O -.-> F
    D -.-> Q
    F -.-> Q
