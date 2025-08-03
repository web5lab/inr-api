# INR Payment Iframe API Documentation

This document provides comprehensive API documentation for integrating with the INR Payment system. You can use these APIs to build your own custom UI instead of using the provided iframe.

**Note**: The system now uses webhook callbacks to notify your application about payment events. See the [Webhook Integration](#webhook-integration) section for details.

## Base URL
```
https://inrapi.web5lab.net/iframe
```

## Authentication
No authentication required for iframe APIs. All endpoints are public.

---

## 📋 Table of Contents
1. [Payment Methods](#payment-methods)
2. [Deposits](#deposits)
3. [Withdrawals](#withdrawals)
4. [Transactions](#transactions)
5. [Webhook Integration](#webhook-integration)

---

## 🏦 Payment Methods

### Get Available Payment Methods
Check which payment methods are available for a specific amount and type.

**Endpoint:** `GET /available-payment-methods`

**Query Parameters:**
- `amount` (required): The payment amount (number)
- `type` (required): Payment method type (`UPI` or `Bank`)

**Example Request:**
```bash
curl "https://your-server.com/iframe/available-payment-methods?amount=5000&type=UPI"
```

**Success Response (200):**
```json
{
  "success": true,
  "available": true,
  "message": "2 UPI payment method(s) available",
  "methods": [
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d0",
      "type": "UPI",
      "name": "PhonePe Business",
      "identifier": "business@phonepe",
      "status": "active",
      "dailyLimit": 100000,
      "monthlyLimit": 3000000,
      "qrCode": "https://example.com/qr-code.png"
    }
  ],
  "totalMethods": 2
}
```

**Not Available Response (200):**
```json
{
  "success": false,
  "available": false,
  "message": "No UPI payment methods available for amount ₹50,000. Maximum available limit exceeded.",
  "methods": [],
  "maxLimit": 25000
}
```

### Get All Active Payment Methods
Get all currently active payment methods.

**Endpoint:** `GET /payment-methods`

**Example Request:**
```bash
curl "https://your-server.com/iframe/payment-methods"
```

**Response (200):**
```json
{
  "success": true,
  "methods": [
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d0",
      "type": "UPI",
      "name": "PhonePe Business",
      "identifier": "business@phonepe",
      "status": "active",
      "dailyLimit": 100000,
      "monthlyLimit": 3000000,
      "qrCode": "https://example.com/qr-code.png"
    },
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d1",
      "type": "Bank",
      "name": "HDFC Business Account",
      "identifier": "1234567890",
      "status": "active",
      "dailyLimit": 500000,
      "monthlyLimit": 15000000,
      "accountHolderName": "INR Payment Solutions Pvt Ltd",
      "ifscCode": "HDFC0001234",
      "bankName": "HDFC Bank",
      "branchName": "Mumbai Central"
    }
  ]
}
```

---

## 💰 Deposits

### Create Deposit Request
Submit a new deposit request with optional payment screenshot.

**Endpoint:** `POST /deposits`

**Content-Type:** `multipart/form-data`

**Form Fields:**
- `amount` (required): Deposit amount (number)
- `paymentMethod` (required): Payment method name (string)
- `transactionId` (required): Transaction ID from payment (string)
- `screenshot` (optional): Payment screenshot (file - image only, max 5MB)

**Example Request:**
```bash
curl -X POST "https://your-server.com/iframe/deposits" \
  -F "amount=5000" \
  -F "paymentMethod=PhonePe Business" \
  -F "transactionId=TXN123456789" \
  -F "uid=UID123456789" \
  -F "screenshot=@payment_screenshot.jpg"
```

**Success Response (201):**
```json
{
  "success": true,
  "message": "Deposit request submitted",
  "deposit": {
    "_id": "64f8a1b2c3d4e5f6a7b8c9d2",
    "amount": 5000,
    "paymentMethod": "PhonePe Business",
    "customerName": "Shiva",
    "uid":"uid123",
    "customerEmail": "helloshiva0801@gmail.com",
    "customerPhone": "1234567890",
    "transactionId": "TXN123456789",
    "status": "pending",
    "screenshot": "https://your-server.com/uploads/screenshots/screenshot-123456.jpg",
    "timestamp": "2024-01-20T10:30:00.000Z",
    "createdAt": "2024-01-20T10:30:00.000Z"
  }
}
```

**Error Response (400):**
```json
{
  "error": "Missing required fields"
}
```

**Duplicate Transaction Error (400):**
```json
{
  "error": "Transaction ID already exists",
  "details": "This transaction ID has already been used"
}
```

### Get User Deposits
Retrieve deposits for a specific user.

**Endpoint:** `GET /deposits`

**Query Parameters:**
- `email` (optional): User email
- `phone` (optional): User phone number

*Note: At least one of email or phone is required*

**Example Request:**
```bash
curl "https://your-server.com/iframe/deposits?email=user@example.com"
```

**Response (200):**
```json
{
  "success": true,
  "deposits": [
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d2",
      "amount": 5000,
      "paymentMethod": "PhonePe Business",
      "customerName": "Shiva",
      "uid":"UID123"
      "customerEmail": "helloshiva0801@gmail.com",
      "customerPhone": "1234567890",
      "transactionId": "TXN123456789",
      "status": "approved",
      "screenshot": "https://your-server.com/uploads/screenshots/screenshot-123456.jpg",
      "timestamp": "2024-01-20T10:30:00.000Z",
      "approvedBy": "Admin User",
      "approvedAt": "2024-01-20T11:00:00.000Z",
      "remarks": "Verified and approved"
    }
  ]
}
```

---

## 💸 Withdrawals

### Create Withdrawal Request
Submit a new withdrawal request.

**Endpoint:** `POST /withdrawals`

**Content-Type:** `application/json`

**Request Body:**
```json
{
  "amount": 2000,
  "customerName": "John Doe",
  "customerEmail": "john@example.com",
  "customerPhone": "+91 9876543210",
  "withdrawalMethod": "UPI",
  "uid":"uid123",
  "accountDetails": {
    "upiId": "john@paytm"
  }
}
```

**For Bank Transfer:**
```json
{
  "amount": 5000,
  "customerName": "John Doe",
  "customerEmail": "john@example.com",
  "customerPhone": "+91 9876543210",
  "withdrawalMethod": "Bank Transfer",
  "uid":"uid123",
  "accountDetails": {
    "accountNumber": "1234567890",
    "ifscCode": "HDFC0001234",
    "accountHolderName": "John Doe",
    "bankName": "HDFC Bank"
  }
}
```

**Success Response (201):**
```json
{
  "success": true,
  "message": "Withdrawal request submitted successfully",
  "withdrawal": {
    "_id": "64f8a1b2c3d4e5f6a7b8c9d3",
    "amount": 2000,
    "customerName": "John Doe",
    "customerEmail": "john@example.com",
    "customerPhone": "+91 9876543210",
    "withdrawalMethod": "UPI",
    "uid":"uid123",
    "accountDetails": {
      "upiId": "john@paytm"
    },
    "status": "pending",
    "requestId": "WR1642678901ABCDE",
    "createdAt": "2024-01-20T12:00:00.000Z"
  }
}
```

### Get User Withdrawals
Retrieve withdrawals for a specific user.

**Endpoint:** `GET /withdrawals`

**Query Parameters:**
- `email` (optional): User email
- `phone` (optional): User phone number

*Note: At least one of email or phone is required*

**Example Request:**
```bash
curl "https://your-server.com/iframe/withdrawals?email=john@example.com"
```

**Response (200):**
```json
{
  "success": true,
  "withdrawals": [
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d3",
      "amount": 2000,
      "customerName": "John Doe",
      "customerEmail": "john@example.com",
      "customerPhone": "+91 9876543210",
      "withdrawalMethod": "UPI",
      "uid":"uid123",
      "accountDetails": {
        "upiId": "john@paytm"
      },
      "status": "approved",
      "requestId": "WR1642678901ABCDE",
      "transactionId": "TXN987654321",
      "processedBy": "Admin User",
      "processedAt": "2024-01-20T13:00:00.000Z",
      "remarks": "Processed successfully",
      "createdAt": "2024-01-20T12:00:00.000Z"
    }
  ]
}
```

---

## 📊 Transactions

### Get User Transactions
Retrieve all transactions (deposits + withdrawals) for a specific user.

**Endpoint:** `GET /transactions`

**Query Parameters:**
- `email` (optional): User email
- `phone` (optional): User phone number
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `type` (optional): Transaction type (`deposit` or `withdrawal`)

*Note: At least one of email or phone is required*

**Example Request:**
```bash
curl "https://your-server.com/iframe/transactions?email=user@example.com&page=1&limit=10&type=deposit"
```

**Response (200):**
```json
{
  "success": true,
  "transactions": [
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d4",
      "transactionId": "TXN123456789",
      "type": "deposit",
      "amount": 5000,
      "status": "completed",
      "customerName": "Shiva",
      "customerEmail": "helloshiva0801@gmail.com",
      "customerPhone": "1234567890",
      "paymentMethod": "PhonePe Business",
      "screenshot": "https://your-server.com/uploads/screenshots/screenshot-123456.jpg",
      "processedBy": "Admin User",
      "processedAt": "2024-01-20T11:00:00.000Z",
      "remarks": "Verified and approved",
      "createdAt": "2024-01-20T10:30:00.000Z"
    },
    {
      "_id": "64f8a1b2c3d4e5f6a7b8c9d5",
      "transactionId": "WR1642678901ABCDE",
      "type": "withdrawal",
      "amount": 2000,
      "status": "pending",
      "customerName": "Shiva",
      "customerEmail": "helloshiva0801@gmail.com",
      "customerPhone": "1234567890",
      "paymentMethod": "UPI",
      "requestId": "WR1642678901ABCDE",
      "accountDetails": {
        "upiId": "user@paytm"
      },
      "createdAt": "2024-01-20T12:00:00.000Z"
    }
  ],
  "pagination": {
    "totalCount": 25,
    "page": 1,
    "limit": 10,
    "totalPages": 3
  }
}
```

---

## 🔄 Status Values

### Deposit/Withdrawal Status:
- `pending`: Awaiting admin review
- `approved`: Approved by admin (deposits)
- `rejected`: Rejected by admin

### Transaction Status:
- `pending`: Awaiting processing
- `completed`: Successfully processed
- `rejected`: Rejected by admin
- `failed`: Processing failed

---

## 🎨 Frontend Integration Examples

### React Example - Check Available Methods:
```javascript
const checkAvailableMethods = async (amount, type) => {
  try {
    const response = await fetch(
      `${API_BASE_URL}/iframe/available-payment-methods?amount=${amount}&type=${type}`
    );
    const data = await response.json();
    
    if (data.available) {
      setPaymentMethods(data.methods);
    } else {
      setError(data.message);
      setMaxLimit(data.maxLimit);
    }
  } catch (error) {
    console.error('Error checking methods:', error);
  }
};
```

### React Example - Create Deposit:
```javascript
const createDeposit = async (depositData) => {
  const formData = new FormData();
  formData.append('amount', depositData.amount);
  formData.append('paymentMethod', depositData.paymentMethod);
  formData.append('transactionId', depositData.transactionId);
  
  if (depositData.screenshot) {
    formData.append('screenshot', depositData.screenshot);
  }
  
  try {
    const response = await fetch(`${API_BASE_URL}/iframe/deposits`, {
      method: 'POST',
      body: formData
    });
    
    const result = await response.json();
    
    if (result.success) {
      console.log('Deposit created:', result.deposit);
    } else {
      console.error('Error:', result.error);
    }
  } catch (error) {
    console.error('Network error:', error);
  }
};
```

### React Example - Get Transactions:
```javascript
const fetchTransactions = async (email, phone, filters = {}) => {
  const params = new URLSearchParams();
  if (email) params.append('email', email);
  if (phone) params.append('phone', phone);
  if (filters.type) params.append('type', filters.type);
  if (filters.page) params.append('page', filters.page);
  
  try {
    const response = await fetch(
      `${API_BASE_URL}/iframe/transactions?${params}`
    );
    const data = await response.json();
    
    if (data.success) {
      setTransactions(data.transactions);
      setPagination(data.pagination);
    }
  } catch (error) {
    console.error('Error fetching transactions:', error);
  }
};
```

---

## 🚨 Error Handling

### Common Error Responses:

**400 Bad Request:**
```json
{
  "error": "Missing required fields"
}
```

**400 Duplicate Transaction:**
```json
{
  "error": "Transaction ID already exists",
  "details": "This transaction ID has already been used"
}
```

**500 Server Error:**
```json
{
  "error": "Server error",
  "details": "Internal server error message"
}
```

---

## 📝 Notes

1. **File Uploads**: Only image files are accepted for screenshots (max 5MB)
2. **Transaction IDs**: Must be unique across all deposits
3. **User Data**: Currently uses hardcoded user data for iframe transactions
4. **Rate Limiting**: API has rate limiting (10,000 requests per 15 minutes per IP)
5. **CORS**: All iframe APIs support CORS for cross-origin requests

---

## 🔧 Development Tips

1. **Testing**: Use tools like Postman or curl to test API endpoints
2. **Error Handling**: Always implement proper error handling in your frontend
3. **Loading States**: Show loading indicators during API calls
4. **Validation**: Validate data on frontend before sending to API
5. **File Size**: Check file size before uploading screenshots

---

## 📞 Support

For technical support or questions about the API, please contact the development team or refer to the main project documentation.

---

## 🔗 Webhook Integration

The INR Payment system automatically sends webhook notifications to your configured callback URL for all payment events. This allows your application to receive real-time updates about payment status changes.

### Setting Up Webhooks

1. **Configure Callback URL**: In the admin dashboard, go to "Callback Config" and enter your webhook endpoint URL
2. **Set Secret Key**: Configure a secret key for webhook signature verification
3. **Choose Events**: Select which events you want to receive notifications for
4. **Test Endpoint**: Use the test function to verify your webhook endpoint is working

### Webhook Events

The system sends webhooks for these events:

| Event | Description | Triggered When |
|-------|-------------|----------------|
| `depositCreated` | New deposit request | User submits deposit via iframe |
| `depositApproved` | Deposit approved | Admin approves deposit request |
| `depositRejected` | Deposit rejected | Admin rejects deposit request |
| `withdrawalCreated` | New withdrawal request | User submits withdrawal via iframe |
| `withdrawalApproved` | Withdrawal approved | Admin approves withdrawal request |
| `withdrawalRejected` | Withdrawal rejected | Admin rejects withdrawal request |

### Webhook Payload Structure

All webhooks follow this structure:

```json
{
  "event": "depositCreated",
  "timestamp": "2024-01-20T10:30:00.000Z",
  "data": {
    // Event-specific data (see examples below)
  },
  "signature": "hmac_sha256_signature"
}
```

### Event-Specific Payloads

#### Deposit Events
```json
{
  "event": "depositCreated",
  "timestamp": "2024-01-20T10:30:00.000Z",
  "data": {
    "transactionId": "TXN123456789",
    "amount": 5000,
    "customerEmail": "user@example.com",
    "customerName": "John Doe",
    "paymentMethod": "PhonePe Business",
    "status": "pending",
    "screenshot": "https://your-server.com/uploads/screenshot.jpg",
    "createdAt": "2024-01-20T10:30:00.000Z"
  },
  "signature": "abc123..."
}
```

#### Deposit Approval/Rejection
```json
{
  "event": "depositApproved",
  "timestamp": "2024-01-20T11:00:00.000Z",
  "data": {
    "transactionId": "TXN123456789",
    "amount": 5000,
    "customerEmail": "user@example.com",
    "customerName": "John Doe",
    "paymentMethod": "PhonePe Business",
    "status": "approved",
    "remarks": "Verified and approved",
    "processedBy": "Admin User",
    "processedAt": "2024-01-20T11:00:00.000Z"
  },
  "signature": "def456..."
}
```

#### Withdrawal Events
```json
{
  "event": "withdrawalCreated",
  "timestamp": "2024-01-20T12:00:00.000Z",
  "data": {
    "requestId": "WR1642678901ABCDE",
    "amount": 2000,
    "customerEmail": "user@example.com",
    "customerName": "John Doe",
    "withdrawalMethod": "UPI",
    "accountDetails": {
      "upiId": "user@paytm"
    },
    "status": "pending",
    "createdAt": "2024-01-20T12:00:00.000Z"
  },
  "signature": "ghi789..."
}
```

#### Withdrawal Approval/Rejection
```json
{
  "event": "withdrawalApproved",
  "timestamp": "2024-01-20T13:00:00.000Z",
  "data": {
    "requestId": "WR1642678901ABCDE",
    "transactionId": "TXN987654321",
    "amount": 2000,
    "customerEmail": "user@example.com",
    "customerName": "John Doe",
    "withdrawalMethod": "UPI",
    "accountDetails": {
      "upiId": "user@paytm"
    },
    "status": "approved",
    "remarks": "Processed successfully",
    "processedBy": "Admin User",
    "processedAt": "2024-01-20T13:00:00.000Z"
  },
  "signature": "jkl012..."
}
```

### Webhook Signature Verification

**Important**: Always verify webhook signatures to ensure authenticity.

#### Node.js Example
```javascript
const crypto = require('crypto');
const express = require('express');
const app = express();

app.use(express.json());

const WEBHOOK_SECRET = 'your_secret_key_from_admin_panel';

function verifyWebhookSignature(payload, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return signature === expectedSignature;
}

app.post('/api/payment-callback', (req, res) => {
  const { event, timestamp, data, signature } = req.body;
  
  // Verify signature
  if (!verifyWebhookSignature(data, signature, WEBHOOK_SECRET)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Process the webhook
  console.log(`Received ${event} event:`, data);
  
  switch (event) {
    case 'depositCreated':
      // Handle new deposit
      handleNewDeposit(data);
      break;
      
    case 'depositApproved':
      // Handle deposit approval
      handleDepositApproval(data);
      break;
      
    case 'withdrawalCreated':
      // Handle new withdrawal
      handleNewWithdrawal(data);
      break;
      
    // ... handle other events
  }
  
  res.status(200).json({ received: true });
});

function handleNewDeposit(data) {
  // Update user balance (pending)
  // Send notification to user
  // Log transaction
}

function handleDepositApproval(data) {
  // Update user balance (confirmed)
  // Send confirmation notification
  // Update transaction status
}
```

#### PHP Example
```php
<?php
$webhookSecret = 'your_secret_key_from_admin_panel';
$payload = file_get_contents('php://input');
$data = json_decode($payload, true);

// Verify signature
$expectedSignature = hash_hmac('sha256', json_encode($data['data']), $webhookSecret);
if ($data['signature'] !== $expectedSignature) {
    http_response_code(401);
    exit('Invalid signature');
}

// Process webhook
switch ($data['event']) {
    case 'depositCreated':
        handleNewDeposit($data['data']);
        break;
    case 'depositApproved':
        handleDepositApproval($data['data']);
        break;
    // ... handle other events
}

http_response_code(200);
echo json_encode(['received' => true]);
?>
```

### Best Practices

1. **Always verify signatures** to ensure webhook authenticity
2. **Handle idempotency** - you might receive the same webhook multiple times
3. **Respond quickly** - return HTTP 200 within 10 seconds
4. **Log webhook events** for debugging and monitoring
5. **Handle failures gracefully** - implement retry logic if needed
6. **Use HTTPS** for your webhook endpoint for security

### Testing Webhooks

Use the "Test Callback" feature in the admin dashboard to:
- Verify your webhook endpoint is accessible
- Test signature verification
- Debug payload handling
- Monitor webhook delivery success/failure rates

### Webhook Delivery

- **Timeout**: 10 seconds
- **Retry Policy**: Currently no automatic retries (implement your own if needed)
- **Expected Response**: HTTP 200 status code
- **Content-Type**: `application/json`
- **Headers**: 
  - `X-Callback-Event`: Event type
  - `X-Callback-Signature`: HMAC signature
