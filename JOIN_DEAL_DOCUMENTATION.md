# Join Deal Payment Integration Documentation

## Overview
This documentation explains how to integrate Paystack payment for joining deals in mobile applications. The payment flow follows the same pattern as the web implementation, ensuring consistency across platforms.

## Payment Flow
1. User selects a deal to join
2. User chooses delivery option (Delivery, Pickup, or Redemption)
3. User initiates payment via Paystack
4. Payment is processed with metadata
5. Webhook handles post-payment processing (group creation, order creation, notifications)

---

## Paystack Configuration

### Environment Variables
```
PAYSTACK_PUBLIC_KEY=pk_test_your_public_key
PAYSTACK_SECRET_KEY=sk_test_your_secret_key (server-side only)
```

### Test Keys (for development)
- **Public Key:** `pk_test_0372249249bd604218edc3421754ddd8fc46b329`
- **Secret Key:** `sk_test_966bec0849d79db678086cd3a670859828830426`

---

## Required Payment Metadata

When initiating payment, you must include specific metadata that the webhook will use to process the transaction:

```typescript
interface JoinDealMetadata {
  userId: string;
  dealId: string;
  amount: number;
  deliveryOption: "Delivery" | "Pickup" | "Redemption";
  redemptionDetails?: object;
}
```

### Metadata Fields
- **userId**: ID of the user making the payment
- **dealId**: ID of the deal being joined
- **amount**: Payment amount in NGN (not kobo)
- **deliveryOption**: How the user wants to receive the deal
- **redemptionDetails**: Additional details for redemption (only for Redemption option)

---


### For Example in React Native

#### 1. Install Paystack SDK
```bash
npm install @hurshore/react-native-paystack-webview
# or
yarn add @hurshore/react-native-paystack-webview
```

#### 2. Payment Component
```typescript
import React, { useState } from 'react';
import { TouchableOpacity, Text } from 'react-native';
import { PayStackWebView } from '@hurshore/react-native-paystack-webview';

interface JoinDealMetadata {
  userId: string;
  dealId: string;
  amount: number;
  deliveryOption: 'Delivery' | 'Pickup' | 'Redemption';
  redemptionDetails?: object;
}

interface PaymentProps {
  deal: Deal;
  user: User;
  deliveryOption: string;
  redemptionDetails?: object;
  onSuccess: (reference: string) => void;
  onError: (error: string) => void;
}

const JoinDealPayment: React.FC<PaymentProps> = ({
  deal,
  user,
  deliveryOption,
  redemptionDetails,
  onSuccess,
  onError
}) => {
  const [showPayment, setShowPayment] = useState(false);

  const initiatePayment = () => {
    setShowPayment(true);
  };

  // Generate unique reference
  const cleanTitle = deal.title.replace(/[^a-zA-Z0-9]/g, '_');
  const cleanUserName = user.name.replace(/[^a-zA-Z0-9]/g, '_');
  const reference = `${cleanTitle}_${cleanUserName}_${Date.now()}`;

  // Calculate total amount
  const totalAmount = calculateDealTotalPrice(deal);

  // Create metadata
  const metadata: JoinDealMetadata = {
    userId: user.id,
    dealId: deal.id,
    amount: totalAmount,
    deliveryOption: deliveryOption as any,
    redemptionDetails
  };

  return (
    <>
      <TouchableOpacity onPress={initiatePayment}>
        <Text>Pay Now</Text>
      </TouchableOpacity>

      {showPayment && (
        <PayStackWebView
          paystackKey="pk_test_0372249249bd604218edc3421754ddd8fc46b329"
          amount={totalAmount * 100} // Convert to kobo
          billingEmail={user.email}
          billingName={user.name}
          reference={reference}
          metadata={JSON.stringify(metadata)}
          onCancel={() => {
            setShowPayment(false);
            onError('Payment cancelled');
          }}
          onSuccess={(res) => {
            setShowPayment(false);
            onSuccess(res.transactionRef.reference);
          }}
          autoStart={true}
        />
      )}
    </>
  );
};

// Helper function
const calculateDealTotalPrice = (deal: Deal): number => {
  const basePrice = deal.discountPrice || deal.pricePerPerson;
  const deliveryFee = deal.deliveryFee || 0;
  return basePrice + deliveryFee;
};
```

---

## Delivery Options

### 1. Delivery
- Physical delivery to user's address
- May include delivery fees
- Requires shipping address

### 2. Pickup
- User picks up from merchant location
- No delivery fees
- May require pickup location details

### 3. Redemption
- Digital redemption (e.g., voucher codes)
- Requires additional redemption details
- No physical delivery

### Redemption Details Example
```typescript
const redemptionDetails = {
  redemptionType: "digital_voucher",
  deliveryMethod: "email",
  additionalInfo: "Send voucher code to email"
};
```

---

## Post-Payment Processing

After successful payment, the webhook (`/api/paystack/webhook`) automatically:

1. **Validates payment** using Paystack signature
2. **Finds or creates group** for the deal
3. **Adds user to group** as paid member
4. **Creates order record** for the user
5. **Creates transaction record**
6. **Sends notifications** to user
7. **Updates merchant balance**
8. **Creates chat group** for group communication
9. **Sends email confirmation** to user

---

## Error Handling

### Common Errors
- **Invalid metadata**: Ensure all required fields are present
- **Network issues**: Implement retry logic
- **Payment cancellation**: Handle user cancellation gracefully
- **Amount validation**: Ensure amount matches deal price

### Error Response Example
```json
{
  "error": "Payment failed",
  "details": "Invalid payment amount"
}
```

---

## Testing

### Test Cards
Use these test cards for development:

#### Successful Payments
- **Card Number**: 4084084084084081
- **Expiry**: Any future date
- **CVV**: Any 3 digits

#### Failed Payments
- **Card Number**: 4084084084084085
- **Expiry**: Any future date
- **CVV**: Any 3 digits

### Test Flow
1. Use test public key in development
2. Use test card numbers for payments
3. Monitor webhook calls in development logs
4. Verify group creation and order processing

---

## Security Considerations

1. **Never expose secret key** in mobile apps
2. **Validate payments server-side** via webhook
3. **Use HTTPS** for all API calls
4. **Implement proper error handling**
5. **Log payment attempts** for debugging

---

## Integration Checklist

- [ ] Paystack SDK integrated
- [ ] Public key configured
- [ ] Metadata structure implemented
- [ ] Payment flow tested
- [ ] Error handling implemented
- [ ] Success navigation configured
- [ ] Webhook endpoint configured
- [ ] Test payments working

---

## Support

For additional help with Paystack integration:
- [Paystack Documentation](https://paystack.com/docs)
- [Android SDK](https://github.com/PaystackHQ/paystack-android)
- [iOS SDK](https://github.com/PaystackHQ/paystack-ios)
- [React Native](https://github.com/just1and0/React-Native-Paystack-WebView)
- [Hurshore React Native Paystack](https://www.npmjs.com/package/@hurshore/react-native-paystack-webview)

**Last Updated:** July 2025
