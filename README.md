# Xeno Marketing Platform API Reference

This reference outlines all available endpoints provided by the Xeno Marketing Platform.

**Base URL**: `http://localhost:3000/api` (Development Environment)

## Authentication

OAuth2.0 handles all authentication flows.

# API Endpoints

## 1. Customer Management

### 1.1. Submit Customer Information

* **Endpoint**: `POST /user`
* **Purpose**: Inserts or updates customer records.
* **Content-Type**: `application/json`

```json
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "phone": "123-456-7890",
  "address": {
    "street": "123 Main St",
    "city": "Anytown",
    "state": "CA",
    "zipCode": "90210",
    "country": "USA"
  }
}
```

* **Fields**:

  * `name` (string): Required. Full name.
  * `email` (string): Required. Unique email.
  * `phone` (string): Required. Contact number.
  * `address` (object): Required. Complete address.

* **Success (201 Created)**:

```json
{
  "success": true,
  "message": "User data ingested successfully.",
  "data": {
    "id": "generated-uuid",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "phone": "123-456-7890",
    "createdAt": "iso-timestamp",
    "address": {
      "userId": "generated-uuid",
      "street": "123 Main St",
      "city": "Anytown",
      "state": "CA",
      "zipCode": "90210",
      "country": "USA"
    }
  }
}
```

* **Failure (400 Bad Request)**:

```json
{
  "success": false,
  "error": "Invalid user data provided.",
  "details": {
    "email": ["Invalid email format"]
  }
}
```

---

## 2. Order Processing

### 2.1. Submit Order

* **Endpoint**: `POST /order`
* **Purpose**: Adds a new order tied to an existing customer.
* **Content-Type**: `application/json`

```json
{
  "customerId": "customer-uuid",
  "items": [
    {
      "productId": "prod-123",
      "name": "Awesome T-Shirt",
      "price": 2500,
      "quantity": 2,
      "total": 5000
    }
  ],
  "totalAmount": 5000,
  "currency": "USD",
  "status": "delivered"
}
```

* **Fields**:

  * `customerId` (UUID): Required.
  * `items` (array): Required. Includes product line details.
  * `totalAmount` (int): Required. In cents.
  * `currency` (string): Required. Currency code.
  * `status` (string): Required. Current order status.

* **Success (201 Created)**:

```json
{
  "success": true,
  "message": "Order data ingested successfully.",
  "data": {
    "id": "generated-order-uuid",
    "customerId": "customer-uuid",
    "totalAmount": 5000,
    "currency": "USD",
    "status": "delivered",
    "createdAt": "iso-timestamp",
    "items": [
      {
        "id": "generated-item-uuid",
        "orderId": "generated-order-uuid",
        "productId": "prod-123",
        "name": "Awesome T-Shirt",
        "price": 2500,
        "quantity": 2,
        "total": 5000
      }
    ]
  }
}
```

* **Failure (404 Not Found)**:

```json
{
  "success": false,
  "error": "Customer with ID customer-uuid not found."
}
```

---

## 3. Audience Segments

### 3.1. Simulate Segment Audience

* **Endpoint**: `POST /segments/preview`
* **Purpose**: Estimates audience size using given rules, without saving.
* **Content-Type**: `application/json`

```json
{
  "rules": {
    "groups": [
      {
        "conditions": [
          {"field": "totalSpend", "operator": "greaterThan", "value": 10000},
          {"field": "state", "operator": "equals", "value": "CA"}
        ]
      },
      {
        "conditions": [
          {"field": "orderCount", "operator": "greaterThanOrEqual", "value": 5}
        ]
      }
    ]
  }
}
```

* **Success (200 OK)**:

```json
{
  "success": true,
  "data": {
    "audienceSize": 150,
    "sampleUserIds": ["user-uuid-1", "user-uuid-2", "..."]
  }
}
```

### 3.2. Save Segment

* **Endpoint**: `POST /segments`
* **Purpose**: Creates a new segment.

```json
{
  "name": "High Value CA Customers",
  "rules": {
    "groups": [
      {
        "conditions": [
          {"field": "totalSpend", "operator": "greaterThan", "value": 10000},
          {"field": "state", "operator": "equals", "value": "CA"}
        ]
      }
    ]
  }
}
```

* **Success (201 Created)**:

