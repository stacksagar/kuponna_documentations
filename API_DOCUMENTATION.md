# Kuponna API Documentation

## Base URL
```
Production: https://kuponna.up.railway.app/api
```

## Authentication
For authenticated endpoints, include the token in request headers:
```
Authorization: Bearer <your-token>
```

## Response Format
```json
{
  "success": true,
  "data": {},
  "pagination": { "total": 100, "limit": 20, "offset": 0 }
}
```

---

## Public Endpoints

### Authentication

**POST** `/api/auth/register` - Register new user
- Required: `name`, `email`, `password`, `role` (`user` | `merchant`)

**POST** `/api/auth/login` - User login
- Required: `email`, `password`

**POST** `/api/auth/logout` - User logout

**POST** `/api/auth/forgot-password` - Request password reset
- Required: `email`

**POST** `/api/auth/reset-password` - Reset password with token
- Required: `token`, `newPassword`

**POST** `/api/auth/verify-email` - Verify email with code
- Required: `code`

**POST** `/api/auth/resend-verification` - Resend verification email

### Deals

**GET** `/api/deals` - Get all deals
- `category`: category name
- `type`: `physical`, `digital`
- `featured`: `true`, `false`
- `search`: search text
- `limit`, `offset`: pagination

**GET** `/api/deals?id={id}` - Get single deal

### Categories

**GET** `/api/categories` - Get all categories
- `withDeals`: `true`, `false`
- `withDealCount`: `true`, `false`
- `limit`, `offset`: pagination

**GET** `/api/categories?id={id}` - Get single category

### Reviews

**GET** `/api/reviews?dealId={dealId}` - Get reviews for a deal

**POST** `/api/reviews` - Create review
- Required: `dealId`, `userId`, `rating`, `title`, `content`

---

## Data Models

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  role: "user" | "merchant" | "admin";
  isVerified: boolean;
  picture?: string;
  phoneNumber?: string;
}

interface Deal {
  id: string;
  title: string;
  description: string;
  pricePerPerson: number;
  discountPrice: number;
  requiredMembers: number;
  currentMembers: number;
  category: string;
  type: "physical" | "digital";
  status: "draft" | "pending" | "active" | "completed" | "cancelled";
  images: string[];
  averageRating: number;
  reviewCount: number;
  merchant: {
    id: string;
    name: string;
    email: string;
  };
}

interface Category {
  id: string;
  name: string;
  dealCount?: number;
}

interface Group {
  id: string;
  dealId: string;
  creatorId: string;
  status: "open" | "full" | "completed" | "expired";
  maxGroupSize: number;
  currentMembers: number;
  deal: Deal;
  creator: User;
  groupMembers: GroupMember[];
}

interface GroupMember {
  id: string;
  groupId: string;
  userId: string;
  paymentStatus: "pending" | "paid" | "failed" | "left";
  user: User;
}

interface Order {
  id: string;
  dealId: string;
  userId: string;
  groupId: string;
  amount: number;
  quantity: number;
  status: "pending" | "paid" | "shipped" | "delivered" | "cancelled" | "fulfilled";
  paymentStatus: "paid" | "unpaid" | "failed";
  type: "physical" | "digital";
}

interface Review {
  id: string;
  dealId: string;
  userId: string;
  rating: number;
  title: string;
  content: string;
  user: User;
}

interface Message {
  id: string;
  chatGroupId: string;
  senderId: string;
  content: string;
  readBy: string[];
  sender: User;
}

interface ChatGroup {
  id: string;
  name: string;
  isGroup: boolean;
  members: GroupMember[];
}
```
