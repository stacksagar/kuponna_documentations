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
- Returns: categories with parent-child relationships
- Response includes `parent` and `children` objects for hierarchical structure

**GET** `/api/categories?id={id}` - Get single category
- Returns: single category with parent-child relationships
- Includes `parent` object (if category has parent) and `children` array (if category has children)

**GET** `/api/categories/grouping` - Get categories grouped by hierarchy (Mobile optimized)
- Purpose: Mobile app friendly endpoint for category selection and navigation
- Returns: Root categories with their children in a structured format
- Response Structure:
  ```json
  {
    "success": true,
    "categories": [
      {
        "id": "uuid",
        "name": "Foods",
        "children": [
          {
            "id": "uuid", 
            "name": "Restaurant"
          },
          {
            "id": "uuid",
            "name": "Fruits"
          }
        ]
      },
      {
        "id": "uuid",
        "name": "Digital",
        "children": [
          {
            "id": "uuid",
            "name": "Course"
          }
        ]
      }
    ]
  }
  ```
- Data Flow Design:
  1. **Root Categories**: Only parent categories (no `parentId`) are returned as main groups
  2. **Children Structure**: Each root category includes all its direct children
  3. **Alphabetical Sorting**: Both root categories and children are sorted alphabetically
  4. **Empty Children**: Root categories with no children will have empty `children` array
  5. **Mobile Optimization**: Simplified structure perfect for dropdown menus, category filters, and navigation
- Use Cases:
  - Category selection in deal creation forms
  - Filter menus in deal browsing
  - Navigation drawers showing category hierarchy
  - Category-based deal organization

### App Settings

**GET** `/api/app/languages` - Get available languages
- `defaultOnly`: `true`, `false` (get only default language)
- `region`: `Africa`, `America` (filter by region)

**GET** `/api/app/countries` - Get available countries
- `defaultOnly`: `true`, `false` (get only default country)
- `withCurrency`: `true`, `false` (include currency information, default: true)
- `region`: `Africa`, `America` (filter by region)
- `continent`: `North America`, `South America`, `Africa` (more specific filtering)

### Reviews

**GET** `/api/reviews?dealId={dealId}` - Get reviews for a deal
- Query Parameters:
  - `dealId`: deal UUID (required)
  - `limit`, `offset`: pagination (optional)
- Returns: array of reviews with user information

**POST** `/api/reviews` - Create review
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "dealId": "string", // REQUIRED: UUID of the deal being reviewed
    "rating": "number", // REQUIRED: 1-5 star rating
    "title": "string", // REQUIRED: review title
    "content": "string" // REQUIRED: review content/description
  }
  ```
- Note: `userId` is set automatically from authentication token
- Returns: created review with user information

---

## Authenticated Endpoints

### Messages

**GET** `/api/user/messages` - Get messages for a chat
- Authentication: Required (JWT token)
- Query Parameters (one required):
  - `chatGroupId`: get messages for group chat (UUID)
  - `recipientId`: get messages for one-on-one chat (UUID)
  - `limit`, `offset`: pagination (optional)
- Returns: array of messages with sender information

**POST** `/api/user/messages` - Send a message
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "content": "string", // REQUIRED: message content
    "chatGroupId": "string", // REQUIRED if not using recipientId (UUID)
    "recipientId": "string", // REQUIRED if not using chatGroupId (UUID)
    "readBy": ["string"] // OPTIONAL: array of user UUIDs who have read the message
  }
  ```
- Note: Must provide either `chatGroupId` OR `recipientId`, not both
- Note: `senderId` is set automatically from authentication token
- Note: If `readBy` is not provided, it defaults to [senderId]
- Returns: created message with sender information

### Message Groups

**GET** `/api/user/messages/groups` - Get user's chat groups
- Authentication: Required (JWT token)
- Returns: array of chat groups user is member of

