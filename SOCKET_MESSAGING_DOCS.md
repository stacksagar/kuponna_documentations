# Express.js Messaging API Documentation

This API provides real-time messaging and REST endpoints for both web (Next.js) and mobile clients, using the same models and authentication as your Next.js app.

---

## Table of Contents
- [Setup](#setup)
- [Authentication](#authentication)
- [REST API Endpoints](#rest-api-endpoints)
- [Socket.IO Events](#socketio-events)
- [Data Models](#data-models)
- [Environment Variables](#environment-variables)
- [Example Usage](#example-usage)

---

## Setup

1. **Install dependencies:**
   ```bash
   cd expressjs-api
   npm install
   ```
2. **Configure environment:**
   - Copy `.env.example` to `.env` and fill in your DB, JWT, and other secrets.
3. **Run the server (dev mode):**
   ```bash
   npm run dev
   ```
4. **Build and start (production):**
   ```bash
   npm run build
   npm start
   ```

---

## Key Features

### Comprehensive Relationship Data
**All API responses and Socket.IO events include complete relationship data to minimize additional API calls:**

- **Messages** always include:
  - Complete sender information (name, picture, email, role)
  - Full chat group details with all members
  - Related group/order information when applicable

- **Chat Groups** always include:
  - All group members with their user details
  - Related group creator information (if applicable)
  - Related order user information (if applicable)

- **Real-time Updates** via Socket.IO:
  - Automatic room joining for all user's groups on connection
  - Real-time message delivery with complete data
  - Group membership updates
  - Read receipt functionality

### Auto-Admin Inclusion
- When creating one-on-one chats, all admin users are automatically added to ensure proper oversight
- Admins can monitor and moderate all conversations

---

## Authentication
- All endpoints and sockets use JWT authentication (same as Next.js app).
- Pass JWT as `Authorization: Bearer <token>` header for REST.
- For Socket.IO, send JWT in the `auth` payload on connect.

---

## REST API Endpoints

### Authentication

#### Login
- **POST** `/api/auth/login`
- **Send Data:**
  ```json
  {
    "email": "user@example.com",
    "password": "yourpassword",
    "rememberMe": false // optional
  }
  ```
- **Returns:** `User` model + JWT token
- **Response:**
  ```json
  {
    "message": "Login successful",
    "token": "<JWT>",
    "user": {
      // Complete User model (see Data Models section)
    }
  }
  ```

#### Refresh Token
- **GET** `/api/auth/refresh`
- **Headers:** `Authorization: Bearer <token>`
- **Send Data:** None (token in header)
- **Returns:** `User` model + new JWT token
- **Response:**
  ```json
  {
    "message": "Token refreshed successfully",
    "user": {
      // Complete User model (see Data Models section)
    },
    "token": "<new JWT>"
  }
  ```

### Messaging

#### Send Message (One-on-One or Group)
- **POST** `/api/messages`
- **Headers:** `Authorization: Bearer <token>`
- **Send Data:**
  ```json
  {
    "chatGroupId": "<groupId>", // for group chat (optional)
    "recipientId": "<userId>",  // for one-on-one chat (optional)
    "content": "Hello!"
  }
  ```
- **Returns:** `ChatMessage` model with `User` (sender) + `ChatGroup` (with `ChatGroupMember[]`)
- **Response:**
  ```json
  {
    "success": true,
    "message": {
      // Complete ChatMessage model with all relationships
      // See Data Models section for full structure
    }
  }
  ```

#### Fetch Messages
- **GET** `/api/messages?chatGroupId=<groupId>` or `/api/messages?recipientId=<userId>`
- **Headers:** `Authorization: Bearer <token>`
- **Send Data:** Query parameters only
- **Returns:** Array of `ChatMessage` models with `User` (sender) + `ChatGroup` (with `ChatGroupMember[]`)
- **Response:**
  ```json
  {
    "success": true,
    "count": 25,
    "messages": [
      // Array of ChatMessage models with all relationships
      // See Data Models section for full structure
    ]
  }
  ```

### Groups

#### Create Group Chat
- **POST** `/api/messages/groups`
- **Headers:** `Authorization: Bearer <token>`
- **Send Data:**
  ```json
  {
    "name": "Group Name",
    "memberIds": ["userId1", "userId2"]
  }
  ```
- **Returns:** `ChatGroup` model with all `ChatGroupMember[]` relationships
- **Response:**
  ```json
  {
    "success": true,
    "group": {
      // Complete ChatGroup model with all members
      // See Data Models section for full structure
    }
  }
  ```

#### List Groups
- **GET** `/api/messages/groups`
- **Headers:** `Authorization: Bearer <token>`
- **Query Parameters:**
  - `includeOneOnOne=true/false` (default: true)
  - `includeGroupChats=true/false` (default: true)
- **Send Data:** Query parameters only
- **Returns:** Array of `ChatGroupMember` models with `User` + `ChatGroup` (with all relationships)
- **Response:**
  ```json
  {
    "success": true,
    "count": 5,
    "groups": [
      // Array of ChatGroupMember models with complete relationships
      // See Data Models section for full structure
    ]
  }
  ```

---

## Socket.IO Events

### Connection & Authentication
- Connect to: `wss://kuponna-api.up.railway.app` (production)  
- Authenticate with JWT:
  ```js
  const socket = io('wss://kuponna-api.up.railway.app', {
    auth: { token: '<JWT>' }
  });

  // Listen for auto-emitted events on connection
  socket.on('groups:joined', (data) => {
    console.log('User joined groups:', data.groups);
    // Returns: Array of ChatGroupMember models with complete relationships
  });

  // Handle connection events
  socket.on('connect', () => {
    console.log('Connected to server');
  });

  socket.on('connect_error', (error) => {
    console.log('Connection failed:', error.message);
  });
  ```

### Events

#### `message:send`
- **Client Sends:**
  ```js
  socket.emit('message:send', {
    chatGroupId: '<groupId>', // for group chat (optional)
    recipientId: '<userId>',  // for one-on-one chat (optional)
    content: 'Hello!'
  }, (response) => {
    // Response callback
  });
  ```
- **Returns:** `ChatMessage` model with `User` (sender) + `ChatGroup` (with `ChatGroupMember[]`)
- **Response:**
  ```js
  { 
    success: true, 
    message: {
      // Complete ChatMessage model (see Data Models section)
    }
  }
  ```

#### `message:receive` (Auto-broadcast)
- **Server Sends to all group members:**
- **Returns:** `ChatMessage` model with `User` (sender) + `ChatGroup` (with `ChatGroupMember[]`)
  ```js
  socket.on('message:receive', (message) => {
    // Complete ChatMessage model with all relationships
    // See Data Models section for full structure
  });
  ```

#### `message:history`
- **Client Sends:**
  ```js
  socket.emit('message:history', {
    chatGroupId: '<groupId>', // for group chat (optional)
    recipientId: '<userId>'   // for one-on-one chat (optional)
  }, (response) => {
    // Response callback
  });
  ```
- **Returns:** Array of `ChatMessage` models with `User` (sender) + `ChatGroup` (with `ChatGroupMember[]`)
- **Response:**
  ```js
  { 
    messages: [
      // Array of ChatMessage models with all relationships
      // See Data Models section for full structure
    ]
  }
  ```

#### `message:markRead`
- **Client Sends:**
  ```js
  socket.emit('message:markRead', {
    messageIds: ['messageId1', 'messageId2'] // array of message IDs to mark as read
  }, (response) => {
    // Response callback
  });
  ```
- **Returns:** Success confirmation
- **Response:**
  ```js
  { success: true }
  ```

#### `groups:list`
- **Client Sends:**
  ```js
  socket.emit('groups:list', (response) => {
    // Response callback
  });
  ```
- **Returns:** Array of `ChatGroupMember` models with `User` + `ChatGroup` (with all relationships)
- **Response:**
  ```js
  { 
    groups: [
      // Array of ChatGroupMember models with complete relationships
      // See Data Models section for full structure
    ]
  }
  ```

#### `groups:joined` (Auto-emitted on connection)
- **Server Sends automatically on connection:**
- **Returns:** Array of `ChatGroupMember` models with `User` + `ChatGroup` (with all relationships)
  ```js
  socket.on('groups:joined', (data) => {
    // data: { 
    //   groups: [
    //     // Array of ChatGroupMember models with complete relationships
    //     // See Data Models section for full structure
    //   ]
    // }
  });
  ```

---

## Data Models

All API responses include complete model relationships. Here are the model structures:

### User Model
```typescript
{
  id: string (UUID)
  email: string
  name: string
  phoneNumber?: string
  picture?: string
  role: "user" | "merchant" | "admin"
  isVerified: boolean
  merchantVerified: boolean
  lastLogin?: Date
  merchantRole?: string
  country?: string
  timezone?: string
  bio?: string
  pushNotification: boolean
  emailNotification: boolean
  smsNotification: boolean
  available_balance: number
  book_balance: number
  monthly_target: number
  refund_balance: number
  categories: string[]
  preferences?: object
  deviceTokens?: Array<{
    deviceToken: string
    deviceType?: string
  }>
  createdAt: Date
  updatedAt: Date
}
```

### ChatMessage Model
```typescript
{
  id: string (UUID)
  chatGroupId: string (UUID)
  senderId: string (UUID)
  content: string
  readBy: string[] // Array of user IDs who read the message
  createdAt: Date
  
  // Included relationships:
  sender: User // Complete User model
  chatGroup: ChatGroup // Complete ChatGroup model with members
}
```

### ChatGroup Model
```typescript
{
  id: string (UUID)
  name: string
  isGroup: boolean // true = group chat, false = one-on-one
  groupId?: string (UUID) // Reference to Group model (if applicable)
  orderId?: string (UUID) // Reference to Order model (if applicable)
  userId?: string (UUID) // Reference to User model (if applicable)
  createdAt: Date
  updatedAt: Date
  
  // Included relationships:
  members: ChatGroupMember[] // Array of all group members
  group?: Group // If related to a Group
  order?: Order // If related to an Order
  user?: User // If related to a User
  
  // Additional computed fields (in groups list):
  memberCount?: number
  isUserMember?: boolean
  avatar?: string // For one-on-one chats
  otherUser?: User // For one-on-one chats
}
```

### ChatGroupMember Model
```typescript
{
  id: string (UUID)
  chatGroupId: string (UUID)
  userId: string (UUID)
  joinedAt: Date
  
  // Included relationships:
  user: User // Complete User model
  chatGroup: ChatGroup // Complete ChatGroup model with all relationships
}
```

### Group Model (Optional - when referenced)
```typescript
{
  id: string (UUID)
  dealId: string (UUID)
  creatorId?: string (UUID)
  status: "open" | "full" | "completed" | "expired"
  maxGroupSize?: number
  description?: string
  createdAt: Date
  updatedAt: Date
  
  // Included relationships:
  creator?: User // Complete User model
  deal?: Deal // Complete Deal model
  
  // Additional computed fields:
  currentMembers?: number
  isFull?: boolean
}
```

### Order Model (Optional - when referenced)
```typescript
{
  id: string (UUID)
  dealId: string (UUID)
  groupId: string (UUID)
  userId: string (UUID)
  deliveryFee?: number
  dealIds?: string[]
  // ... other order fields
  createdAt: Date
  updatedAt: Date
  
  // Included relationships:
  user: User // Complete User model
  deal?: Deal // Complete Deal model
  group?: Group // Complete Group model
}
```

### Deal Model (Optional - when referenced)
```typescript
{
  id: string (UUID)
  title: string
  description: string
  price: number
  discountPrice?: number
  // ... other deal fields
  createdAt: Date
  updatedAt: Date
}
```

---

## Environment Variables
See `.env.example` for all required variables:
- `DB_NAME`, `DB_USER`, `DB_PASS`, `DB_HOST`, `DB_PORT`
- `JWT_SECRET`
- `NEXT_PUBLIC_GOOGLE_OAUTH_CLIENT_ID`, `GOOGLE_OAUTH_CLIENT_SECRET` (if using Google login)
- `CLOUDINARY_URL` (if using Cloudinary)

---

## Example Usage

### Frontend Integration Pattern
```javascript
// 1. Login and get token
const loginResponse = await fetch('https://kuponna-api.up.railway.app/api/auth/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: 'user@example.com',
    password: 'password'
  })
});
const { token, user } = await loginResponse.json();

// 2. Get user's groups list
const groupsResponse = await fetch('https://kuponna-api.up.railway.app/api/messages/groups', {
  headers: { 'Authorization': `Bearer ${token}` }
});
const { groups } = await groupsResponse.json();
// Returns: Array of ChatGroupMember models with complete relationships

// 3. Connect to Socket.IO
const socket = io('wss://kuponna-api.up.railway.app', {
  auth: { token }
});

// 4. Listen for real-time messages
socket.on('message:receive', (message) => {
  // message is complete ChatMessage model with sender and chatGroup
  console.log('New message from:', message.sender.name);
  console.log('In group:', message.chatGroup.name);
  // Update UI with message.chatGroup.id to know which group
});

// 5. Send a message
socket.emit('message:send', {
  chatGroupId: 'group-uuid',
  content: 'Hello everyone!'
}, (response) => {
  if (response.success) {
    // Message sent successfully
    // response.message contains complete ChatMessage model
  }
});
```

### Complete API Response Examples

#### Login Response
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "user-uuid",
    "name": "John Doe",
    "email": "user@example.com",
    "role": "user",
    "isVerified": true,
    "merchantVerified": false,
    "picture": "https://example.com/avatar.jpg",
    "pushNotification": true,
    "emailNotification": true,
    "available_balance": 1000.50,
    "createdAt": "2025-01-01T00:00:00Z"
  }
}
```

#### Send Message Response
```json
{
  "success": true,
  "message": {
    "id": "msg-uuid",
    "chatGroupId": "group-uuid",
    "senderId": "user-uuid",
    "content": "Hello everyone!",
    "readBy": ["user-uuid"],
    "createdAt": "2025-08-01T10:30:00Z",
    "sender": {
      "id": "user-uuid",
      "name": "John Doe",
      "email": "user@example.com",
      "role": "user",
      "picture": "https://example.com/avatar.jpg"
    },
    "chatGroup": {
      "id": "group-uuid",
      "name": "Family Group",
      "isGroup": true,
      "members": [
        {
          "id": "member1-uuid",
          "userId": "user1-uuid",
          "chatGroupId": "group-uuid",
          "joinedAt": "2025-08-01T09:00:00Z",
          "user": {
            "id": "user1-uuid",
            "name": "Alice Smith",
            "email": "alice@example.com",
            "role": "user",
            "picture": "https://example.com/alice.jpg"
          }
        },
        {
          "id": "member2-uuid",
          "userId": "user2-uuid",
          "chatGroupId": "group-uuid",
          "joinedAt": "2025-08-01T09:00:00Z",
          "user": {
            "id": "user2-uuid",
            "name": "Bob Johnson",
            "email": "bob@example.com",
            "role": "user",
            "picture": "https://example.com/bob.jpg"
          }
        }
      ]
    }
  }
}
```

#### List Groups Response
```json
{
  "success": true,
  "count": 3,
  "groups": [
    {
      "id": "membership-uuid",
      "chatGroupId": "group-uuid",
      "userId": "current-user-uuid",
      "joinedAt": "2025-08-01T09:00:00Z",
      "user": {
        "id": "current-user-uuid",
        "name": "Current User",
        "email": "current@example.com",
        "role": "user",
        "picture": "https://example.com/current.jpg"
      },
      "chatGroup": {
        "id": "group-uuid",
        "name": "Family Group",
        "isGroup": true,
        "memberCount": 3,
        "isUserMember": true,
        "members": [
          // All group members with complete User models
        ],
        "group": {
          // If related to a Group model
          "id": "related-group-uuid",
          "creator": {
            "id": "creator-uuid",
            "name": "Group Creator",
            "email": "creator@example.com"
          }
        }
      }
    }
  ]
}
```

### cURL Examples

#### Login
```bash
curl -X POST https://kuponna-api.up.railway.app/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"user@example.com","password":"password"}'
```

#### Send Message
```bash
curl -X POST https://kuponna-api.up.railway.app/api/messages \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <token>' \
  -d '{"chatGroupId":"group-uuid","content":"Hello!"}'
```

#### Get Messages
```bash
curl -X GET 'https://kuponna-api.up.railway.app/api/messages?chatGroupId=group-uuid' \
  -H 'Authorization: Bearer <token>'
```

#### List Groups
```bash
curl -X GET https://kuponna-api.up.railway.app/api/messages/groups \
  -H 'Authorization: Bearer <token>'
```

#### Create Group
```bash
curl -X POST https://kuponna-api.up.railway.app/api/messages/groups \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <token>' \
  -d '{"name":"New Group","memberIds":["user1-uuid","user2-uuid"]}'
```

---

## License
MIT 