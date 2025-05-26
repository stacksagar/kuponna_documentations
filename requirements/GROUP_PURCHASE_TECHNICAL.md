# Group Purchase Process: Technical Breakdown (Next.js)

This document explains the technical flow for group purchase in the Kuponna platform, covering database models, API endpoints, and frontend flow.

---

## 1. Database Models (Entities)

### Deal
- `id`
- `title`
- `description`
- `price`
- `minGroupSize`
- `merchantId`
- `status` (active, expired, etc.)

### Group
- `id`
- `dealId` (FK to Deal)
- `creatorUserId`
- `status` (open, full, expired, completed)
- `createdAt`

### GroupMember
- `id`
- `groupId` (FK to Group)
- `userId`
- `joinedAt`
- `paymentStatus` (pending, paid, failed)

### Order
- `id`
- `userId`
- `groupId`
- `dealId`
- `status` (pending, paid, fulfilled, canceled)
- `redemptionType` (physical, digital)

---

## 2. API Endpoints (Examples)

- `GET /api/deals` — List all deals
- `GET /api/deals/:id` — Deal details
- `POST /api/groups` — Create a new group for a deal
- `POST /api/groups/:groupId/join` — Join an existing group
- `GET /api/groups/:groupId` — Get group status and members
- `POST /api/groups/:groupId/invite` — Invite users (optional)
- `POST /api/groups/:groupId/checkout` — Initiate checkout for group members
- `POST /api/orders` — Create order after payment
- `GET /api/orders/:orderId` — Order status

---

## 3. Frontend Flow (Pages/Components)

1. **Deal Listing Page**
   - Shows all available deals
   - Each deal shows group purchase option if enabled

2. **Deal Details Page**
   - Shows deal info, min group size, current groups
   - User can join existing group or create a new group

3. **Group Page**
   - Shows group progress (e.g., 3/5 joined)
   - List of members, invite option
   - Notifies when group is full

4. **Checkout Page**
   - Each member pays for their share
   - Payment status tracked per member

5. **Order Confirmation Page**
   - Shows order status, redemption info

6. **Order Tracking/History Page**
   - User can track delivery or voucher status

---

## 4. Example User Flow

1. User selects a deal with min group size 5
2. User creates a new group (or joins existing)
3. User invites friends to join (group status: 1/5, 2/5, ...)
4. When 5/5 joined, all members are notified
5. Each member completes payment
6. Orders are generated and merchant is notified
7. Merchant fulfills order (ships product or sends voucher)
8. Users receive product/voucher and can mark as received

---

## 5. Notes
- Groups can have an expiration time (e.g., 24h to fill group)
- If group size not met, group is canceled and no payment is processed
- All group members must pay for the group order to proceed

---

*This document is a technical guide for implementing group purchase in a Next.js app.*