**POST** `/api/user/messages/groups` - Create group chat
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "name": "string", // REQUIRED: group name
    "memberIds": ["string"] // REQUIRED: array of user UUIDs to add to group
  }
  ```
- Note: Creator is automatically added as group member
- Returns: created group with member information

### Group Members

**POST** `/api/user/messages/groups/{groupId}/members` - Add member to group
- Authentication: Required (JWT token)
- Path Parameters:
  - `groupId`: group UUID (required)
- Request Body:
  ```json
  {
    "userId": "string" // REQUIRED: UUID of user to add to group
  }
  ```
- Returns: updated group member information

**DELETE** `/api/user/messages/groups/{groupId}/members` - Remove member from group
- Authentication: Required (JWT token)
- Path Parameters:
  - `groupId`: group UUID (required)
- Query Parameters:
  - `userId`: UUID of user to remove (required)
- Returns: success confirmation

### Cart Management

**GET** `/api/user/cart` - Get user's cart items with totals
- Authentication: Required (JWT token)
- Returns: 
  ```json
  {
    "items": [Cart], // array of cart items with deal details
    "summary": {
      "totalItems": "number",
      "totalAmount": "number"
    }
  }
  ```

**POST** `/api/user/cart` - Add deal to cart
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "dealId": "string", // REQUIRED: UUID of the deal
    "note": "string" // OPTIONAL: additional notes
  }
  ```
- Note: `userId` is set automatically from authentication token
- Note: Prevents duplicate entries (same user + same deal)
- Returns: created cart item with deal details

**PUT** `/api/user/cart/{itemId}` - Update cart item
- Authentication: Required (JWT token)
- Path Parameters:
  - `itemId`: cart item UUID (required)
- Request Body:
  ```json
  {
    "note": "string" // OPTIONAL: update notes for the cart item
  }
  ```
- Note: Users can only update their own cart items
- Returns: updated cart item

**DELETE** `/api/user/cart/{itemId}` - Remove item from cart
- Authentication: Required (JWT token)
- Path Parameters:
  - `itemId`: cart item UUID (required)
- Note: Users can only delete their own cart items
- Returns: success confirmation

### Notifications

