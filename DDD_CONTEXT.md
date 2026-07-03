# Shopping Cart Domain-Driven Design (DDD) Context

This document compiles all Domain-Driven Design (DDD) concepts, Bounded Contexts, Ports, Adapters, and Architectural guidelines defined for the Shopping Cart interface project. It serves as the foundational reference when starting the project from scratch.

## 1. Architectural Principles
- **Architecture Base:** Hexagonal Architecture (Ports & Adapters).
- **Core Strategy:** Domain-Driven Design (DDD).
- **State Management:** Zustand (used as the primary state management library for UI/Application layer state and repositories).
- **Financial Representation:** All financial amounts are represented using a `Money` Value Object to avoid floating-point precision issues.
- **Folder Structure Pattern:**
  - `ui/`: Driving adapters (e.g., React components).
  - `application/`: Application Layer (Use Cases and Ports).
  - `domain/`: Pure domain core (Aggregates, Entities, Events, Value Objects) containing no external dependencies.
  - `infrastructure/`: Driven adapters (Repositories, Context Adapters).
- **Context Decoupling:** Contexts remain decoupled and communicate by using interfaces (Ports) or publishing/subscribing to asynchronous Domain Events via an Event Bus.

---

## 2. Bounded Contexts Overview

The domain is split into three primary Bounded Contexts:

### 🛍️ Cart Context
- **Responsibility**: Manages the user's active shopping cart, items, and calculating the raw subtotal.
- **Aggregate Root**: `Cart` (contains `CartItem` entities uniquely identified by `SkuId`).
- **Invariants**: Item quantity cannot be less than 1. Can contain multiple coupons. Subtotal dynamically calculated.

### 📦 Inventory Context
- **Responsibility**: Manages product variants and available stock levels.
- **Aggregate Root**: `ProductVariant`.
- **Invariants**: Cart item quantity cannot exceed available stock. Cannot have negative `TotalOnHand`.

### 🎟️ Pricing & Promotions Context
- **Responsibility**: Manages coupon codes, validation rules, and calculating order discounts.
- **Aggregate Root**: `Coupon`.
- **Invariants**: Validates whether a discount code exists, returning specific domain errors. Discount calculation cannot result in a negative total.

---

## 3. Context Mapping & Domain Events
Contexts communicate directly via Application Services (Queries/Commands) using Ports, or asynchronously via an Event Bus.

**Cross-Context Communication Examples:**
- **Synchronous / Ports:** Cart queries Inventory for stock availability via `IInventoryService`; Cart validates code against Pricing via `IPricingService`.
- **Asynchronous / Events:** Cart publishes `CheckoutInitiated`, which Inventory listens to in order to reserve stock asynchronously.

---

## 4. Deep-Dive: Cart Bounded Context

### Ubiquitous Language
- **Cart:** A container for items where we can add, remove, and change the quantity of things.
- **Cart Item:** A representation of the Inventory Item we are currently working with inside the Cart.
- **skuId:** The unique identifier representing a specific variant of an inventory item.

### Structural Elements
- **Aggregate Root:** `Cart`
- **Entities:** `CartItem` (Identifier: `skuId`)
- **Value Objects:** `Money`

### Lifecycle
- `Active` -> `Checkout_Pending` -> `Checked_Out`

### Ports (CQRS)
**Driving Adapters & Use Cases (Inbound):**
- *Commands:* `AddItemToCart`, `RemoveItemFromCart`, `ChangeCartItemQuantity`, `ApplyCouponToCart`, `RemoveCouponFromCart`, `InitiateCheckout`
- *Queries:* `GetCart`

**Driven Adapters & Ports (Outbound):**
- `IInventoryService`: Queries stock availability synchronously.
- `IPricingService`: Validates codes and calculates the actual order discount.
- `ICartRepository`: Repository interface (implemented by `ZustandCartRepository`).

### Domain Events
- `ItemAddedToCart`, `CartItemQuantityChanged`, `ItemRemovedFromCart`, `CouponApplied`, `CouponRemoved`, `CartCleared`, `CheckoutInitiated`, `CartCheckedOut`

---

## 5. Deep-Dive: Inventory Bounded Context

### Ubiquitous Language
- **ProductVariant:** The specific version of a product (e.g., Orange vs. Pink) being stocked.
- **Stock Validation / Availability:** Checking if we have enough of a variant on hand to fulfill a request.
- **Reservation:** Temporarily holding a specific quantity of stock for a particular order that has initiated checkout.
- **Depletion:** Permanently removing stock once an order is fully confirmed and payment is successful.

### Structural Elements
- **Aggregate Root:** `ProductVariant` (Identifier: `skuId`)
- **Value Objects:** `StockReservation` (Fields: `orderId`, `quantity`, `reservedAtTimestamp`)

### Lifecycle
- State is derived dynamically via Math (`Available` = `TotalOnHand` - `Reserved`).

### Ports (CQRS)
**Driving Adapters & Use Cases (Inbound):**
- *Commands:* `ReserveStock`, `ReleaseStockReservation`, `ConfirmStockDepletion`
- *Queries:* `CheckStockAvailability`, `GetProductVariant`

**Driven Adapters & Ports (Outbound):**
- `IStockRepository`: Loads/Saves inventory state (e.g., `MockInventoryRepository`).

### Domain Events
- `StockReserved`, `StockDepleted`

---

## 6. Deep-Dive: Pricing & Promotions Bounded Context

### Ubiquitous Language
- **Coupon:** A promotion code that grants a discount on the order total.
- **Validation:** The process of checking if a coupon code is existing and applicable.
- **Discount:** The amount subtracted from the subtotal based on coupon rules.

### Structural Elements
- **Aggregate Root:** `Coupon` (Identifier: `code`)
- **Value Objects:** `Money`, `DiscountAmount`

### Lifecycle
- `Active` -> `Expired` / `Deactivated`

### Ports (CQRS)
**Driving Adapters & Use Cases (Inbound):**
- *Commands:* `ValidateCoupon`
- *Queries:* `CalculateDiscount`, `GetCouponDetails`

**Driven Adapters & Ports (Outbound):**
- `ICouponRepository`: Loads coupon data (e.g., `MockCouponRepository`).

### Domain Events
- `CouponValidated`, `CouponValidationFailed`, `DiscountCalculated`
