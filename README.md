Small eShop Backend
Overview
Small eShop Backend is a high-performance RESTful API for an e-commerce platform, built with Rust to ensure safety, speed, and reliability. It provides core functionalities for managing users, categories, and products, designed for small to medium-sized online stores. The backend leverages Actix-Web for web handling, SQLx for type-safe database interactions with PostgreSQL, and adopts an hexagonal architecture (ports and adapters) to promote modularity, testability, and maintainability.
Features

User Management: Create and retrieve user accounts with validation for email, first name, last name, and role.
Category Management: Full CRUD operations for product categories, including name and description.
Product Management: Full CRUD operations for products with category associations, precise monetary calculations using sqlx::types::BigDecimal, and inventory tracking.
Database Integrity: Enforces foreign key constraints (e.g., category_id in products) with cascading deletes, and check constraints for positive prices and non-negative stock.
Error Handling: Robust error handling with detailed logging and appropriate HTTP status codes (e.g., 400 for invalid input, 404 for not found).
Hexagonal Architecture: Separates business logic from infrastructure, enabling easy swapping of databases or frameworks.
Async/Await: Non-blocking operations for high scalability.

Tech Stack

Language: Rust (Edition 2021)
Web Framework: Actix-Web 4.9
Database: PostgreSQL with SQLx 0.7 (async, type-safe queries)
Dependencies:
uuid: UUID generation for unique identifiers
chrono: Timestamp management
sqlx::types::BigDecimal: Precise monetary calculations
serde: JSON serialization/deserialization
log and env_logger: Logging
dotenv: Environment configuration



Hexagonal Architecture
The backend follows an hexagonal architecture (also known as ports and adapters) to decouple business logic from infrastructure concerns. This design enhances testability, maintainability, and flexibility.
Components

Domain (Core):

Contains business entities (User, Category, Product) and core logic.
Defines ports (traits) for repository and service interfaces, e.g., CategoryRepository, ProductService.
Located in src/domain.


Application:

Implements use cases (services) that orchestrate business logic, e.g., CreateProductUseCase.
Interacts with the domain via ports.
Located in src/application.


Adapters:

Primary Adapters: Handle incoming requests (e.g., Actix-Web handlers in src/api).
Secondary Adapters: Implement ports for external systems (e.g., SQLx-based repositories in src/infrastructure).
Map domain entities to/from external formats (e.g., JSON, SQL).


Ports:

Input Ports: Define service interfaces for use cases (e.g., create_category, get_products).
Output Ports: Define repository interfaces for data persistence (e.g., save_product, find_category_by_id).



Benefits

Modularity: Business logic is independent of the web framework or database.
Testability: Easy to mock repositories and services for unit tests.
Flexibility: Swap SQLx/PostgreSQL for another database or Actix-Web for another framework with minimal changes.

Example Flow

A POST /api/products request hits the Actix-Web handler (create_product in src/api/produits.rs).
The handler calls the CreateProductUseCase (application layer) via an input port.
The use case validates the input and interacts with the ProductRepository (output port) to persist the product.
The SQLx-based ProductRepository (secondary adapter) executes the SQL query to save the product in PostgreSQL.
The response is mapped back to JSON and returned to the client.

Database Schema
The backend uses two main tables:
CREATE TABLE categories (
    id UUID PRIMARY KEY,
    name TEXT NOT NULL,
    description TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL
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
    created_at TIMESTAMP WITH TIME ZONE NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL,
    CONSTRAINT positive_price_av CHECK (price_av > 0),
    CONSTRAINT positive_price_ap CHECK (price_ap > 0),
    CONSTRAINT positive_stock CHECK (stock_quantity >= 0)
);

CREATE INDEX idx_products_category_id ON products(category_id);

API Endpoints
Categories



Method
Endpoint
Description
Request Body
Response



POST
/api/categories
Create a category
CreateCategory JSON
201: Category JSON


GET
/api/categories
List all categories
None
200: [Category] JSON


GET
/api/categories/{id}
Get a category by ID
None
200: Category JSON


PATCH
/api/categories/{id}
Update a category
UpdateCategory JSON
200: Category JSON


DELETE
/api/categories/{id}
Delete a category (cascades)
None
204: No Content


Products



Method
Endpoint
Description
Request Body
Response



POST
/api/products
Create a product
CreateProduct JSON
201: Product JSON


GET
/api/products
List all products
None
200: [Product] JSON


GET
/api/products/{id}
Get a product by ID
None
200: Product JSON


PATCH
/api/products/{id}
Update a product
UpdateProduct JSON
200: Product JSON


DELETE
/api/products/{id}
Delete a product
None
204: No Content


Users



Method
Endpoint
Description
Request Body
Response



POST
/api/users
Create a user
CreateUser JSON
201: User JSON


GET
/api/users
List all users
None
200: [User] JSON


Example Requests
Create a Product:
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

Response:
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

Update a Category:
curl -X PATCH http://127.0.0.1:8080/api/categories/550e8400-e29b-41d4-a716-446655440000 \
-H "Content-Type: application/json" \
-d '{"name": "Electronics"}'

Response:
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Electronics",
  "description": "Category for electronic products",
  "created_at": "2025-06-14T10:00:00Z",
  "updated_at": "2025-06-14T12:30:00Z"
}

Project Structure
small_eshop/
├── src/
│   ├── api/                    # Primary adapters (Actix-Web handlers)
│   │   └── produits.rs         # Category and product endpoints
│   ├── application/            # Use cases (services)
│   │   ├── category_service.rs # Category use cases
│   │   └── product_service.rs  # Product use cases
│   ├── domain/                 # Core business logic
│   │   ├── entities/           # Entities (Category, Product, User)
│   │   └── ports/              # Input/output ports (traits)
│   ├── infrastructure/         # Secondary adapters
│   │   └── repository/         # SQLx-based repositories
│   ├── models/                 # Data transfer objects (DTOs)
│   │   └── produits.rs         # Category/Product DTOs
│   └── main.rs                 # Application entry point
├── migrations/                 # SQLx database migrations
├── .env                        # Environment variables
├── Cargo.toml                  # Dependencies and project config
└── README.md                   # Project documentation

Setup
Prerequisites

Rust (stable, edition 2021)
PostgreSQL
Cargo
SQLx CLI (cargo install sqlx-cli)

Installation

Clone the Repository:
git clone https://github.com/your-username/small-eshop-backend.git
cd small-eshop-backend


Configure Environment:Create a .env file:
DATABASE_URL=postgres://username:password@localhost:5432/small_eshop


Set Up Database:Create the database and apply migrations:
cargo sqlx database create
cargo sqlx migrate run


Run the Application:
cargo run

The API will be available at http://127.0.0.1:8080.


Testing
Test endpoints using curl, Postman, or similar tools. For automated tests, add unit tests in src/application and integration tests in tests/.
Example Unit Test (for ProductService):
#[tokio::test]
async fn test_create_product() {
    let repo = MockProductRepository::new();
    let service = ProductService::new(Arc::new(repo));
    let input = CreateProduct { /* ... */ };
    let result = service.create_product(input).await;
    assert!(result.is_ok());
}

Future Improvements

Authentication: Implement JWT-based authentication for secure endpoints.
Pagination: Add pagination and filtering for large product listings.
API Documentation: Generate OpenAPI/Swagger documentation.
SQL Triggers: Automate updated_at timestamps with database triggers.
Monitoring: Integrate metrics (e.g., Prometheus) for performance tracking.

Contributing
Contributions are welcome! Please open an issue or submit a pull request for bug fixes, features, or improvements. Follow the hexagonal architecture for new features.
License
This project is licensed under the MIT License.
