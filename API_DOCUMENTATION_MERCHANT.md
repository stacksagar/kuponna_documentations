# Merchant API Documentation

## Business Profile

### GET `/api/merchant/business-profile`
- **Description:** Get the current merchant's business profile
- **Authentication:** Required (JWT token, merchant only)
- **Returns:** Merchant profile object for the authenticated merchant

### PATCH `/api/merchant/business-profile`
- **Description:** Update merchant's business profile (partial update)
- **Authentication:** Required (JWT token, merchant only)
- **Request Body:**
  ```json
  {
    "firstName": "string", // OPTIONAL
    "lastName": "string", // OPTIONAL
    "email": "string", // OPTIONAL
    "phoneNumber": "string", // OPTIONAL
    "category": "string", // OPTIONAL
    "businessName": "string", // OPTIONAL
    "businessAddress": "string", // OPTIONAL
    "businessEmail": "string", // OPTIONAL
    "businessPhone": "string", // OPTIONAL
    "businessCategory": "string", // OPTIONAL
    "fileUrl": "string" // OPTIONAL (business document or image URL)
  }
  ```
- **Note:** Cannot update `id`, `userId`, `status`, `createdAt`, `updatedAt`
- **Returns:** Updated merchant profile object

---

## Deals Management

### GET `/api/merchant/deals`
- **Description:** List all deals created by the merchant
- **Authentication:** Required (JWT token, merchant only)
- **Query Parameters:**
  - `id`: (optional) filter by deal ID
  - `status`: (optional) filter by deal status (`draft`, `pending`, `active`, `completed`, `cancelled`)
  - `type`: (optional) filter by deal type (`physical`, `digital`)
- **Returns:**
  ```json
  {
    "items": [Deal]
  }
  ```
  Where each `Deal` includes all deal fields and related groups/members.

### POST `/api/merchant/deals`
- **Description:** Create a new deal
- **Authentication:** Required (JWT token, merchant only)
- **Request Body:**
  ```json
  {
    "title": "string", // REQUIRED
    "pricePerPerson": "number", // OPTIONAL
    "discountPrice": "number", // OPTIONAL
    "deliveryFee": "number", // OPTIONAL
    "discountPercentage": "number", // OPTIONAL
    "requiredMembers": "number", // OPTIONAL
    "description": "string", // OPTIONAL
    "redemptionExpired": "string (date)", // OPTIONAL
    "endDate": "string (date)", // OPTIONAL
    "images": ["string"], // OPTIONAL (array of image URLs)
    "videoUrl": "string", // OPTIONAL
    "category": "string", // OPTIONAL
    "tags": ["string"], // OPTIONAL
    "status": "draft|pending", // OPTIONAL (default: pending)
    "location": "string", // OPTIONAL
    "type": "physical|digital" // REQUIRED
  }
  ```
- **Returns:** Created deal object

### PATCH `/api/merchant/deals`
- **Description:** Update an existing deal (only if status is `draft` or `pending`)
- **Authentication:** Required (JWT token, merchant only)
- **Request Body:**
  ```json
  {
    "id": "string", // REQUIRED (deal ID)
    "title": "string", // OPTIONAL
    "description": "string", // OPTIONAL
    "pricePerPerson": "number", // OPTIONAL
    "requiredMembers": "number", // OPTIONAL
    "deliveryFee": "number", // OPTIONAL
    "discountPercentage": "number", // OPTIONAL
    "redemptionExpired": "string (date)", // OPTIONAL
    "endDate": "string (date)" // OPTIONAL
  }
  ```
- **Returns:** Updated deal object
- **Note:** Cannot update if status is `active` or `completed`

### DELETE `/api/merchant/deals`
- **Description:** Delete a deal by ID
- **Authentication:** Required (JWT token, merchant only)
- **Request Body:**
  ```json
  {
    "id": "string" // REQUIRED (deal ID)
  }
  ```
- **Returns:** `{ success: true }` on success

---

## Order Management

### PATCH `/api/merchant/order`
- **Description:** Update order status (e.g., mark as shipped, delivered, etc.)
- **Authentication:** Required (JWT token, merchant only)
- **Request Body:**
  ```json
  {
    "orderId": "string", // REQUIRED
    "status": "pending|paid|shipped|delivered|cancelled|rejected|refund-requested|refunded|fulfilled" // REQUIRED
  }
  ```
- **Returns:** Updated order object

### PUT `/api/merchant/order`
- **Description:** Update order details (e.g., delivery info, files, etc.)
- **Authentication:** Required (JWT token, merchant only)
- **Request Body:**
  ```json
  {
    "orderId": "string", // REQUIRED
    "payableData": { /* key-value pairs to update, e.g. deliveryAddress, files, status, etc. */ }
  }
  ```
- **Returns:** Updated order object
- **Note:** If status is set to `delivered`, a delivery email is sent to the user automatically.

---

## Notes
- All endpoints require merchant authentication (JWT token)
- All IDs are UUID strings
- All date fields are ISO 8601 strings
- All responses are JSON
- For full model field details, see the main API documentation