**POST** `/api/user/device-token` - Register device for push notifications
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "deviceToken": "string", // REQUIRED: The push token from the device
    "deviceType": "android|ios|web" // OPTIONAL: Device platform
  }
  ```
- Returns: `{ success: true }` on success

**GET** `/api/user/notifications` - Get user notifications
- Authentication: Required (JWT token)
- Returns:
  ```json
  {
    "items": [
      {
        "id": "string",
        "type": "system|deal|order|refund|complaint",
        "title": "string",
        "message": "string",
        "read": false,
        "url": "string",
        "createdAt": "2024-07-30T12:00:00Z"
      }
    ]
  }
  ```

**PUT** `/api/user/notifications/{id}/read` - Mark notification as read
- Authentication: Required (JWT token)
- Path Parameters:
  - `id`: notification UUID (required)
- Returns: `{ success: true }` on success

### User Preferences

**GET** `/api/user/preferences` - Get user preferences
- Authentication: Required (JWT token)
- Returns: complete user preferences object
- Response:
  ```json
  {
    "success": true,
    "data": {
      "theme": "dark",
      "language": "en",
      "currency": "NGN",
      "customSetting": "value"
    },
    "message": "User preferences retrieved successfully"
  }
  ```

**PUT** `/api/user/preferences` - Update user preferences
- Authentication: Required (JWT token)
- Request Body: Any valid JSON object with preference key-value pairs
  ```json
  {
    "theme": "dark",
    "language": "fr",
    "currency": "USD",
    "notifications": {
      "email": true,
      "push": false
    },
    "customPreference": "any value"
  }
  ```
- Note: Merges with existing preferences (preserves existing data)
- Note: Any JSON structure is accepted - completely flexible
- Returns: updated complete preferences object

### Notification Settings

**GET** `/api/user/notifications/settings` - Get notification preferences
- Authentication: Required (JWT token)
- Returns: all notification settings (email, push, SMS, marketing, etc.)

**PUT** `/api/user/notifications/settings` - Update notification preferences
- Authentication: Required (JWT token)
- Request Body (all fields optional):
  ```json
  {
    "emailNotifications": "boolean", // OPTIONAL: enable/disable email notifications
    "pushNotifications": "boolean", // OPTIONAL: enable/disable push notifications
    "smsNotifications": "boolean", // OPTIONAL: enable/disable SMS notifications
    "marketingEmails": "boolean", // OPTIONAL: marketing email preferences
    "dealAlerts": "boolean", // OPTIONAL: deal alert notifications
    "priceDropAlerts": "boolean", // OPTIONAL: price drop notifications
    "groupInvitations": "boolean", // OPTIONAL: group invitation notifications
    "orderUpdates": "boolean", // OPTIONAL: order status updates
    "newMessages": "boolean", // OPTIONAL: new message notifications
    "weeklyDeals": "boolean", // OPTIONAL: weekly deals digest
    "flashSales": "boolean", // OPTIONAL: flash sale notifications
    "reviewReminders": "boolean", // OPTIONAL: review reminder notifications
    "accountSecurity": "boolean" // OPTIONAL: security-related notifications
  }
  ```
- Note: Only provided fields will be updated
- Returns: updated notification settings

### File Upload

### File Upload

**POST** `/api/upload` - Upload files to cloud storage
- Authentication: Required (JWT token)
- Content-Type: `multipart/form-data`
- Request Body:
  ```
  file: [File] (required) - The file to upload
  category: [String] (optional) - File category (defaults to "general")
  ```
- File Limitations:
  - **Maximum Size**: 50MB for all file types
  - **Supported Types**: All file types supported (images, documents, videos, etc.)
  - **Auto-Detection**: File type and optimization handled automatically by Cloudinary
- Technical Details:
  - Uses direct Cloudinary API for reliable uploads
  - Automatic endpoint selection based on file type:
    - Images: `/image/upload` endpoint
    - Videos: `/video/upload` endpoint  
    - Documents/Others: `/raw/upload` endpoint
  - Files are organized in folders: `kuponna/{userRole}/{userId}/{category}/`
  - Unique filenames generated: `{userId}_{category}_{timestamp}`
- Returns:
  ```json
  {
    "success": true,
    "message": "File uploaded successfully",
    "data": {
      "id": "string", // Cloudinary public_id
      "url": "string", // Direct file URL
      "urls": {
        "original": "string", // Original file URL
        "thumbnail": "string|null", // Thumbnail URL (images only) - 300x300px
        "medium": "string|null" // Medium size URL (images only) - 800x600px
      },
      "originalName": "string", // Original filename
      "size": "number", // File size in bytes
      "format": "string", // File format (jpg, pdf, mp4, etc.)
      "width": "number|null", // Width for images/videos
      "height": "number|null", // Height for images/videos
      "category": "string", // File category
      "resourceType": "string", // Cloudinary resource type (image, video, raw)
      "mimeType": "string", // File MIME type
      "uploadedAt": "string", // ISO timestamp
      "uploadedBy": {
        "userId": "string",
        "userRole": "string",
        "userName": "string"
      },
      "cloudinary": {
        "public_id": "string", // Use this for deletions
        "version": "number",
        "etag": "string",
        "folder": "string"
      }
    },
    "meta": {
      "uploadProgress": 100, // Always 100% when upload completes
      "processingTime": "string", // Upload processing time
      "canDelete": true,
      "canUpdate": "boolean" // Based on user role and category
    }
  }
  ```

**GET** `/api/upload` - Get user's uploaded files
- Authentication: Required (JWT token)
- Query Parameters:
  - `category`: string (optional) - filter by category
  - `limit`: number (optional) - results per page (max 50, default 20)
  - `page`: number (optional) - page number (default 1)
- Returns:
  ```json
  {
    "success": true,
    "data": {
      "files": [
        {
          "id": "string", // Cloudinary public_id
          "url": "string", // File URL
          "originalName": "string", // Original filename
          "size": "number", // File size in bytes
          "format": "string", // File format
          "width": "number|null", // Width for images/videos
          "height": "number|null", // Height for images/videos
          "resourceType": "string", // Cloudinary resource type
          "createdAt": "string", // ISO timestamp
          "folder": "string", // Cloudinary folder path
          "category": "string" // File category
        }
      ],
      "pagination": {
        "page": "number",
        "limit": "number", 
        "total": "number",
        "hasMore": "boolean"
      }
    }
  }
  ```

**DELETE** `/api/upload` - Delete uploaded file
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "publicId": "string" // REQUIRED: Cloudinary public_id from upload response
  }
  ```
