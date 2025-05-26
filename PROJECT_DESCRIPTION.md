# Kuponna Project Description

This project, named `group-deal` (as per `package.json`), is a Next.js application bootstrapped with `create-next-app`. It appears to be a platform for managing group deals, orders, and related functionalities for users and merchants.

## Technologies Used

The project utilizes the following key technologies and libraries:

- **Framework:** Next.js
- **Frontend:** React, Mantine (UI library), Framer Motion (animations), Styled Components
- **State Management/Data Fetching:** Axios
- **Backend/Database:** Sequelize (ORM), MySQL2 (database connector)
- **Authentication:** Next-Auth, jsonwebtoken, bcryptjs
- **Other Dependencies:** date-fns, d3-geo, lucide-react (icons), nodemailer, react-date-range, react-icons, react-qr-code, react-select, react-svg-worldmap, react-toastify, recharts, swiper, uid, zod (schema validation)

## Getting Started

To get started with the project, first clone the repository and install the dependencies. Then, run the development server:

```bash
npm install
# or
yarn install
# or
pnpm install
# or
bun install
```

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

The project uses `next/font` to automatically optimize and load Geist font.

## API Overview

The project exposes a comprehensive set of APIs to manage various aspects of the platform. Detailed documentation for the API endpoints can be found in `docs/API_DOCUMENTATION.md`. The main API areas include:

- **Deals:** Manage group deals (create, update, list, delete).
- **Deal Participants:** Handle users joining and leaving deals.
- **Deal Images:** Manage images associated with deals.
- **Orders:** Manage orders placed for deals.
- **Redemption Cards:** Manage redemption cards for order fulfillment.
- **Refund Requests:** Handle user requests for refunds.
- **Transactions:** Manage payment, payout, and refund transactions.
- **Messages:** Facilitate messaging between users, merchants, and admins.
- **Notifications:** Manage notifications for users and merchants.
- **Delivery Tracking:** Track the delivery status of orders.
- **Merchant Profile:** Manage merchant business profiles.
- **Complaints:** Handle user complaints and refund requests.
- **Vouchers:** (Mentioned in API docs, likely for managing discount vouchers)
- **Groups:** (Mentioned in API docs, likely related to group purchase functionality)

The API documentation provides details on endpoints, required parameters, request bodies, and response structures for each area.

## Project Structure (Key Directories)

- `docs/`: Contains project documentation, including API documentation and requirements.
- `public/`: Contains static assets like images, icons, and branding.
- `src/`: Contains the main application source code, including:
    - `app/`: Next.js application pages and routing.
    - `components/`: Reusable React components.
    - `config/`: Configuration files.
    - `data/`: Data-related files.
    - `hooks/`: Custom React hooks.
    - `lib/`: Utility functions and libraries.
    - `models/`: Database models (likely using Sequelize).
    - `server/`: Server-side code, including API routes.
    - `templates/`: Email or other templates.
    - `types/`: TypeScript type definitions.
    - `utils/`: General utility functions.

This document provides a high-level overview of the Kuponna project. For detailed API information, please refer to `docs/API_DOCUMENTATION.md`.
