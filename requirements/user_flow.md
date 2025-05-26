# User End-to-End Flow

This document outlines the end-to-end flow for a user on the platform, from registration to various actions like purchasing deals, requesting redemption cards, and contacting support.

![User Flow Diagram](../../requirements/user.PNG)

## Flow Description

1.  **User Registers/Login**: The process begins with the user either registering as a new user or logging in if they already have an account.
2.  **New User?**: A decision point to check if the user is new.
3.  **User provides name, email, password**: If a new user, they provide their registration details.
4.  **System verifies email**: The system verifies the user's email address.
5.  **User accesses dashboard**: After successful registration or login, the user accesses their dashboard.
6.  **User browses deals**: The user can browse available deals.
7.  **User filters/searches deals by category, price, location**: Users can refine their search for deals using filters.
8.  **User selects a deal?**: The user decides whether to select a deal.
9.  **View deal details**: If a deal is selected, the user views its details.
10. **User chooses to buy individually or join a group?**: The user decides whether to purchase the deal individually or join/create a group purchase.
11. **Joining a group purchase?**: A decision point for group purchase.
12. **User joins existing group or creates a new one**: If joining a group, the user can join an existing one or create a new one.
13. **User invites others to join**: The user can invite others to join the group.
14. **Group size met?**: A check to see if the minimum group size for the deal is met.
15. **Notify user about progress**: The system notifies the user about the group purchase progress.
16. **User proceeds to checkout**: Once ready, the user proceeds to checkout.
17. **User makes payment**: The user completes the payment for the deal.
18. **Invoice Needed?**: A decision point if an invoice is required.
19. **System generates invoice**: If an invoice is needed, the system generates it.
20. **User wants Redemption Card?**: The user indicates if they want a Redemption Card.
21. **User requests Redemption Card**: If desired, the user requests a Redemption Card.
22. **User receives tracking link from Admin**: The user receives a tracking link for the Redemption Card delivery from the Admin.
23. **User tracks status of Redemption Card delivery**: The user can track the delivery status.
24. **User confirms receipt with delivery personnel**: The user confirms receiving the Redemption Card.
25. **Physical product?**: A decision point if the deal involves a physical product.
26. **Merchant ships order**: If a physical product, the merchant ships the order.
27. **User receives physical product**: The user receives the physical product.
28. **User marks order as received**: The user confirms receiving the physical product.
29. **User receives digital coupon or Redemption Card**: If not a physical product, the user receives a digital coupon or Redemption Card.
30. **User redeems it at merchant location**: The user redeems the digital coupon or Redemption Card at the merchant's location.
31. **Redemption Card/Coupon is marked as used**: The Redemption Card or Coupon is marked as used.
32. **User wants to cancel order?**: The user decides if they want to cancel the order.
33. **User submits cancellation request**: If cancellation is desired, the user submits a request.
34. **Refund is processed if eligible**: If eligible, a refund is processed.
35. **User wants to contact support?**: The user decides if they need to contact support.
36. **User sends inquiry to merchant or support team**: The user sends an inquiry.
37. **Issue with order?**: A decision point if there is an issue with the order.
38. **User raises complaint ticket**: If there is an issue, the user raises a complaint ticket.
39. **Flow ends**: The user flow concludes.