- Note: Users can only delete their own files
- Note: Use the `public_id` from the upload response or file listing
- Returns:
  ```json
  {
    "success": true,
    "message": "File deleted successfully",
    "data": {
      "publicId": "string",
      "deletedAt": "string" // ISO timestamp
    }
  }
  ```

#### Mobile Development Examples

**React Native Example:**
```javascript
const uploadFile = async (fileUri, authToken, category = 'general') => {
  const formData = new FormData();
  formData.append('file', {
    uri: fileUri,
    type: 'image/jpeg', // or detect from file
    name: 'upload.jpg'
  });
  formData.append('category', category);

  try {
    const response = await fetch('https://kuponna.up.railway.app/api/upload', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${authToken}`,
        'Content-Type': 'multipart/form-data',
      },
      body: formData,
    });

    const result = await response.json();
    if (result.success) {
      console.log('Upload successful:', result.data.url);
      return result.data;
    } else {
      throw new Error(result.details);
    }
  } catch (error) {
    console.error('Upload failed:', error);
    throw error;
  }
};
```

**Android (Kotlin) Example:**
```kotlin
fun uploadFile(file: File, authToken: String, category: String = "general") {
    val requestBody = MultipartBody.Builder()
        .setType(MultipartBody.FORM)
        .addFormDataPart("file", file.name, RequestBody.create(MediaType.parse("*/*"), file))
        .addFormDataPart("category", category)
        .build()

    val request = Request.Builder()
        .url("https://kuponna.up.railway.app/api/upload")
        .header("Authorization", "Bearer $authToken")
        .post(requestBody)
        .build()

    client.newCall(request).enqueue(object : Callback {
        override fun onResponse(call: Call, response: Response) {
            val responseBody = response.body()?.string()
            // Handle success response
        }
        
        override fun onFailure(call: Call, e: IOException) {
            // Handle error
        }
    })
}
```

**iOS (Swift) Example:**
```swift
func uploadFile(fileData: Data, authToken: String, category: String = "general") {
    guard let url = URL(string: "https://kuponna.up.railway.app/api/upload") else { return }
    
    var request = URLRequest(url: url)
    request.httpMethod = "POST"
    request.setValue("Bearer \(authToken)", forHTTPHeaderField: "Authorization")
    
    let boundary = UUID().uuidString
    request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
    
    var body = Data()
    body.append("--\(boundary)\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"file\"; filename=\"upload.jpg\"\n".data(using: .utf8)!)
    body.append("Content-Type: image/jpeg\n\n".data(using: .utf8)!)
    body.append(fileData)
    body.append("\n--\(boundary)\n".data(using: .utf8)!)
    body.append("Content-Disposition: form-data; name=\"category\"\n\n".data(using: .utf8)!)
    body.append(category.data(using: .utf8)!)
    body.append("\n--\(boundary)--\n".data(using: .utf8)!)
    
    request.httpBody = body
    
    URLSession.shared.dataTask(with: request) { data, response, error in
        // Handle response
    }.resume()
}
```

**Flutter Example:**
```dart
import 'package:http/http.dart' as http;
import 'dart:io';

