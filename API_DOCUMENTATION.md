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
 
### Deals

**GET** `/api/deals` - Get all deals
- `category`: category name
- `type`: `physical`, `digital`
- `featured`: `true`, `false`
- `search`: search text
- `limit`, `offset`: pagination

**GET** `/api/deals?id={id}` - Get single deal

**GET** `/api/deals/featured` - Get featured deals only
- `status`: deal status (default: `active`)
- `limit`, `offset`: pagination

**GET** `/api/deals/top-weekly` - Get top weekly deals
- `limit`, `offset`: pagination

**GET** `/api/deals/best-selling` - Get best selling deals
- `limit`, `offset`: pagination

**GET** `/api/deals/greetings` - Get greeting cards and celebration deals
- `limit`, `offset`: pagination

**GET** `/api/deals/group-discounts` - Get deals with group discounts
- `minDiscount`: minimum discount percentage/amount
- `limit`, `offset`: pagination

**GET** `/api/deals/recommendations` - Get recommended deals
- `category`: filter by category
- `limit`, `offset`: pagination

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

## Authenticated Endpoints

### Messages

**GET** `/api/user/messages` - Get messages for a chat
- `chatGroupId`: get messages for group chat
- `recipientId`: get messages for one-on-one chat

**POST** `/api/user/messages` - Send a message
- Required: `content`
- Required: `chatGroupId` OR `recipientId`

### Message Groups

**GET** `/api/user/messages/groups` - Get user's chat groups

**POST** `/api/user/messages/groups` - Create group chat
- Required: `name`, `memberIds[]`

### Group Members

**POST** `/api/user/messages/groups/{groupId}/members` - Add member to group
- Required: `userId`

**DELETE** `/api/user/messages/groups/{groupId}/members` - Remove member from group
- Required: `userId` (as query parameter)

---

## Data Models

```typescript
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

interface Review {
  id: string;
  dealId: string;
  userId: string;
  rating: number;
  title: string;
  content: string;
  user: {
    id: string;
    name: string;
    picture?: string;
  };
}

interface Message {
  id: string;
  chatGroupId: string;
  senderId: string;
  content: string;
  readBy: string[];
  createdAt: string;
  sender: {
    id: string;
    name: string;
    picture?: string;
  };
}

interface ChatGroup {
  id: string;
  name: string;
  isGroup: boolean;
  avatar?: string;
}

interface ChatGroupMember {
  id: string;
  chatGroupId: string;
  userId: string;
}
```
