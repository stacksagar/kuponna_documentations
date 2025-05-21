# Kuponna Platform Requirements

This document summarizes the core requirements and workflows for the Kuponna platform, based on the provided user, admin, and merchant flowcharts.

---

## 1. User Requirements

### Registration & Login
- Users can register or log in.
- New users provide name, email, and password.
- Email verification is required.

### Dashboard & Browsing
- Users access a dashboard to browse available deals.
- Deals can be filtered/searched by category, price, and location.

### Deal Selection & Group Purchase
- Users can view deal details.
- Option to buy individually or join a group purchase.
- Users can join existing groups or create new ones and invite others.
- Users are notified about group progress until the minimum group size is met.

### Checkout & Payment
- Users proceed to checkout and make payments.
- Option to request an invoice.

### Redemption & Delivery
- Users can request a Redemption Card (physical or digital).
- Tracking and confirmation of Redemption Card delivery.
- For physical products: merchant ships order, user receives and marks as received.
- For digital products: user receives coupon/card and redeems at merchant location.

### Order Management
- Users can cancel orders and request refunds (if eligible).
- Users can contact support or merchants for inquiries.
- Users can raise complaint tickets for order issues.

---

## 2. Admin Requirements

### Platform Management
- Admin logs in and accesses dashboard with analytics and system status.

### Merchant Management
- Admin reviews new merchant registrations and verifies details.
- Admin approves or rejects merchant applications.

### Deal Management
- Admin reviews and approves/rejects new deals submitted by merchants.
- Ensures deals follow platform guidelines.

### Complaint & Refund Handling
- Admin reviews user complaints and refund requests.
- Investigates and processes valid complaints/refunds.
- Notifies merchants and users of outcomes.

### Redemption Card Management
- Admin manages Redemption Card orders and shipping.
- Updates tracking status and notifies users/merchants.

### Platform Issue Escalation
- Admin escalates technical issues to support as needed.

---

## 3. Merchant Requirements

### Registration & Verification
- Merchants register and provide business details.
- Admin verifies and approves merchant accounts.

### Deal Creation & Management
- Merchants create new deals (title, price, discount, min group size, images).
- Submit deals for admin approval.
- Track deal progress and group size.

### Order Fulfillment
- Process orders once group size is met and payments are made.
- Ship physical products or generate/deliver digital vouchers.
- Track Redemption Card delivery if requested.

### Payouts & Transactions
- Merchants receive payouts and track earnings/transactions.

### Customer Support
- Respond to customer inquiries.
- Review and process refund requests, invalidate vouchers if needed.

---

## 4. General Platform Features
- Email notifications for key events (registration, deal progress, shipping, etc.).
- Support for both physical and digital products.
- Group buying functionality with progress tracking.
- Integrated support and complaint management system.

---

## 5. Group Purchase, Group, and Group Size Explained

### What is a Group?
A **group** is a collection of users who come together to purchase a deal collectively. Groups are formed to take advantage of special discounts or offers that are only available if a minimum number of people participate in the purchase.

### What is Group Purchase?
A **group purchase** (also known as group buying) is a process where multiple users join together to buy a product or service as a group. This allows them to access lower prices or special deals that are not available to individual buyers.

### What is Group Size?
**Group size** refers to the number of users required to successfully activate a group deal. Each deal that supports group purchase will specify a minimum group size (e.g., 5 users). The deal is only activated and processed if this minimum number of users join the group purchase.

### Relationship Between Deal and Group Size
- Each deal can have a minimum group size set by the merchant (e.g., "This deal requires at least 5 buyers").
- Users can join an existing group for a deal or create a new group and invite others.
- The deal is only confirmed and processed when the group size requirement is met.
- If the group size is not met within a specified time, the deal may be canceled or users may be notified to invite more participants.
- Once the group size is met, users proceed to payment and the order is fulfilled by the merchant.

### Example Scenario
1. A merchant creates a deal: "50% off on dinner for a group of 5 or more."
2. User A wants the deal and joins or creates a group for this deal.
3. User A invites friends (Users B, C, D, E) to join the group.
4. When 5 users have joined, the group size is met, and the deal is activated.
5. All group members pay, and the merchant processes the order.

---

*This document is based on the provided flowcharts and is intended to guide development and stakeholder understanding of the Kuponna platform.*