Future<Map<String, dynamic>> uploadFile(File file, String authToken, {String category = 'general'}) async {
  final uri = Uri.parse('https://kuponna.up.railway.app/api/upload');
  final request = http.MultipartRequest('POST', uri);
  
  request.headers['Authorization'] = 'Bearer $authToken';
  request.fields['category'] = category;
  request.files.add(await http.MultipartFile.fromPath('file', file.path));
  
  try {
    final response = await request.send();
    final responseBody = await response.stream.bytesToString();
    final result = json.decode(responseBody);
    
    if (result['success']) {
      return result['data'];
    } else {
      throw Exception(result['details']);
    }
  } catch (e) {
    print('Upload failed: $e');
    rethrow;
  }
}
```


### Payment Methods

**GET** `/api/user/payment-methods` - Get saved payment methods
- Authentication: Required (JWT token)
- Returns: user's payment methods grouped by type (cards, bank accounts, wallets, mobile money)
- Includes default method identification

**POST** `/api/user/payment-methods` - Add payment method
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "type": "card|bank_transfer|wallet|mobile_money", // REQUIRED
    "provider": "paystack|flutterwave|paypal|stripe|bank|mtn|airtel|glo|9mobile", // REQUIRED
    "accountName": "string", // OPTIONAL: account holder name
    "accountNumber": "string", // OPTIONAL: account/card number
    "bankName": "string", // OPTIONAL: bank name (for bank transfers)
    "bankCode": "string", // OPTIONAL: bank code
    "cardLastFour": "string", // OPTIONAL: last 4 digits of card
    "cardBrand": "string", // OPTIONAL: visa, mastercard, etc.
    "cardExpiry": "string", // OPTIONAL: MM/YY format
    "nickname": "string", // OPTIONAL: user-friendly name
    "isDefault": "boolean", // OPTIONAL: set as default method
    "metadata": {} // OPTIONAL: additional provider-specific data
  }
  ```
- Note: `userId` is set automatically from authentication token
- Note: `isActive` is automatically set to true
- Note: If `isDefault` is true, other methods are automatically set to non-default
- Returns: created payment method

**DELETE** `/api/user/payment-methods/{id}` - Remove payment method
- Authentication: Required (JWT token)
- Path Parameters:
  - `id`: payment method UUID (required)
- Note: Users can only delete their own payment methods
- Note: Automatically sets another method as default if removed method was default
- Returns: success confirmation

### Transaction History
**GET** `/api/user/transaction` - Transaction history
- Authentication: Required (JWT token)
- Query Parameters (all optional):
  - `status`: `pending`, `completed`, `failed`, `refunded` (filter by status)
  - `method`: `card`, `bank_transfer`, `wallet`, `cash` (filter by payment method)
  - `startDate`: date in YYYY-MM-DD format (filter from date)
  - `endDate`: date in YYYY-MM-DD format (filter to date)
  - `limit`, `offset`: pagination
- Returns: transactions array with deal details, merchant info, and summary statistics

### Receipts

**GET** `/api/user/receipts/{transactionId}` - Get payment receipt
- Authentication: Required (JWT token)
- Path Parameters:
  - `transactionId`: transaction UUID (required)
- Returns: detailed receipt with transaction, customer, deal, merchant, and group information
- Includes receipt number and generation metadata
- Note: Users can only access receipts for their own transactions

### Support Tickets

**GET** `/api/support/tickets` - Get user's support tickets
- Query Parameters:
  - `status`: `open`, `in_progress`, `resolved`, `closed` (optional)
  - `category`: `technical`, `billing`, `account`, `deals`, `payments`, `refunds`, `general`, `bug_report`, `feature_request` (optional)
  - `priority`: `low`, `medium`, `high`, `urgent` (optional)
  - `limit`, `offset`: pagination (optional)
- Returns: tickets array with summary statistics and user/assignee information

**POST** `/api/support/tickets` - Create support ticket
- Authentication: Required (JWT token)
- Request Body:
  ```json
  {
    "subject": "string", // REQUIRED: 5-200 characters
    "description": "string", // REQUIRED: 10-5000 characters
    "category": "technical|billing|account|deals|payments|refunds|general|bug_report|feature_request", // OPTIONAL: defaults to "general"
    "priority": "low|medium|high|urgent", // OPTIONAL: defaults to "medium"
    "assignedToId": "string", // OPTIONAL: UUID of admin user to assign (admin only)
    "tags": ["string"], // OPTIONAL: array of tags
    "attachments": ["string"], // OPTIONAL: array of attachment URLs
    "metadata": {}, // OPTIONAL: additional data
    "isPublic": "boolean", // OPTIONAL: defaults to true
    "customerSatisfactionRating": "number", // OPTIONAL: 1-5 rating
    "customerFeedback": "string", // OPTIONAL: customer feedback text
    "internalNotes": "string" // OPTIONAL: admin-only notes (admin only)
  }
  ```
