
# 🛒 Small eShop Backend

A **high-performance RESTful API** for an e-commerce platform, built in **Rust** to ensure safety, speed, and reliability. Ideal for **small to medium-sized online stores**, this backend provides core functionalities such as user, category, and product management. Built with **hexagonal architecture** for maintainability and scalability.

---

## 🚀 Features

- **User Management:** Create and retrieve users with validated fields (email, names, role).
- **Category Management:** Full CRUD for product categories.
- **Product Management:** Full CRUD for products with category linkage, precise pricing using `BigDecimal`, and inventory tracking.
- **Database Integrity:** Enforced foreign key constraints and check constraints for prices and stock.
- **Error Handling:** Robust logging and HTTP status code responses.
- **Hexagonal Architecture:** Clean separation of concerns for testability and modularity.
- **Async/Await:** Fully non-blocking, scalable API.

---

## 🧱 Tech Stack

- **Language:** Rust (2021 Edition)
- **Web Framework:** [Actix-Web 4.9](https://actix.rs/)
- **Database:** PostgreSQL with [SQLx 0.7](https://docs.rs/sqlx/)
- **Dependencies:**
  - `uuid`: UUID generation
  - `chrono`: Timestamp handling
  - `sqlx::types::BigDecimal`: Accurate monetary calculations
  - `serde`: JSON (de)serialization
  - `log`, `env_logger`: Logging
  - `dotenv`: Environment management

---

## 🧩 Hexagonal Architecture (Ports & Adapters)

- **Domain (Core):** Business logic, core entities (`User`, `Category`, `Product`) and trait definitions for ports.
- **Application:** Use cases that orchestrate the domain, e.g., `CreateProductUseCase`.
- **Adapters:**
  - *Primary:* Actix-Web HTTP handlers (e.g., `api/produits.rs`)
  - *Secondary:* SQLx-based repositories (e.g., `infrastructure/repository`)
- **Ports:**
  - *Input Ports:* Service interfaces for use cases.
  - *Output Ports:* Repository interfaces for persistence.

> 🎯 **Goal**: Modular, swappable components with strong boundaries.

---

## 🔄 Example Flow

1. `POST /api/products` hits the Actix-Web route handler.
2. Handler invokes `CreateProductUseCase` via input port.
3. Use case validates input and calls `ProductRepository` (output port).
4. SQLx repository executes query to PostgreSQL.
5. Response is serialized and returned as JSON.

---

## 🗃️ Database Schema

```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL
);

CREATE TABLE products (
    id UUID PRIMARY KEY,
    category_id UUID REFERENCES categories(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    description TEXT NOT NULL,
    price_av DECIMAL(10,2) NOT NULL,
    price_ap DECIMAL(10,2) NOT NULL,
    stock_quantity INTEGER NOT NULL DEFAULT 0,
    image_url TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL,
    CONSTRAINT positive_price_av CHECK (price_av > 0),
    CONSTRAINT positive_price_ap CHECK (price_ap > 0),
    CONSTRAINT positive_stock CHECK (stock_quantity >= 0)
);

CREATE INDEX idx_products_category_id ON products(category_id);
````

---

## 📦 API Endpoints

### 🗂️ Categories

| Method | Endpoint               | Description         |
| ------ | ---------------------- | ------------------- |
| POST   | `/api/categories`      | Create a category   |
| GET    | `/api/categories`      | List all categories |
| GET    | `/api/categories/{id}` | Get category by ID  |
| PATCH  | `/api/categories/{id}` | Update a category   |
| DELETE | `/api/categories/{id}` | Delete a category   |

### 📦 Products

| Method | Endpoint             | Description       |
| ------ | -------------------- | ----------------- |
| POST   | `/api/products`      | Create a product  |
| GET    | `/api/products`      | List all products |
| GET    | `/api/products/{id}` | Get product by ID |
| PATCH  | `/api/products/{id}` | Update a product  |
| DELETE | `/api/products/{id}` | Delete a product  |

### 👤 Users

| Method | Endpoint     | Description    |
| ------ | ------------ | -------------- |
| POST   | `/api/users` | Create a user  |
| GET    | `/api/users` | List all users |

---

## 🧪 Example Request

### ➕ Create a Product

```bash
curl -X POST http://127.0.0.1:8080/api/products \
-H "Content-Type: application/json" \
-d '{
  "category_id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Smartphone",
  "description": "A high-end smartphone",
  "price_av": "699.99",
  "price_ap": "799.99",
  "stock_quantity": 50,
  "image_url": "https://example.com/smartphone.jpg"
}'
```

**Response**

```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "category_id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Smartphone",
  "description": "A high-end smartphone",
  "price_av": "699.99",
  "price_ap": "799.99",
  "stock_quantity": 50,
  "image_url": "https://example.com/smartphone.jpg",
  "created_at": "2025-06-14T12:25:00Z",
  "updated_at": "2025-06-14T12:25:00Z"
}
```

---

## 📁 Project Structure

```
small_eshop/
├── src/
│   ├── api/                    # Actix-Web route handlers
│   │   └── produits.rs
│   ├── application/            # Use cases (business orchestration)
│   │   ├── category_service.rs
│   │   └── product_service.rs
│   ├── domain/                 # Entities & traits
│   │   ├── entities/
│   │   └── ports/
│   ├── infrastructure/         # SQLx-based adapters
│   │   └── repository/
│   ├── models/                 # DTOs
│   │   └── produits.rs
│   └── main.rs
├── migrations/
├── .env
├── Cargo.toml
└── README.md
```

---

## ⚙️ Setup Instructions

### 📦 Prerequisites

* Rust (stable, edition 2021)
* PostgreSQL
* Cargo
* SQLx CLI:

  ```bash
  cargo install sqlx-cli --no-default-features --features postgres
  ```

### 🛠 Installation

```bash
git clone https://github.com/your-username/small-eshop-backend.git
cd small-eshop-backend
```

### 🔧 Environment Configuration

Create a `.env` file with:

```
DATABASE_URL=postgres://username:password@localhost:5432/small_eshop
```

### 🛢️ Database Setup

```bash
cargo sqlx database create
cargo sqlx migrate run
```

### 🚀 Run the Server

```bash
cargo run
```

API available at:
📍 `http://127.0.0.1:8080`

---

## ✅ Testing

You can test endpoints using:

* `curl`
* Postman
* Any HTTP client

### 🧪 Example Unit Test

```rust
#[tokio::test]
async fn test_create_product() {
    let repo = MockProductRepository::new();
    let service = ProductService::new(Arc::new(repo));
    let input = CreateProduct { /* ... */ };
    let result = service.create_product(input).await;
    assert!(result.is_ok());
}
```

---

## 📈 Future Improvements

* [ ] JWT Authentication
* [ ] Pagination & Filtering
* [ ] OpenAPI/Swagger Documentation
* [ ] SQL Triggers for `updated_at`
* [ ] Prometheus Monitoring Integration

---

## 🤝 Contributing

Contributions are welcome! Please:

* Open an issue or submit a PR
* Follow the hexagonal architecture pattern

