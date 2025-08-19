erDiagram
    customers {
      INT customer_id PK
      STRING region
      DATE signup_date
    }

    products {
      INT product_id PK
      STRING product_category
      DECIMAL product_price
    }

    orders {
      INT order_id PK
      INT customer_id FK
      DATE order_date
      INT product_id FK
      DECIMAL product_price
    }

    payments {
      INT payment_id PK
      INT order_id FK
      STRING payment_method
      STRING payment_status
    }

    customers ||--o{ orders : "places"
    products  ||--o{ orders : "contained_in"
    orders    ||--o{ payments : "settled_by"