- Note: `userId` is automatically set from authentication token
- Note: `ticketNumber` is auto-generated by the system
- Note: `status` is automatically set to "open"
- Note: Some fields like `assignedToId` and `internalNotes` may require admin privileges
- Returns: created ticket with user information

**GET** `/api/support/tickets/{id}` - Get ticket details
- Path Parameters:
  - `id`: ticket UUID (required)
- Returns: detailed ticket information with computed fields (isOverdue, responseTime)
- Note: Users can only access their own tickets

**POST** `/api/support/tickets/{id}/messages` - Add message to ticket
- Path Parameters:
  - `id`: ticket UUID (required)
- Request Body:
  ```json
  {
    "message": "string", // REQUIRED: 1-5000 characters
    "attachments": ["string"], // OPTIONAL: array of attachment URLs
    "metadata": {} // OPTIONAL: additional data
  }
  ```
- Note: `userId` and `ticketId` are set automatically
- Note: `type` is automatically set to "user" for customer messages
- Note: `isInternal` is automatically set to false for customer messages
- Returns: created message with user information
- Automatically updates ticket status and lastResponseAt timestamp

**GET** `/api/support/tickets/{id}/messages` - Get ticket messages
- Path Parameters:
  - `id`: ticket UUID (required)
- Query Parameters:
  - `limit`: number of messages (optional, default: 50)
  - `offset`: pagination offset (optional, default: 0)
- Returns: messages array with ticket summary and pagination info
- Note: Only shows public messages (isInternal: false) to users

---

## Data Models

