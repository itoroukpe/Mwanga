
## Current Order Workflow (What Exists)

graph TD
    A[Customer Browses Marketplace] --> B[Adds Items to Cart]
    B --> C[Views Cart Page]
    C --> D[Clicks Proceed to Checkout]
    D --> E[Fills Shipping Address Form]
    E --> F[Submits Order]
    F --> G[create-order Edge Function]

    G --> H{Server-Side Validation}
    H --> I[Verify Authentication]
    I --> J[Fetch Artwork Details]
    J --> K[Calculate Prices Server-Side]
    K --> L[Check Stock Availability]
    L --> M[Create Stripe Payment Intent]
    M --> N[Create Order Record]
    N --> O[Create Order Items]
    O --> P[Create Artist Earnings]
    P --> Q[Update Stock Quantities]
    Q --> R[Clear Cart]

    R --> S[Return client_secret to Frontend]
    S --> T[Order Status: PENDING]

    T --> U[??? Payment Capture Missing ???]
    U --> V[??? Customer Order Tracking Missing ???]
    V --> W[??? Shipping Integration Missing ???]
    W --> X[??? Delivery Confirmation Missing ???]
    X --> Y[??? Artist Payout Missing ???]

    style U fill:#ff6b6b
    style V fill:#ff6b6b
    style W fill:#ff6b6b
    style X fill:#ff6b6b
    style Y fill:#ff6b6b

## Critical Missing Components

### 🔴 **1. Payment Completion Flow**
**Status**: The Stripe payment intent is created but never captured!
- The `client_secret` is returned but not used on the frontend
- No Stripe Elements integration to collect payment details
- No webhook to confirm payment success
- Orders stay in "pending" status indefinitely

### 🔴 **2. Customer Order Tracking**
**Status**: Completely missing
- No customer-facing "My Orders" page
- Customers cannot view their order status
- No order confirmation page after checkout
- No email notifications to customers

### 🔴 **3. Shipping & Fulfillment Workflow**
**Status**: Only status field exists, no process
- Admin can update status manually
- No shipping label generation
- No tracking number storage
- No carrier integration (FedEx, UPS, DHL)
- No estimated delivery dates

### 🔴 **4. Order Status Notifications**
**Status**: Missing for all parties
- No email to customer when order is confirmed
- No email when order ships (with tracking)
- No email when order is delivered
- No notifications to artists about sales

### 🔴 **5. Artist Payout System**
**Status**: Earnings tracked but no payout mechanism
- `artist_earnings` records created with status "pending"
- No way to mark earnings as paid
- No integration with Stripe Connect for payouts
- No payout schedule or dashboard for artists

### 🔴 **6. Returns & Refunds**
**Status**: Status exists but no process
- "cancelled" and "refunded" statuses exist
- No refund processing through Stripe
- No return request workflow
- No restocking of inventory

## Complete Transaction Workflow (Proposed)

sequenceDiagram
    participant C as Customer
    participant FE as Frontend
    participant EF as Edge Function
    participant DB as Database
    participant S as Stripe
    participant W as Webhook
    participant E as Email Service
    participant A as Artist
    participant Admin as Admin Panel

    Note over C,Admin: Phase 1: Order Creation (IMPLEMENTED ✅)
    C->>FE: Add items to cart
    C->>FE: Proceed to checkout
    FE->>EF: create-order (shipping address)
    EF->>DB: Validate inventory
    EF->>S: Create payment intent
    S-->>EF: client_secret
    EF->>DB: Create order (pending)
    EF-->>FE: Return client_secret

    Note over C,Admin: Phase 2: Payment (MISSING ❌)
    FE->>FE: Show Stripe Elements
    C->>FE: Enter card details
    FE->>S: Confirm payment
    S->>W: payment_intent.succeeded webhook
    W->>DB: Update order status (paid)
    W->>E: Send confirmation email
    E->>C: Order confirmation + receipt
    E->>A: New sale notification

    Note over C,Admin: Phase 3: Fulfillment (MISSING ❌)
    Admin->>Admin: Process order
    Admin->>Admin: Generate shipping label
    Admin->>DB: Update status (processing → shipped)
    Admin->>DB: Add tracking number
    DB->>E: Trigger shipping email
    E->>C: Shipment notification + tracking

    Note over C,Admin: Phase 4: Delivery (MISSING ❌)
    Admin->>DB: Mark as delivered
    DB->>E: Trigger delivery email
    E->>C: Delivery confirmation

    Note over C,Admin: Phase 5: Payout (MISSING ❌)
    Admin->>DB: Approve artist payout
    DB->>S: Transfer to artist Stripe account
    S->>A: Funds deposited
    DB->>DB: Update earnings (paid)
    E->>A: Payout notification

