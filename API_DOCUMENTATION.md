# Kuponna API Documentation

## Table of Contents

- [Frontend Usage Guide](#frontend-usage-guide)
- [Deals API](#deals-api)
- [Deal Participants API](#deal-participants-api)
- [Deal Images API](#deal-images-api)
- [Orders API](#orders-api)
- [Redemption Cards API](#redemption-cards-api)
- [Refund Requests API](#refund-requests-api)
- [Transactions API](#transactions-api)
- [Messages API](#messages-api)
- [Notifications API](#notifications-api)
- [Delivery Tracking API](#delivery-tracking-api)
- [Merchant Profile API](#merchant-profile-api)
- [Complaints API](#complaints-api)
- [Vouchers API](#vouchers-api)
- [Groups API](#groups-api)
- [Error Responses](#error-responses)
- [Best Practices](#best-practices)

---

## Frontend Usage Guide

**All examples use [axios](https://github.com/axios/axios). Install with:**

```bash
npm install axios
```

See [KUPONNA_AUTHENTICATION_DOCUMENTATION.md] for authentication flows (login, registration, etc.).

- All requests (except public GETs) require authentication (JWT cookie/session).
- Use plural endpoints for lists: `/api/deals`, `/api/orders`, etc.
- Use singular endpoints for single items: `/api/deals?dealId=...`, `/api/orders?orderId=...`, etc.
- Use query params to filter: `/api/orders?userId=...&dealId=...`
- Use the correct HTTP method: GET (read), POST (create), PUT (update), DELETE (remove).
- All responses are JSON.

**See each API section below for detailed request/response schemas and code examples.**

---

# Deals API

## Overview

- **Purpose:** Manage group deals (create, update, list, delete)
- **Who can use:**
  - `GET`: Anyone (public)
  - `POST`, `PUT`, `DELETE`: Merchants (authenticated)

## Endpoints

- `GET /api/deals` — List all deals (with optional filters)
- `GET /api/deals?dealId=...` — Get a single deal by ID
- `POST /api/deals` — Create a new deal (merchant only)
- `PUT /api/deals?dealId=...` — Update a deal (merchant only, own deals)
- `DELETE /api/deals?dealId=...` — Delete a deal (merchant only, own deals)

## Query Parameters

| Name       | Type   | Required | Description             |
| ---------- | ------ | -------- | ----------------------- |
| dealId     | string | No       | Get a single deal by ID |
| merchantId | string | No       | Filter by merchant      |
| status     | string | No       | Filter by deal status   |
| type       | string | No       | Filter by deal type     |

## Request Body (POST/PUT)

| Field           | Type   | Required | Validation/Description      |
| --------------- | ------ | -------- | --------------------------- |
| title           | string | Yes      | Min 2 chars                 |
| description     | string | Yes      | Min 10 chars                |
| type            | string | Yes      | Enum: 'physical', 'digital' |
| originalPrice   | number | Yes      | > 0                         |
| discountedPrice | number | Yes      | > 0, < originalPrice        |
| minGroupSize    | number | Yes      | >= 2                        |
| maxGroupSize    | number | No       | >= minGroupSize             |
| startDate       | string | Yes      | ISO 8601 date               |
| endDate         | string | Yes      | ISO 8601 date               |
| images          | array  | No       | Array of image URLs         |
| location        | string | No       |                             |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "merchantId": "string",
      "title": "string",
      "description": "string",
      "type": "physical|digital",
      "originalPrice": 100,
      "discountedPrice": 80,
      "minGroupSize": 5,
      "maxGroupSize": 10,
      "currentGroupSize": 2,
      "startDate": "2024-06-01T00:00:00Z",
      "endDate": "2024-06-30T23:59:59Z",
      "status": "draft|pending_approval|active|completed|cancelled",
      "images": ["url1", "url2"],
      "location": "string",
      "createdAt": "2024-05-01T12:00:00Z",
      "updatedAt": "2024-05-01T12:00:00Z"
    }
  ]
}
```

## Response (GET by ID)

```json
{
  "item": {
    "id": "string",
    "merchantId": "string",
    ... // same as above
  }
}
```

## Example: Fetching All Deals

```js
import axios from "axios";

async function getAllDeals() {
  const res = await axios.get("/api/deals", { withCredentials: true });
  console.log(res.data.items); // Array of deals
}
```

## Example: Creating a Deal

```js
import axios from "axios";

async function createDeal() {
  const res = await axios.post(
    "/api/deals",
    {
      title: "New Deal",
      description: "Great offer",
      type: "physical",
      originalPrice: 100,
      discountedPrice: 80,
      minGroupSize: 5,
      startDate: "2024-06-01T00:00:00Z",
      endDate: "2024-06-30T23:59:59Z",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Deal Participants API

## Overview

- **Purpose:** Join, leave, and manage deal participation
- **Who can use:**
  - `GET`: Authenticated users (filter by dealId, userId, etc.)
  - `POST`, `PUT`, `DELETE`: Users (authenticated)

## Endpoints

- `GET /api/deal-participants` — List participants (with filters)
- `GET /api/deal-participants?dealParticipantId=...` — Get a single participant
- `POST /api/deal-participants` — Join a deal
- `PUT /api/deal-participants?dealParticipantId=...` — Update participant status
- `DELETE /api/deal-participants?dealParticipantId=...` — Leave a deal

## Query Parameters

| Name              | Type   | Required | Description              |
| ----------------- | ------ | -------- | ------------------------ |
| dealParticipantId | string | No       | Get a single participant |
| dealId            | string | No       | Filter by deal           |
| userId            | string | No       | Filter by user           |
| status            | string | No       | Filter by status         |

## Request Body (POST)

| Field         | Type   | Required | Description   |
| ------------- | ------ | -------- | ------------- |
| dealId        | string | Yes      | Deal to join  |
| paymentAmount | number | Yes      | Amount to pay |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "userId": "string",
      "dealId": "string",
      "status": "pending|paid|completed|cancelled|refunded",
      "paymentId": "string|null",
      "paymentAmount": 80,
      "joinedAt": "2024-06-01T12:00:00Z",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Joining a Deal

```js
import axios from "axios";

async function joinDeal() {
  const res = await axios.post(
    "/api/deal-participants",
    { dealId: "123", paymentAmount: 80 },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Deal Images API

## Overview

- **Purpose:** Manage images for deals
- **Who can use:**
  - `GET`: Authenticated users (filter by dealId, etc.)
  - `POST`, `DELETE`: Merchants (authenticated, own deals)

## Endpoints

- `GET /api/deal-images` — List images (with filters)
- `GET /api/deal-images?dealImageId=...` — Get a single image
- `POST /api/deal-images` — Add image to a deal
- `DELETE /api/deal-images?dealImageId=...` — Remove image

## Query Parameters

| Name        | Type   | Required | Description        |
| ----------- | ------ | -------- | ------------------ |
| dealImageId | string | No       | Get a single image |
| dealId      | string | No       | Filter by deal     |

## Request Body (POST)

| Field  | Type   | Required | Description          |
| ------ | ------ | -------- | -------------------- |
| dealId | string | Yes      | Deal to add image to |
| url    | string | Yes      | Image URL            |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "dealId": "string",
      "url": "string",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Adding an Image

```js
import axios from "axios";

async function addDealImage() {
  const res = await axios.post(
    "/api/deal-images",
    { dealId: "123", url: "https://example.com/image.jpg" },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Orders API

## Overview

- **Purpose:** Manage orders for deals
- **Who can use:**
  - `GET`: Authenticated users/merchants (own orders)
  - `POST`: Users (authenticated)
  - `PUT`, `DELETE`: Users (own orders), merchants (own deals)

## Endpoints

- `GET /api/orders` — List orders (with filters)
- `GET /api/orders?orderId=...` — Get a single order
- `POST /api/orders` — Create order (user only)
- `PUT /api/orders?orderId=...` — Update order (user/merchant, own)
- `DELETE /api/orders?orderId=...` — Cancel order (user only, own)

## Query Parameters

| Name    | Type   | Required | Description        |
| ------- | ------ | -------- | ------------------ |
| orderId | string | No       | Get a single order |
| userId  | string | No       | Filter by user     |
| dealId  | string | No       | Filter by deal     |

## Request Body (POST)

| Field         | Type   | Required | Description             |
| ------------- | ------ | -------- | ----------------------- |
| dealId        | string | Yes      | Deal to order           |
| amount        | number | Yes      | Order amount            |
| type          | string | Yes      | 'physical' or 'digital' |
| paymentMethod | string | No       | e.g., 'card', 'wallet'  |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "dealId": "string",
      "userId": "string",
      "status": "pending|paid|shipped|delivered|cancelled|refunded",
      "type": "physical|digital",
      "amount": 80,
      "paymentMethod": "string|null",
      "paidAt": "2024-06-01T12:00:00Z|null",
      "shippedAt": "2024-06-02T12:00:00Z|null",
      "deliveredAt": "2024-06-03T12:00:00Z|null",
      "cancelledAt": "2024-06-04T12:00:00Z|null",
      "refundedAt": "2024-06-05T12:00:00Z|null",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating an Order

```js
import axios from "axios";

async function createOrder() {
  const res = await axios.post(
    "/api/orders",
    { dealId: "123", amount: 80, type: "physical" },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Redemption Cards API

## Overview

- **Purpose:** Manage redemption cards for orders (digital/physical delivery)
- **Who can use:**
  - `GET`: Users (own), merchants (own), admins (all)
  - `POST`, `PUT`, `DELETE`: Merchants (own), admins (all)

## Endpoints

- `GET /api/redemption-cards` — List redemption cards (with filters)
- `GET /api/redemption-cards?redemptionCardId=...` — Get a single redemption card
- `POST /api/redemption-cards` — Create redemption card (merchant/admin only)
- `PUT /api/redemption-cards?redemptionCardId=...` — Update redemption card (merchant/admin only)
- `DELETE /api/redemption-cards?redemptionCardId=...` — Delete redemption card (merchant/admin only)

## Query Parameters

| Name             | Type   | Required | Description                  |
| ---------------- | ------ | -------- | ---------------------------- |
| redemptionCardId | string | No       | Get a single redemption card |
| orderId          | string | No       | Filter by order              |
| userId           | string | No       | Filter by user               |
| status           | string | No       | Filter by status             |

## Request Body (POST)

| Field              | Type   | Required | Description                                                     |
| ------------------ | ------ | -------- | --------------------------------------------------------------- |
| orderId            | string | Yes      | Order for the card                                              |
| userId             | string | Yes      | User receiving the card                                         |
| code               | string | Yes      | Unique redemption code                                          |
| status             | string | No       | Enum: 'pending', 'shipped', 'delivered', 'activated', 'expired' |
| deliveryTrackingId | string | No       | Delivery tracking reference                                     |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "orderId": "string",
      "userId": "string",
      "code": "string",
      "status": "pending|shipped|delivered|activated|expired",
      "deliveryTrackingId": "string|null",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Redemption Card

```js
import axios from "axios";

async function createRedemptionCard() {
  const res = await axios.post(
    "/api/redemption-cards",
    {
      orderId: "123",
      userId: "456",
      code: "ABCDEF123",
      status: "pending",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Refund Requests API

## Overview

- **Purpose:** Manage refund requests for orders
- **Who can use:**
  - `GET`: Users (own), merchants (own), admins (all)
  - `POST`: Users (own)
  - `PUT`: Merchants (own), admins (all)
  - `DELETE`: Admins (all)

## Endpoints

- `GET /api/refund-requests` — List refund requests (with filters)
- `GET /api/refund-requests?refundRequestId=...` — Get a single refund request
- `POST /api/refund-requests` — Create refund request (user only)
- `PUT /api/refund-requests?refundRequestId=...` — Update refund request (merchant/admin only)
- `DELETE /api/refund-requests?refundRequestId=...` — Delete refund request (admin only)

## Query Parameters

| Name            | Type   | Required | Description                 |
| --------------- | ------ | -------- | --------------------------- |
| refundRequestId | string | No       | Get a single refund request |
| orderId         | string | No       | Filter by order             |
| userId          | string | No       | Filter by user              |
| status          | string | No       | Filter by status            |

## Request Body (POST)

| Field   | Type   | Required | Description       |
| ------- | ------ | -------- | ----------------- |
| orderId | string | Yes      | Order to refund   |
| reason  | string | Yes      | Reason for refund |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "userId": "string",
      "orderId": "string",
      "reason": "string",
      "status": "pending|approved|rejected",
      "adminId": "string|null",
      "processedAt": "2024-06-01T12:00:00Z|null",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Refund Request

```js
import axios from "axios";

async function createRefundRequest() {
  const res = await axios.post(
    "/api/refund-requests",
    {
      orderId: "123",
      reason: "Product was damaged",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Transactions API

## Overview

- **Purpose:** Manage payment, payout, and refund transactions
- **Who can use:**
  - `GET`: Users (own), merchants (own), admins (all)
  - `POST`, `PUT`, `DELETE`: Admins (all)

## Endpoints

- `GET /api/transactions` — List transactions (with filters)
- `GET /api/transactions?transactionId=...` — Get a single transaction
- `POST /api/transactions` — Create transaction (admin only)
- `PUT /api/transactions?transactionId=...` — Update transaction (admin only)
- `DELETE /api/transactions?transactionId=...` — Delete transaction (admin only)

## Query Parameters

| Name          | Type   | Required | Description              |
| ------------- | ------ | -------- | ------------------------ |
| transactionId | string | No       | Get a single transaction |
| userId        | string | No       | Filter by user           |
| orderId       | string | No       | Filter by order          |
| type          | string | No       | Filter by type           |
| status        | string | No       | Filter by status         |

## Request Body (POST)

| Field           | Type   | Required | Description                            |
| --------------- | ------ | -------- | -------------------------------------- |
| userId          | string | Yes      | User for the transaction               |
| orderId         | string | No       | Order for the transaction              |
| amount          | number | Yes      | Transaction amount                     |
| type            | string | Yes      | Enum: 'payment', 'payout', 'refund'    |
| status          | string | No       | Enum: 'pending', 'completed', 'failed' |
| paymentMethod   | string | No       | e.g., 'card', 'wallet'                 |
| transactionDate | string | No       | ISO 8601 date                          |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "userId": "string",
      "orderId": "string|null",
      "amount": 80,
      "type": "payment|payout|refund",
      "status": "pending|completed|failed",
      "paymentMethod": "string|null",
      "transactionDate": "2024-06-01T12:00:00Z",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Transaction (Admin Only)

```js
import axios from "axios";

async function createTransaction() {
  const res = await axios.post(
    "/api/transactions",
    {
      userId: "123",
      orderId: "456",
      amount: 80,
      type: "payment",
      status: "pending",
      paymentMethod: "card",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Messages API

## Overview

- **Purpose:** Send and manage messages between users, merchants, and admins
- **Who can use:**
  - `GET`: Users/merchants (own), admins (all)
  - `POST`: Users/merchants (own)
  - `PUT`, `DELETE`: Sender only

## Endpoints

- `GET /api/messages` — List messages (with filters)
- `GET /api/messages?messageId=...` — Get a single message
- `POST /api/messages` — Send a message
- `PUT /api/messages?messageId=...` — Update a message (sender only)
- `DELETE /api/messages?messageId=...` — Delete a message (sender only)

## Query Parameters

| Name       | Type   | Required | Description          |
| ---------- | ------ | -------- | -------------------- |
| messageId  | string | No       | Get a single message |
| senderId   | string | No       | Filter by sender     |
| receiverId | string | No       | Filter by receiver   |
| orderId    | string | No       | Filter by order      |
| dealId     | string | No       | Filter by deal       |
| type       | string | No       | Filter by type       |
| status     | string | No       | Filter by status     |

## Request Body (POST)

| Field      | Type   | Required | Description                                           |
| ---------- | ------ | -------- | ----------------------------------------------------- |
| receiverId | string | Yes      | Receiver of the message                               |
| orderId    | string | No       | Related order                                         |
| dealId     | string | No       | Related deal                                          |
| content    | string | Yes      | Message content                                       |
| type       | string | Yes      | Enum: 'user-merchant', 'user-admin', 'merchant-admin' |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "senderId": "string",
      "receiverId": "string",
      "orderId": "string|null",
      "dealId": "string|null",
      "content": "string",
      "type": "user-merchant|user-admin|merchant-admin",
      "status": "sent|read",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Sending a Message

```js
import axios from "axios";

async function sendMessage() {
  const res = await axios.post(
    "/api/messages",
    {
      receiverId: "456",
      content: "Hello!",
      type: "user-merchant",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Notifications API

## Overview

- **Purpose:** Manage notifications for users and merchants
- **Who can use:**
  - `GET`: Users/merchants (own), admins (all)
  - `POST`: Admins (all)
  - `PUT`, `DELETE`: Recipient only

## Endpoints

- `GET /api/notifications` — List notifications (with filters)
- `GET /api/notifications?notificationId=...` — Get a single notification
- `POST /api/notifications` — Create notification (admin only)
- `PUT /api/notifications?notificationId=...` — Update notification (recipient only)
- `DELETE /api/notifications?notificationId=...` — Delete notification (recipient only)

## Query Parameters

| Name           | Type   | Required | Description               |
| -------------- | ------ | -------- | ------------------------- |
| notificationId | string | No       | Get a single notification |
| userId         | string | No       | Filter by user            |
| type           | string | No       | Filter by type            |
| read           | bool   | No       | Filter by read/unread     |

## Request Body (POST)

| Field   | Type   | Required | Description                                            |
| ------- | ------ | -------- | ------------------------------------------------------ |
| userId  | string | Yes      | Recipient of the notification                          |
| type    | string | Yes      | Enum: 'system', 'deal', 'order', 'refund', 'complaint' |
| title   | string | Yes      | Notification title                                     |
| message | string | Yes      | Notification message                                   |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "userId": "string",
      "type": "system|deal|order|refund|complaint",
      "title": "string",
      "message": "string",
      "read": false,
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Notification (Admin Only)

```js
import axios from "axios";

async function createNotification() {
  const res = await axios.post(
    "/api/notifications",
    {
      userId: "123",
      type: "order",
      title: "Order Update",
      message: "Your order has shipped.",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Delivery Tracking API

## Overview

- **Purpose:** Track delivery status for orders
- **Who can use:**
  - `GET`: Users (own), merchants (own), admins (all)
  - `POST`, `PUT`: Merchants (own), admins (all)
  - `DELETE`: Admins (all)

## Endpoints

- `GET /api/delivery-tracking` — List delivery tracking records (with filters)
- `GET /api/delivery-tracking?deliveryTrackingId=...` — Get a single delivery tracking record
- `POST /api/delivery-tracking` — Create delivery tracking (merchant/admin only)
- `PUT /api/delivery-tracking?deliveryTrackingId=...` — Update delivery tracking (merchant/admin only)
- `DELETE /api/delivery-tracking?deliveryTrackingId=...` — Delete delivery tracking (admin only)

## Query Parameters

| Name               | Type   | Required | Description                  |
| ------------------ | ------ | -------- | ---------------------------- |
| deliveryTrackingId | string | No       | Get a single tracking record |
| orderId            | string | No       | Filter by order              |
| status             | string | No       | Filter by status             |

## Request Body (POST)

| Field          | Type   | Required | Description                                                     |
| -------------- | ------ | -------- | --------------------------------------------------------------- |
| orderId        | string | Yes      | Order to track                                                  |
| trackingNumber | string | Yes      | Tracking number                                                 |
| carrier        | string | No       | Carrier name                                                    |
| status         | string | No       | Enum: 'pending', 'shipped', 'in_transit', 'delivered', 'failed' |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "orderId": "string",
      "trackingNumber": "string",
      "carrier": "string|null",
      "status": "pending|shipped|in_transit|delivered|failed",
      "updatedAt": "2024-06-01T12:00:00Z",
      "createdAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Delivery Tracking Record

```js
import axios from "axios";

async function createDeliveryTracking() {
  const res = await axios.post(
    "/api/delivery-tracking",
    {
      orderId: "123",
      trackingNumber: "TRACK123",
      carrier: "DHL",
      status: "shipped",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Merchant Profile API

## Overview

- **Purpose:** Manage merchant business profiles
- **Who can use:**
  - `GET`: Merchants (own), admins (all)
  - `POST`: Merchants (own)
  - `PUT`: Merchants (own), admins (all)
  - `DELETE`: Admins (all)

## Endpoints

- `GET /api/merchant-profile` — List merchant profiles (with filters)
- `GET /api/merchant-profile?merchantProfileId=...` — Get a single merchant profile
- `POST /api/merchant-profile` — Create merchant profile (merchant only)
- `PUT /api/merchant-profile?merchantProfileId=...` — Update merchant profile (merchant/admin only)
- `DELETE /api/merchant-profile?merchantProfileId=...` — Delete merchant profile (admin only)

## Query Parameters

| Name              | Type   | Required | Description          |
| ----------------- | ------ | -------- | -------------------- |
| merchantProfileId | string | No       | Get a single profile |
| userId            | string | No       | Filter by user       |
| status            | string | No       | Filter by status     |

## Request Body (POST)

| Field              | Type   | Required | Description               |
| ------------------ | ------ | -------- | ------------------------- |
| businessName       | string | Yes      | Merchant business name    |
| businessAddress    | string | Yes      | Merchant business address |
| businessPhone      | string | Yes      | Merchant business phone   |
| businessEmail      | string | Yes      | Merchant business email   |
| registrationNumber | string | No       | Registration number       |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "userId": "string",
      "businessName": "string",
      "businessAddress": "string",
      "businessPhone": "string",
      "businessEmail": "string",
      "registrationNumber": "string|null",
      "status": "pending|approved|rejected",
      "verifiedAt": "2024-06-01T12:00:00Z|null",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Merchant Profile

```js
import axios from "axios";

async function createMerchantProfile() {
  const res = await axios.post(
    "/api/merchant-profile",
    {
      businessName: "My Store",
      businessAddress: "123 Main St",
      businessPhone: "555-1234",
      businessEmail: "store@example.com",
      registrationNumber: "RC123456",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Complaints API

## Overview

- **Purpose:** Manage user complaints and refund requests
- **Who can use:**
  - `GET`: Users (own), merchants (own), admins (all)
  - `POST`: Users (own)
  - `PUT`: Merchants (own), admins (all)
  - `DELETE`: Admins (all)

## Endpoints

- `GET /api/complaints` — List complaints (with filters)
- `GET /api/complaints?complaintId=...` — Get a single complaint
- `POST /api/complaints` — Create complaint (user only)
- `PUT /api/complaints?complaintId=...` — Update complaint (merchant/admin only)
- `DELETE /api/complaints?complaintId=...` — Delete complaint (admin only)

## Query Parameters

| Name        | Type   | Required | Description            |
| ----------- | ------ | -------- | ---------------------- |
| complaintId | string | No       | Get a single complaint |
| userId      | string | No       | Filter by user         |
| dealId      | string | No       | Filter by deal         |
| orderId     | string | No       | Filter by order        |
| status      | string | No       | Filter by status       |

## Request Body (POST)

| Field       | Type   | Required | Description                 |
| ----------- | ------ | -------- | --------------------------- |
| dealId      | string | No       | Related deal                |
| orderId     | string | No       | Related order               |
| type        | string | Yes      | Enum: 'complaint', 'refund' |
| subject     | string | Yes      | Complaint subject           |
| description | string | Yes      | Complaint description       |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "userId": "string",
      "dealId": "string|null",
      "orderId": "string|null",
      "type": "complaint|refund",
      "subject": "string",
      "description": "string",
      "status": "pending|under_review|resolved|rejected",
      "adminId": "string|null",
      "resolution": "string|null",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Complaint

```js
import axios from "axios";

async function createComplaint() {
  const res = await axios.post(
    "/api/complaints",
    {
      type: "complaint",
      subject: "Product not delivered",
      description: "I have not received my product.",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Vouchers API

## Overview

- **Purpose:** Manage digital/physical vouchers for orders
- **Who can use:**
  - `GET`: Users (own), merchants (own), admins (all)
  - `POST`, `PUT`: Merchants (own), admins (all)
  - `DELETE`: Admins (all)

## Endpoints

- `GET /api/vouchers` — List vouchers (with filters)
- `GET /api/vouchers?voucherId=...` — Get a single voucher
- `POST /api/vouchers` — Create voucher (merchant/admin only)
- `PUT /api/vouchers?voucherId=...` — Update voucher (merchant/admin only)
- `DELETE /api/vouchers?voucherId=...` — Delete voucher (admin only)

## Query Parameters

| Name      | Type   | Required | Description          |
| --------- | ------ | -------- | -------------------- |
| voucherId | string | No       | Get a single voucher |
| userId    | string | No       | Filter by user       |
| orderId   | string | No       | Filter by order      |
| status    | string | No       | Filter by status     |

## Request Body (POST)

| Field       | Type   | Required | Description                                      |
| ----------- | ------ | -------- | ------------------------------------------------ |
| orderId     | string | Yes      | Order for the voucher                            |
| userId      | string | Yes      | User receiving the voucher                       |
| code        | string | Yes      | Unique voucher code                              |
| status      | string | No       | Enum: 'active', 'used', 'invalidated', 'expired' |
| deliveredAt | string | No       | ISO 8601 date                                    |

## Response (GET all)

```json
{
  "items": [
    {
      "id": "string",
      "orderId": "string",
      "userId": "string",
      "code": "string",
      "status": "active|used|invalidated|expired",
      "deliveredAt": "2024-06-01T12:00:00Z|null",
      "createdAt": "2024-06-01T12:00:00Z",
      "updatedAt": "2024-06-01T12:00:00Z"
    }
  ]
}
```

## Example: Creating a Voucher

```js
import axios from "axios";

async function createVoucher() {
  const res = await axios.post(
    "/api/vouchers",
    {
      orderId: "123",
      userId: "456",
      code: "VOUCHER123",
      status: "active",
    },
    { withCredentials: true }
  );
  console.log(res.data);
}
```

---

# Groups API

## Overview

- **Purpose:** Manage group purchasing groups for deals (create, join, leave, checkout)
- **Who can use:**
  - `GET`: Anyone for public groups, authenticated for group details
  - `POST`, `PATCH`: Authenticated users

## Endpoints

- `GET /api/groups` — List all groups (with optional filters)
- `POST /api/groups` — Create a new group for a deal
- `GET /api/groups/[groupId]` — Get a specific group with members
- `POST /api/groups/[groupId]/join` — Join an existing group
- `POST /api/groups/[groupId]/leave` — Leave a group
- `POST /api/groups/[groupId]/checkout` — Process payment for a group purchase
- `GET /api/groups/[groupId]/invite` — Get invite link/info
- `POST /api/groups/[groupId]/invite` — Send email invites

## Query Parameters for GET /api/groups

| Name   | Type   | Required | Description             |
| ------ | ------ | -------- | ----------------------- |
| dealId | string | No       | Filter by deal          |
| status | string | No       | Filter by group status  |

## Request Body for POST /api/groups (Create Group)

| Field  | Type   | Required | Description               |
| ------ | ------ | -------- | ------------------------- |
| dealId | string | Yes      | The deal to create a group for |

## Response for GET /api/groups

```json
{
  "items": [
    {
      "id": "string",
      "dealId": "string",
      "creatorUserId": "string",
      "status": "open | full | completed | expired",
      "createdAt": "2024-05-21T00:00:00.000Z",
      "updatedAt": "2024-05-21T00:00:00.000Z"
    }
  ]
}
```

## Response for GET /api/groups/[groupId]

```json
{
  "id": "string",
  "status": "open | full | completed | expired",
  "createdAt": "2024-05-21T00:00:00.000Z",
  "deal": {
    "id": "string",
    "title": "string",
    "discountedPrice": 80,
    "minGroupSize": 5,
    "maxGroupSize": 10
  },
  "creator": {
    "id": "string",
    "name": "string"
  },
  "members": [
    {
      "id": "string",
      "userId": "string",
      "joinedAt": "2024-05-21T00:00:00.000Z",
      "paymentStatus": "pending | paid | failed | left",
      "User": {
        "id": "string",
        "name": "string",
        "email": "string"
      }
    }
  ],
  "memberCount": 3
}
```

## Request Body for POST /api/groups/[groupId]/checkout

| Field         | Type   | Required | Description                |
| ------------- | ------ | -------- | -------------------------- |
| paymentMethod | object | Yes      | Payment method information |

## Response for POST /api/groups/[groupId]/checkout

```json
{
  "success": true,
  "paymentStatus": "paid",
  "groupStatus": "completed",
  "message": "Payment successful"
}
```

## Example: Create a Group

```javascript
import axios from 'axios';

async function createGroup(dealId) {
  try {
    const response = await axios.post('/api/groups', {
      dealId: dealId
    });
    return response.data;
  } catch (error) {
    console.error('Error creating group:', error);
    throw error;
  }
}
```

## Example: Join a Group

```javascript
import axios from 'axios';

async function joinGroup(groupId) {
  try {
    const response = await axios.post(`/api/groups/${groupId}/join`);
    return response.data;
  } catch (error) {
    console.error('Error joining group:', error);
    throw error;
  }
}
```

## Example: Complete Checkout

```javascript
import axios from 'axios';

async function checkout(groupId, paymentMethod) {
  try {
    const response = await axios.post(`/api/groups/${groupId}/checkout`, {
      paymentMethod: paymentMethod
    });
    return response.data;
  } catch (error) {
    console.error('Error processing payment:', error);
    throw error;
  }
}
```

---

# Error Responses

All endpoints may return the following error responses:

| Status | Response Example                       |
| ------ | -------------------------------------- |
| 400    | `{ "error": "Invalid input" }`         |
| 401    | `{ "error": "Unauthorized" }`          |
| 403    | `{ "error": "Forbidden" }`             |
| 404    | `{ "error": "Not found" }`             |
| 500    | `{ "error": "Internal server error" }` |

---

# Best Practices

- Always check for `error` in the response.
- Use `credentials: 'include'` for all authenticated requests.
- Use the correct query param for each model.
- For authentication, see [KUPONNA_AUTHENTICATION_DOCUMENTATION.md].
- For advanced filtering, combine multiple query params as needed.

---
