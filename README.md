# Körber InventoryService

## 📦 Service Overview
The Inventory Service is responsible for managing product stock levels across multiple batches. It ensures that inventory is handled according to the FEFO (First Expired, First Out) principle, which is critical for perishable goods or products with limited shelf lives.

# 🚀 Key Features

* **Batch Management**: Products are stored in batches, each with its own batch_id, quantity, and expiry_date.

* **Factory Design Pattern**: Implements a strategy-based approach for updating stock. This allows the system to switch between FEFO or other future strategies ( like LIFO) without changing the core service code.

* **Automated Data Seeding**: Uses Liquibase to read inventory.csv and populate the H2 database on startup.

* **RESTful API**: Provides endpoints to view sorted inventory and process stock deductions.


## 🛠 Tech Stack
Java 17

Spring Boot 3.x

Spring Data JPA (Persistence)

H2 Database (In-memory)

Liquibase (Database Migrations)

Lombok (Boilerplate reduction)

## 📖 API Reference
Get Sorted Inventory
Returns all batches for a specific product, sorted by the closest expiry date first.

* **URL**: /inventory/{productId}
* **Method**: `GET`
* **Success Response**:

#### JSON
```json
{
  "productId": 1005,
  "productName": "Smartwatch",
  "batches": [
    { "batchId": 5, "quantity": 39, "expiryDate": "2026-03-31" },
    { "batchId": 7, "quantity": 40, "expiryDate": "2026-04-24" }
  ]
}
```

Update Inventory (Internal)
Deducts stock based on the FEFO strategy. This is typically called by the Order Service.

* **URL**: /inventory/update
* **Method**: `POST`
* **Params**: `productId` (Long), `quantity` (Integer)

* **Returns**: `List<Long>` (The IDs of the batches affected by this deduction).

## 🏗 Design Patterns: The Factory Pattern
The service uses a `StockUpdateFactory` to determine how stock should be deducted.

Strategy Interface: `StockUpdateStrategy`

Concrete Strategy: `FEFOUpdateStrategy` (Deducts from the earliest expiring batches first).

Factory: StockUpdateFactory (Returns the correct strategy bean).

## 🧪 Testing
The service includes unit tests to ensure the FEFO logic works correctly even when an order spans multiple batches.

Run tests via Maven:

```Bash

mvn test
```
**Key Test Case**: `shouldDeductFromMultipleBatchesWhenFirstIsInsufficient()`

* Tests that if Batch A has 10 units and Batch B has 10 units, an order of 15 units correctly leaves Batch A with 0 and Batch B with 5.

## 🗄 Database Configuration
* **Console**: http://localhost:8081/h2-console

* **JDBC URL**: `jdbc:h2:mem:inventorydb`

* **Migration Path**: `src/main/resources/db/changelog/master.xml`

* **CSV Data Path**: `src/main/resources/db/changelog/data/inventory.csv`
