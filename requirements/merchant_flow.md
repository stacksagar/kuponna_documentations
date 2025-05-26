# Merchant End-to-End Flow

This document outlines the end-to-end flow for a merchant on the platform, from registration to managing deals, processing orders, and handling inquiries.

![Merchant Flow Diagram](../../requirements/merchant.jpg)

## Flow Description

1.  **Merchant Registers/Login**: The process begins with the merchant either registering as a new merchant or logging in if they already have an account.
2.  **New Merchant?**: A decision point to check if the merchant is new.
3.  **Merchant provides business details**: If a new merchant, they provide their business details.
4.  **Admin verifies & approves merchant**: The Admin verifies and approves the merchant account.
5.  **Merchant accesses dashboard**: After successful registration or login, the merchant accesses their dashboard.
6.  **Merchant creates a new deal**: The merchant creates a new deal.
7.  **Merchant sets deal details (title, price, discount, min group size)**: The merchant provides details for the deal.
8.  **Merchant uploads images**: The merchant uploads images for the deal.
9.  **Merchant submits deal for approval**: The merchant submits the deal for Admin approval.
10. **Admin approval required?**: A decision point if Admin approval is required.
11. **Admin reviews & approves deal**: If approval is required, the Admin reviews and approves the deal.
12. **Deal is published in the marketplace**: The approved deal is published in the marketplace.
13. **Users start joining the deal**: Users start joining the deal.
14. **Minimum group size met?**: A check to see if the minimum group size for the deal is met.
15. **Notify merchant about deal progress**: The system notifies the merchant about the deal progress.
16. **Deal gets activated**: Once the minimum group size is met (if applicable), the deal gets activated.
17. **Users make payments**: Users make payments for the activated deal.
18. **Orders are generated for the merchant**: Orders are generated for the merchant.
19. **User requested Redemption Card?**: A decision point if the user requested a Redemption Card.
20. **Merchant tracks Redemption Card delivery status**: If a Redemption Card was requested, the merchant tracks its delivery status.
21. **Physical product?**: A decision point if the deal involves a physical product.
22. **Merchant processes order**: If a physical product, the merchant processes the order.
23. **Merchant ships the product**: The merchant ships the physical product.
24. **Merchant generates digital voucher**: If not a physical product, the merchant generates a digital voucher.
25. **Merchant delivers voucher via email/SMS**: The merchant delivers the digital voucher via email or SMS.
26. **Merchant receives payout**: The merchant receives payout for the completed orders.
27. **Merchant tracks earnings & transactions**: The merchant can track their earnings and transactions.
28. **Customer inquiries?**: A decision point if there are customer inquiries.
29. **Merchant responds to customer messages**: If there are inquiries, the merchant responds to customer messages.
30. **Refund request received?**: A decision point if a refund request is received.
31. **Merchant reviews refund request**: If a refund request is received, the merchant reviews it.
32. **Approved?**: A decision point if the refund request is approved.
33. **Refund processed**: If approved, the refund is processed.
34. **Voucher invalidated (if digital)**: If the refund is for a digital voucher, the voucher is invalidated.
35. **Customer notified**: The customer is notified about the refund status.
36. **Flow ends**: The merchant flow concludes.
