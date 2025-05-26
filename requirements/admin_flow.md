# Admin End-to-End Flow

This document outlines the end-to-end flow for an Admin on the platform, covering tasks from logging in to managing merchant applications, deals, complaints, and redemption card orders.

![Admin Flow Diagram](../../requirements/admin.jpg)

## Flow Description

1.  **Admin Logs in**: The process begins with the Admin logging in.
2.  **Admin accesses dashboard**: After logging in, the Admin accesses their dashboard.
3.  **Admin views platform analytics and system status**: The Admin can view platform analytics and system status.
4.  **New merchant registration?**: A decision point to check for new merchant registrations.
5.  **Admin reviews merchant details**: If there are new registrations, the Admin reviews the merchant details.
6.  **Merchant verification passed?**: A decision point on whether the merchant verification passed.
7.  **Admin approves merchant account**: If verification passes, the Admin approves the merchant account.
8.  **Admin rejects application**: If verification fails, the Admin rejects the application.
9.  **New deal submitted?**: A decision point to check for new deal submissions.
10. **Admin reviews deal details**: If there are new deals, the Admin reviews the deal details.
11. **Deal follows guidelines?**: A decision point on whether the deal follows the guidelines.
12. **Admin approves deal and publishes it**: If the deal follows guidelines, the Admin approves and publishes it.
13. **Admin rejects deal and notifies merchant**: If the deal does not follow guidelines, the Admin rejects it and notifies the merchant.
14. **User complaints or refund requests?**: A decision point to check for user complaints or refund requests.
15. **Admin reviews user complaints**: If there are complaints or refund requests, the Admin reviews them.
16. **Valid complaint?**: A decision point on whether the complaint is valid.
17. **Admin investigates complaint**: If the complaint is valid, the Admin investigates it.
18. **Refund is valid?**: A decision point on whether a refund is valid.
19. **Admin processes refund**: If a refund is valid, the Admin processes it.
20. **Admin rejects refund request**: If a refund is not valid, the Admin rejects the refund request.
21. **Admin rejects complaint**: If the complaint is not valid, the Admin rejects the complaint.
22. **Admin notifies merchant and user**: The Admin notifies the merchant and user about the outcome of the complaint or refund request.
23. **Platform issue detected?**: A decision point to check if a platform issue is detected.
24. **Admin escalates to tech support**: If a platform issue is detected, the Admin escalates it to tech support.
25. **Redemption Card Orders Exist?**: A decision point to check if Redemption Card orders exist.
26. **Admin views Redemption Card orders**: If Redemption Card orders exist, the Admin views them.
27. **Admin ships Redemption Card via delivery personnel**: The Admin ships the Redemption Card using delivery personnel.
28. **Admin updates tracking status**: The Admin updates the tracking status for the Redemption Card delivery.
29. **Admin notifies user & merchant of delivery status**: The Admin notifies the user and merchant about the delivery status.
30. **Admin logs out**: The Admin logs out of the system.
31. **Flow ends**: The Admin flow concludes.
