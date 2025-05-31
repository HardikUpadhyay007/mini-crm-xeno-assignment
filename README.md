Xeno Marketing Platform – API Reference

Environment: Development
Base Endpoint: http://localhost:3000/api
 Authentication

All API interactions are secured using OAuth 2.0 protocol. Ensure proper token handling and renewal logic is in place before making requests.
 User Operations
 Add or Update Customer Info

    HTTP Method: POST

    Route: /user

    Purpose: Submit new customer information or update existing details in the database.

Payload (JSON format):

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

Field Descriptions:

    name (string): Customer's full name.

    email (string): Must be unique. Email address of the user.

    phone (string): Contact number.

    address (object): Must include street, city, state, zipCode, and country.

Success Response (201):

{
  "success": true,
  "message": "User data ingested successfully.",
  "data": {
    "id": "generated-uuid",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "phone": "123-456-7890",
    "createdAt": "timestamp",
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

Failure Response (400 - Validation Issue):

{
  "success": false,
  "error": "Invalid user data provided.",
  "details": {
    "email": ["Invalid email format"]
  }
}

 Order Operations
 Submit an Order

    HTTP Method: POST

    Route: /order

    Function: Records a purchase made by a customer. The customerId must already exist in the system.

Request Body Example:

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

Important Fields:

    items must contain valid productId, name, price (in cents), quantity, and calculated total.

    totalAmount is the sum of item totals.

    status supports values like processing, shipped, delivered.

Success (201 Created):

{
  "success": true,
  "message": "Order data ingested successfully.",
  "data": {
    "id": "generated-order-uuid",
    "customerId": "customer-uuid",
    "totalAmount": 5000,
    "currency": "USD",
    "status": "delivered",
    "createdAt": "timestamp",
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

Error (404 - Customer Not Found):

{
  "success": false,
  "error": "Customer with ID customer-uuid not found."
}

 Segmentation Engine
 Preview Audience by Rules

    Endpoint: POST /segments/preview

    Use Case: Simulates segmentation rules to estimate audience size without creating a segment.

Input Format:

{
  "rules": {
    "groups": [
      {
        "conditions": [
          { "field": "totalSpend", "operator": "greaterThan", "value": 10000 },
          { "field": "state", "operator": "equals", "value": "CA" }
        ]
      },
      {
        "conditions": [
          { "field": "orderCount", "operator": "greaterThanOrEqual", "value": 5 }
        ]
      }
    ]
  }
}

Success Output (200 OK):

{
  "success": true,
  "data": {
    "audienceSize": 150,
    "sampleUserIds": ["user-uuid-1", "user-uuid-2"]
  }
}

 Save New Segment

    Route: POST /segments

    Purpose: Persists a new segment based on provided conditions.

Example Request:

{
  "name": "High Value CA Customers",
  "rules": {
    "groups": [
      {
        "conditions": [
          { "field": "totalSpend", "operator": "greaterThan", "value": 10000 },
          { "field": "state", "operator": "equals", "value": "CA" }
        ]
      }
    ]
  }
}

On Success (201):

{
  "success": true,
  "message": "Segment \"High Value CA Customers\" created successfully.",
  "data": {
    "id": "segment-uuid",
    "name": "High Value CA Customers",
    "rules": { /* same as input */ },
    "audienceUserIds": ["..."],
    "createdAt": "timestamp",
    "updatedAt": "timestamp"
  }
}

 List Segments

    Method: GET

    Endpoint: /segments

    Returns: All stored segments sorted by newest first.

Response Example:

{
  "success": true,
  "data": [
    {
      "id": "segment-uuid-1",
      "name": "High Value CA Customers",
      "rules": { /* rule config */ },
      "audienceUserIds": ["..."],
      "createdAt": "timestamp",
      "updatedAt": "timestamp"
    }
  ]
}

 View Single Segment

    Endpoint: GET /segments/:segmentId

    Input Param: segmentId (UUID)

    Use: Retrieve full information of a specific segment.

 Campaign Operations
 Launch Campaign with Segment

    Endpoint: POST /campaigns

    Goal: Create a new segment and immediately initiate a campaign for that segment.

Sample Payload:

{
  "campaignName": "Welcome New Users Q2",
  "message": "Hello {{name}}, welcome to our platform! Enjoy 10% off your first order.",
  "segmentName": "New Users - Last 14 Days",
  "segmentRules": {
    "groups": [
      {
        "conditions": [
          { "field": "userCreatedAt", "operator": "newerThanDays", "value": 14 }
        ]
      }
    ]
  }
}

Expected Result:

{
  "success": true,
  "message": "Segment and Campaign created. Campaign status: PROCESSING.",
  "data": {
    "segment": { /* Segment info */ },
    "campaign": {
      "id": "campaign-uuid",
      "name": "Welcome New Users Q2",
      "messageTemplate": "Hello {{name}}, welcome to our platform! Enjoy 10% off your first order.",
      "status": "PROCESSING",
      "audienceSize": 120,
      "sentCount": 0,
      "failedCount": 0,
      "createdAt": "timestamp",
      "updatedAt": "timestamp",
      "segmentId": "segment-uuid"
    }
  }
}

 Campaign Listing

    Route: GET /campaigns

    Output: Full list of campaigns, including associated segment metadata.

 Campaign Details by ID

    Endpoint: GET /campaigns/:campaignId

    Purpose: Provides detailed data for a given campaign including the linked segment.