```json
{
  "success": true,
  "message": "Segment \"High Value CA Customers\" created successfully.",
  "data": {
    "id": "generated-segment-uuid",
    "name": "High Value CA Customers",
    "rules": { /* rules */ },
    "audienceUserIds": ["..."],
    "createdAt": "iso-timestamp",
    "updatedAt": "iso-timestamp"
  }
}
```

### 3.3. Retrieve All Segments

* **Endpoint**: `GET /segments`

* **Purpose**: Fetches every saved segment sorted by newest first.

* **Success (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "id": "segment-uuid-1",
      "name": "High Value CA Customers",
      "rules": { /* rules */ },
      "audienceUserIds": ["..."],
      "createdAt": "iso-timestamp",
      "updatedAt": "iso-timestamp"
    }
  ]
}
```

### 3.4. Fetch Segment Details

* **Endpoint**: `GET /segments/:segmentId`

* **Purpose**: Get full data for a single segment.

* **Success (200 OK)**:

```json
{
  "success": true,
  "data": {
    "id": "segment-uuid-1",
    "name": "High Value CA Customers",
    "rules": { /* rules */ },
    "audienceUserIds": ["..."],
    "createdAt": "iso-timestamp",
    "updatedAt": "iso-timestamp"
  }
}
```

---

## 4. Campaigns

### 4.1. Launch Campaign with Segment

* **Endpoint**: `POST /campaigns`
* **Purpose**: Generates a new segment and associates it with a campaign.

```json
{
  "campaignName": "Welcome New Users Q2",
  "message": "Hello {{name}}, welcome to our platform! Enjoy 10% off your first order.",
  "segmentName": "New Users - Last 14 Days",
  "segmentRules": {
    "groups": [
      {
        "conditions": [
          {"field": "userCreatedAt", "operator": "newerThanDays", "value": 14}
        ]
      }
    ]
  }
}
```

* **Success (201 Created)**:

```json
{
  "success": true,
  "message": "Segment \"New Users - Last 14 Days\" and Campaign \"Welcome New Users Q2\" created successfully. Campaign status: PROCESSING.",
  "data": {
    "segment": {
      "id": "generated-segment-uuid",
      "name": "New Users - Last 14 Days",
      "rules": { /* rules */ },
      "audienceUserIds": ["..."],
      "createdAt": "iso-timestamp",
      "updatedAt": "iso-timestamp"
    },
    "campaign": {
      "id": "generated-campaign-uuid",
      "name": "Welcome New Users Q2",
      "messageTemplate": "Hello {{name}}, welcome to our platform! Enjoy 10% off your first order.",
      "status": "PROCESSING",
      "audienceSize": 120,
      "sentCount": 0,
      "failedCount": 0,
      "createdAt": "iso-timestamp",
      "updatedAt": "iso-timestamp",
      "segmentId": "generated-segment-uuid"
    }
  }
}
```

### 4.2. List Campaigns

* **Endpoint**: `GET /campaigns`

* **Purpose**: Lists all campaigns with segment metadata.

* **Success (200 OK)**:

```json
{
  "success": true,
  "data": [
    {
      "id": "campaign-uuid-1",
      "name": "Welcome New Users Q2",
      "messageTemplate": "Hello {{name}}, ...",
      "status": "PROCESSING",
      "audienceSize": 120,
      "sentCount": 0,
      "failedCount": 0,
      "createdAt": "iso-timestamp",
      "updatedAt": "iso-timestamp",
      "segmentId": "segment-uuid-associated",
      "segmentName": "New Users - Last 14 Days",
      "segment": {
        "name": "New Users - Last 14 Days"
      }
    }
  ]
}
```

### 4.3. Get Campaign Info

* **Endpoint**: `GET /campaigns/:campaignId`

* **Purpose**: Retrieve details of a specific campaign and its segment.

* **Success (200 OK)**:

```json
{
  "success": true,
  "data": {
    "id": "campaign-uuid-1",
    "name": "Welcome New Users Q2",
    "messageTemplate": "Hello {{name}}, ...",
    "status": "PROCESSING",
    "audienceSize": 120,
    "sentCount": 0,
    "failedCount": 0,
    "createdAt": "iso-timestamp",
    "updatedAt": "iso-timestamp",
    "segmentId": "segment-uuid-associated",
    "segment": {
      "id": "segment-uuid-associated",
      "name": "New Users - Last 14 Days",
      "rules": { /* rules */ },
      "audienceUserIds": ["..."]
    }
  }
}
```
