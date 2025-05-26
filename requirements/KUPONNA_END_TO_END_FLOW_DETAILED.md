# Kuponna End-to-End User & Developer Flow (Detailed, Step-by-Step)

This document provides a complete, click-by-click and API-by-API breakdown of the Kuponna group purchase process. It covers every user action, UI element, backend call, and data change from signup to order completion.

---

## 1. User Signup & Login

### UI/UX
- User lands on the homepage.
- Clicks **Sign Up** button (`/signup` page).
- Fills in name, email, password, clicks **Register**.

### Backend/API
- `POST /api/auth/register` with `{ name, email, password }`.
- System creates user with status `pending_verification`.
- Sends verification email with token.

### UI/UX
- User receives email, clicks **Verify Email** link.

### Backend/API
- `GET /api/auth/verify?token=...`
- System sets user status to `active`.

### UI/UX
- User logs in via **Login** button (`/login` page).
- `POST /api/auth/login` with `{ email, password }`.
- On success, user is redirected to `/dashboard`.

---

## 2. Browsing & Selecting Deals

### UI/UX
- On dashboard, user sees a list of deals (cards or list view).
- Can filter/search by category, price, location using filter bar.
- Clicks **View Deal** button on a deal card.

### Backend/API
- `GET /api/deals` (with filters as query params).
- `GET /api/deals/:dealId` for deal details.

### UI/UX
- On deal details page:
  - Sees deal info, price, min group size, current groups.
  - Sees two buttons: **Buy Individually** and **Join Group Purchase**.

---

## 3. Group Purchase Initiation & Joining (In-Depth)

### UI/UX
- On deal details page, user clicks **Join Group Purchase**.
- Modal or section appears:
  - Option to **Join Existing Group** (shows open groups with available spots)
  - Or **Create New Group** (button)

#### a) Join Existing Group
- User sees a list of open groups for this deal (e.g., "Group #123: 3/5 joined").
- User clicks **Join Group** button next to a group.
- System checks if group is not full and user is not already a member.
- If valid, user is added to the group as a `GroupMember` (status: `pending`).
- UI redirects user to the **Group Page** (`/groups/:groupId`), showing group progress, members, and invite options.
- If group becomes full after this join, system updates group status to `full` and notifies all members.

#### b) Create New Group
- User clicks **Create New Group**.
- System creates a new `Group` record (status: `open`, dealId, creatorUserId).
- User is added as the first `GroupMember` (status: `pending`).
- UI redirects user to the **Group Page** (`/groups/:groupId`), showing group progress, members, and invite options.

### API/Backend
- `GET /api/deals/:dealId/groups?status=open` (list open groups)
- `POST /api/groups/:groupId/join` (join existing group)
- `POST /api/groups` (create new group)
- On join/create, system creates/updates `Group` and `GroupMember` models accordingly.

### Data Model
- **Group**: Created (C) if new, Read (R) if joining existing
- **GroupMember**: Created (C) for each join
- Group status: Updated (U) to `full` if min size reached

### What Happens Next?
- User lands on the **Group Page**:
  - Sees group progress bar (e.g., `4/5 joined`)
  - Sees list of members
  - Sees **Invite Friends** button (copies invite link or sends email)
  - If group is full, sees **Checkout** button enabled
- As more users join, group progress updates in real time.
- If group is not full, user can invite others; if full, all members are notified to proceed to checkout.

---

## 4. Inviting & Joining Group Members (Summary)

### UI/UX
- After joining/creating, user lands on **Group Page** (`/groups/:groupId`).
- Sees group progress bar (e.g., `2/5 joined`).
- Sees list of members.
- Sees **Invite Friends** button (copies invite link or sends email).
- Invite link: `/deals/:dealId/groups/:groupId/join`

### Backend/API
- `GET /api/groups/:groupId` for group status and members.
- `POST /api/groups/:groupId/invite` (optional, for email invites).

### Data Changes
- Each new member: `POST /api/groups/:groupId/join`.
- Group member status: `pending`.
- Group status updates as members join.

---

## 5. Group Size Fulfillment, Checkout & Payment

### UI/UX
- When group reaches min size (e.g., 5/5):
  - UI shows **Group Full! Proceed to Checkout** banner.
  - All members see **Checkout** button enabled.
  - All members receive notification (email/in-app).

### Backend/API
- System updates group status to `full`.
- Notifies all group members.

### Data Changes
- Group status: `full`.

---

## 6. Order Generation & Merchant Notification

### Backend/API
- For each paid member:
  - `POST /api/orders` with `{ userId, groupId, dealId }`.
  - Order status: `processing`.
- System notifies merchant (email, dashboard notification).

### Data Changes
- New order records for each member.
- Order status: `processing`.

---

## 7. Fulfillment (Physical or Digital)

### Merchant UI/UX
- Merchant sees new group order in dashboard.
- For physical: clicks **Ship Order**, enters tracking info.
- For digital: clicks **Send Voucher**, enters code/link.

### Backend/API
- For physical: `POST /api/orders/:orderId/ship` with tracking info.
- For digital: `POST /api/orders/:orderId/deliver` with voucher info.
- System updates order status: `shipped` (physical) or `delivered` (digital).
- Notifies user.

### Data Changes
- Order status: `processing` → `shipped`/`delivered`.
- Tracking/voucher info saved.

---

## 8. Order Receipt & Completion

### UI/UX
- User receives notification (email/in-app).
- On **Order Details** page, sees **Mark as Received** (physical) or **Redeem Voucher** (digital) button.
- Clicks button to confirm receipt.

### Backend/API
- `POST /api/orders/:orderId/complete`.
- System updates order status to `completed`.

### Data Changes
- Order status: `completed`.

---

## 9. Post-Order Actions

### UI/UX
- User can rate the deal (**Rate Deal** button).
- Can click **Contact Support** or **Request Refund** (if eligible).
- If refund: fills form, clicks **Submit Refund Request**.

### Backend/API
- `POST /api/orders/:orderId/rate` with rating/comment.
- `POST /api/orders/:orderId/support` for support ticket.
- `POST /api/orders/:orderId/refund` for refund request.
- Admin/merchant reviews and processes requests.

### Data Changes
- New rating/support/refund records.
- Order status: `refund requested` → `refunded` (if approved).

---

## 10. Statuses & Notifications
- All status changes trigger notifications (email, in-app): group full, payment, shipping, delivery, refund, etc.
- Group: `open` → `full` → `completed` (or `expired`)
- Member: `pending` → `paid`/`failed`
- Order: `processing` → `shipped`/`delivered` → `completed`/`refund requested`/`refunded`

---

# Deal Start-to-Finish Flow: User, UI, API, and Database Model Actions

This section is now merged into the above steps for clarity and to avoid duplication. All CRUD actions, button clicks, and model updates are described in context above.

---

*This document is a complete, step-by-step, user-and-developer-friendly flow for Kuponna group purchase, covering every UI action, API call, and data change, with in-depth explanation for joining/creating groups and model updates.*