```typescript
interface Deal {
  id: string;
  merchantId: string;
  title: string;
  description?: string;
  pricePerPerson?: number;
  discountPrice?: number;
  deliveryFee?: number;
  discountType?: "percentage" | "fixed";
  discountPercentage?: number;
  requiredMembers: number;
  soldCount?: number;
  views?: number;
  startDate?: string;
  endDate?: string;
  redemptionExpired?: string;
  redemptionLimit?: number;
  videoUrl?: string;
  tags?: string[];
  status?: "draft" | "pending" | "active" | "completed" | "cancelled";
  location?: string;
  icon?: string;
  thumbnail?: string;
  isFeatured?: boolean;
  amenities?: string[];
  type?: "physical" | "digital";
  requiresRedemptionCard?: boolean;
  createdAt: string;
  updatedAt: string;
  // Computed fields
  currentMembers?: number;
  averageRating?: number;
  reviewCount?: number;
  images?: string[];
  merchant?: {
    id: string;
    name: string;
    email: string;
    picture?: string;
  };
}

interface User {
  id: string;
  email: string;
  password: string; // Only for internal use, never exposed in API responses
  role: "admin" | "user" | "merchant";
  name: string;
  phoneNumber: string;
  picture?: string;
  isVerified: boolean;
  merchantVerified: boolean;
  lastLogin?: string;
  merchantRole?: string;
  country?: string;
  timezone?: string;
  bio?: string;
  pushNotification: boolean;
  emailNotification: boolean;
  smsNotification: boolean;
  available_balance: number;
  book_balance: number;
  monthly_target: number;
  refund_balance: number;
  categories: string[];
  preferences?: any;
  createdAt: string;
  updatedAt: string;
}

interface Category {
  id: string;
  name: string;
  parentId?: string;
  dealCount?: number;
  createdAt: string;
  updatedAt: string;
  // Relationship objects
  parent?: {
    id: string;
    name: string;
  };
  children?: Array<{
    id: string;
    name: string;
  }>;
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

interface Cart {
  id: string;
  userId: string;
  dealId: string;
  note?: string;
  createdAt: string;
  deal: {
    id: string;
    title: string;
    description: string;
    pricePerPerson: number;
    discountPrice: number;
    images: string[];
    merchant: {
      id: string;
      name: string;
      email: string;
      picture?: string;
    };
  };
}

interface Language {
  code: string;
  name: string;
  nativeName: string;
  flag: string;
  region: string;
  isDefault: boolean;
}

interface Country {
  code: string;
  name: string;
  currency?: string;
  currencySymbol?: string;
  flag: string;
  phoneCode: string;
  region: string;
  continent: string;
  isDefault: boolean;
}

interface NotificationSettings {
  emailNotifications: boolean;
  pushNotifications: boolean;
  smsNotifications: boolean;
  marketingEmails: boolean;
  dealAlerts: boolean;
  priceDropAlerts: boolean;
  groupInvitations: boolean;
  orderUpdates: boolean;
  newMessages: boolean;
  weeklyDeals: boolean;
  flashSales: boolean;
  reviewReminders: boolean;
  accountSecurity: boolean;
}

interface PaymentMethod {
  id: string;
  userId: string;
  type: "card" | "bank_transfer" | "wallet" | "mobile_money";
  provider: "paystack" | "flutterwave" | "paypal" | "stripe" | "bank" | "mtn" | "airtel" | "glo" | "9mobile";
  accountName?: string;
  accountNumber?: string;
  bankName?: string;
  bankCode?: string;
  cardLastFour?: string;
  cardBrand?: string;
  cardExpiry?: string;
  nickname?: string;
  isDefault: boolean;
  isActive: boolean;
  metadata?: any;
  createdAt: string;
  updatedAt: string;
}

interface Transaction {
  id: string;
  groupId: number;
  orderId: number;
  userId: number;
  amount: number;
  method: "card" | "bank_transfer" | "cash" | "wallet";
  status: "pending" | "completed" | "failed" | "refunded";
  transactionId?: string;
  paidAt?: string;
  createdAt: string;
  updatedAt: string;
  // Relationship data
  group?: {
    id: string;
    status: string;
    maxGroupSize?: number;
    currentMembers?: number;
  };
  order?: {
    id: string;
    deal: {
      id: string;
      title: string;
      pricePerPerson: number;
      images: string[];
      merchant: {
        id: string;
        name: string;
        email: string;
      };
    };
  };
  user?: {
    id: string;
    name: string;
    email: string;
  };
}
  group?: {
    id: string;
    status: string;
    maxGroupSize?: number;
    currentMembers?: number;
  };
}

interface Receipt {
  transaction: {
    id: string;
    transactionId: string;
    amount: number;
    method: string;
    status: string;
    paidAt?: string;
    createdAt: string;
  };
  customer: {
    name: string;
    email: string;
    phone?: string;
  };
  deal?: {
    id: string;
    title: string;
    description: string;
    pricePerPerson: number;
    discountPrice: number;
    category: string;
    images: string[];
    merchant: {
      name: string;
      email: string;
      phone?: string;
    };
  };
  group?: {
    id: string;
    status: string;
    maxGroupSize?: number;
    currentMembers?: number;
  };
  order?: {
    id: string;
    quantity: number;
    totalAmount: number;
  };
  receiptNumber: string;
  generatedAt: string;
  platform: string;
}

interface SupportTicket {
  id: string;
  ticketNumber: string;
  userId: string;
  subject: string;
  description: string;
  category: 'technical' | 'billing' | 'account' | 'deals' | 'payments' | 'refunds' | 'general' | 'bug_report' | 'feature_request';
  priority: 'low' | 'medium' | 'high' | 'urgent';
  status: 'open' | 'in_progress' | 'resolved' | 'closed';
  assignedToId?: string;
  tags?: string[];
  attachments?: string[];
  metadata?: any;
  lastResponseAt?: string;
  resolvedAt?: string;
  closedAt?: string;
  isPublic: boolean;
  customerSatisfactionRating?: number;
  customerFeedback?: string;
  internalNotes?: string;
  createdAt: string;
  updatedAt: string;
  // Virtual fields
  isOverdue?: boolean;
  responseTime?: number;
  user?: {
    id: string;
    name: string;
    email: string;
    picture?: string;
  };
  assignedTo?: {
    id: string;
    name: string;
    email: string;
    picture?: string;
  };
  messages?: SupportMessage[];
}

interface SupportMessage {
  id: string;
  ticketId: string;
  userId: string;
  message: string;
  type: 'user' | 'admin' | 'system';
  attachments?: string[];
  isInternal: boolean;
  metadata?: any;
  createdAt: string;
  updatedAt: string;
  user?: {
    id: string;
    name: string;
    email: string;
    picture?: string;
  };
}
```