## Recommended Implementation Plan

### **Phase 2A: Payment Completion (CRITICAL - HIGH PRIORITY)**

**Step 1: Add Stripe Elements to Checkout**
- Install `@stripe/stripe-js` and `@stripe/react-stripe-js` packages
- Create `PaymentForm` component with Stripe Elements
- Update `CheckoutDialog` to show payment form after shipping address
- Use the `client_secret` returned from create-order to confirm payment
- Handle payment success/failure states with proper error messages

**Step 2: Create Stripe Webhook Handler**
- Create Edge Function `supabase/functions/stripe-webhook/index.ts`
- Verify webhook signature using Stripe signing secret
- Handle `payment_intent.succeeded` event to update order status to "paid"
- Handle `payment_intent.payment_failed` to mark order as "failed"
- Handle `charge.refunded` to process refunds
- Add proper logging for debugging

**Step 3: Order Confirmation Page**
- Create `src/pages/OrderConfirmation.tsx` route at `/order-confirmation/:orderId`
- Display order summary, payment confirmation, estimated delivery
- Provide downloadable receipt
- Show next steps for customer

---

### **Phase 2B: Customer Order Management (HIGH PRIORITY)**

**Step 4: My Orders Page**
- Create `src/pages/MyOrders.tsx` route at `/orders`
- Create `useCustomerOrders` hook to fetch customer's orders
- Display order list with filters (status, date range)
- Show order cards with thumbnail, status, total, date
- Add search by order number functionality

**Step 5: Order Detail/Tracking Page**
- Create `src/pages/OrderTracking.tsx` route at `/orders/:orderId`
- Display full order details, items, shipping address
- Show order status timeline (pending → paid → processing → shipped → delivered)
- Display tracking number and carrier link when available
- Add "Contact Support" button for issues

**Step 6: Navigation Updates**
- Add "My Orders" link to customer dashboard
- Add order count badge in navigation
- Update Dashboard to show recent orders

---

### **Phase 2C: Email Notifications (MEDIUM PRIORITY)**

**Step 7: Order Confirmation Email**
- Create Edge Function `send-order-confirmation` 
- Trigger from webhook after payment success
- Include order summary, items, total, estimated delivery
- Add tracking link to order status page
- Use RESEND_API_KEY (already available)

**Step 8: Shipping Notification Email**
- Create Edge Function `send-shipping-notification`
- Trigger when admin updates status to "shipped"
- Include tracking number and carrier information
- Add expected delivery date
- Provide link to carrier tracking page

**Step 9: Artist Sale Notification**
- Create Edge Function `notify-artist-sale`
- Trigger after payment success
- Notify artist of new sale with artwork details
- Show their earnings amount
- Provide link to earnings dashboard

---

### **Phase 2D: Admin Fulfillment Tools (MEDIUM PRIORITY)**

**Step 10: Enhanced Order Management**
- Add bulk actions (mark as shipped, export orders)
- Add filtering by date range, status, customer
- Add order notes/comments field for internal tracking
- Add "Print Packing Slip" functionality

**Step 11: Shipping Integration**
- Add fields to orders table: `tracking_number`, `shipping_carrier`, `estimated_delivery_date`
- Create UI in OrderDetailModal to add tracking info
- Add support for multiple carriers (UPS, FedEx, DHL, USPS)
- Generate tracking URL based on carrier

**Step 12: Inventory Management**
- Add low stock alerts when quantity < threshold
- Add stock history tracking table
- Show "Out of Stock" badge on artworks
- Prevent orders when stock = 0

---

### **Phase 2E: Artist Payouts (MEDIUM PRIORITY)**

**Step 13: Payout Dashboard for Artists**
- Create `src/pages/artist/Payouts.tsx`
- Show pending earnings vs paid earnings
- Display payout history with dates and amounts
- Add bank account connection flow (Stripe Connect)
- Show next payout date

**Step 14: Admin Payout Processing**
- Create payout approval workflow in admin panel
- Add `payout_batch_id` field to artist_earnings
- Create Edge Function to process batch payouts via Stripe
- Update earnings status to "paid" after successful transfer
- Generate payout reports for accounting

**Step 15: Stripe Connect Integration**
- Set up Stripe Connect for artist accounts
- Store `stripe_account_id` in artist_profiles (already exists!)
- Create onboarding flow for artists to connect bank account
- Handle payout transfers automatically after delivery confirmation

---

### **Phase 2F: Returns & Refunds (LOW PRIORITY)**

**Step 16: Return Request System**
- Create `returns` table with reason, status, photos
- Add "Request Return" button on order detail page
- Create admin interface to review return requests
- Add return label generation

**Step 17: Refund Processing**
- Create Edge Function to process Stripe refunds
- Update order status to "refunded"
- Restore inventory quantities
- Deduct from artist earnings if not yet paid
- Send refund confirmation emails

---

### **Database Schema Updates Required**

```sql
-- Add shipping and tracking fields to orders
ALTER TABLE orders 
ADD COLUMN tracking_number TEXT,
ADD COLUMN shipping_carrier TEXT,
ADD COLUMN estimated_delivery_date TIMESTAMP WITH TIME ZONE,
ADD COLUMN order_notes TEXT;

-- Add payout tracking
ALTER TABLE artist_earnings
ADD COLUMN payout_batch_id TEXT,
ADD COLUMN payout_date TIMESTAMP WITH TIME ZONE;

-- Create returns table
CREATE TABLE returns (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES orders(id) NOT NULL,
  reason TEXT NOT NULL,
  status TEXT DEFAULT 'pending',
  photos TEXT[],
  admin_notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

---

### **Priority Order for Implementation**

1. **🔴 CRITICAL**: Payment completion (Stripe Elements + Webhook) - Without this, no orders can be completed!
2. **🟠 HIGH**: Customer order tracking page - Customers need to see their orders
3. **🟠 HIGH**: Order confirmation email - Basic customer communication
4. **🟡 MEDIUM**: Shipping notifications and tracking numbers
5. **🟡 MEDIUM**: Artist payout dashboard and processing
6. **🟢 LOW**: Returns and refunds system

## Summary

Your transaction workflow has a **solid foundation** with secure server-side order creation, but it's **incomplete**. The biggest issue is that **payments are created but never captured** - the Stripe payment intent exists but customers never enter their card details!

**The critical path to complete the transaction flow is:**

1. ✅ Cart & Checkout UI (Done)
2. ✅ Server-side order creation (Done)
3. ❌ **Payment collection with Stripe Elements** (MISSING - Most Critical!)
4. ❌ **Webhook to confirm payment** (MISSING - Critical!)
5. ❌ Customer order tracking page (MISSING - Important)
6. ❌ Shipping & fulfillment workflow (MISSING - Important)
7. ❌ Email notifications (MISSING - Important)
8. ❌ Artist payout system (MISSING - Medium)

Would you like me to implement Phase 2A (Payment Completion) first? This is the most critical missing piece - without it, no orders can actually be completed and paid for!

Implement Phase 2A: Payment Completion
