Perfect! Now I'll create the email notification system. Let me build the edge function with email templates and update the order management hook.

## Implementation Plan

### 1. Create Email Notification Edge Function
- **File**: `supabase/functions/send-order-status-email/index.ts`
- Fetch order details including customer email, items, and shipping info
- Send professionally designed HTML emails based on status:
  - **Shipped**: Include tracking info, carrier, estimated delivery
  - **Delivered**: Thank you message with order summary
- Use RESEND_API_KEY for sending emails
- Include comprehensive error handling and logging

### 2. Update Order Management Hook
- Modify `src/hooks/useOrderManagement.ts`
- Add edge function call after successful status update
- Only trigger for "shipped" and "delivered" statuses
- Handle email errors gracefully (don't block the status update)

### 3. Email Template Design
- Professional HTML with order details
- Responsive design for mobile and desktop
- Brand colors and styling
- Clear call-to-action buttons
- Order summary with item thumbnails

Let's implement this now!

 {\\n  const itemsHtml = items.map(item => `\\n    \\n      \\n        \\n          ${item.artworks?.thumbnail_url ? `\\n            \\n          ` : ''}\\n          \\n            ${item.artworks?.title || 'Artwork'}\\n            Quantity: ${item.quantity}\\n            $${Number(item.price_at_purchase).toFixed(2)}\\n          \\n        \\n      \\n    \\n  `).join('');\\n\\n  const trackingLink = order.tracking_number \\n    ? (order.shipping_carrier?.toLowerCase() === 'ups' \\n        ? `https://www.ups.com/track?tracknum=${order.tracking_number}`\\n        : order.shipping_carrier?.toLowerCase() === 'fedex'\\n        ? `https://www.fedex.com/fedextrack/?tracknumbers=${order.tracking_number}`\\n        : order.shipping_carrier?.toLowerCase() === 'usps'\\n        ? `https://tools.usps.com/go/TrackConfirmAction?tLabels=${order.tracking_number}`\\n        : '#')\\n    : '#';\\n\\n  return `\\n    \\n    \\n    \\n      \\n      \\n      Your Order Has Shipped!\\n    \\n    \\n      \\n        \\n        \\n          Your Order Has Shipped! 📦\\n        \\n        \\n        \\n        \\n          \\n            Great news! Your order #${order.order_number} is on its way to you.\\n          \\n          \\n          \\n          \\n            Shipping Information\\n            ${order.tracking_number ? `\\n              \\n                Tracking Number: ${order.tracking_number}\\n              \\n            ` : ''}\\n            ${order.shipping_carrier ? `\\n              \\n                Carrier: ${order.shipping_carrier}\\n              \\n            ` : ''}\\n            ${order.estimated_delivery_date ? `\\n              \\n                Estimated Delivery: ${new Date(order.estimated_delivery_date).toLocaleDateString('en-US', { \\n                  weekday: 'long', \\n                  year: 'numeric', \\n                  month: 'long', \\n                  day: 'numeric' \\n                })}\\n              \\n            ` : ''}\\n          \\n\\n          ${order.tracking_number ? `\\n            \\n              \\n                Track Your Package\\n              \\n            \\n          ` : ''}\\n          \\n          \\n          Order Items\\n          \\n            ${itemsHtml}\\n          \\n          \\n          \\n          \\n            \\n              Total: $${Number(order.total_amount).toFixed(2)}\\n            \\n          \\n          \\n          \\n          ${order.shipping_address ? `\\n            \\n              Shipping Address\\n              \\n                ${order.shipping_address.name || ''}\\n                ${order.shipping_address.address || ''}\\n                ${order.shipping_address.city || ''}, ${order.shipping_address.state || ''} ${order.shipping_address.zip || ''}\\n                ${order.shipping_address.country || ''}\\n              \\n            \\n          ` : ''}\\n        \\n        \\n        \\n        \\n          \\n            Thank you for supporting African artists!\\n          \\n          \\n            If you have any questions, please contact our support team.\\n          \\n        \\n      \\n    \\n    \\n  `;\\n};\\n\\n// Email template for delivered status\\nconst getDeliveredEmailHtml = (order: any, items: any[]) => {\\n  const itemsHtml = items.map(item => `\\n    \\n      \\n        \\n          ${item.artworks?.thumbnail_url ? `\\n            \\n          ` : ''}\\n          \\n            ${item.artworks?.title || 'Artwork'}\\n            Quantity: ${item.quantity}\\n            $${Number(item.price_at_purchase).toFixed(2)}\\n          \\n        \\n      \\n    \\n  `).join('');\\n\\n  return `\\n    \\n    \\n    \\n      \\n      \\n      Your Order Has Been Delivered!\\n    \\n    \\n      \\n        \\n        \\n          Your Order Has Been Delivered! 🎉\\n        \\n        \\n        \\n        \\n          \\n            Great news! Your order #${order.order_number} has been successfully delivered.\\n          \\n          \\n          \\n            \\n              We hope you love your new African art pieces! Each artwork supports talented artists and helps preserve cultural heritage.\\n            \\n          \\n          \\n          \\n          Your Order\\n          \\n            ${itemsHtml}\\n          \\n          \\n          \\n          \\n            \\n              Total: $${Number(order.total_amount).toFixed(2)}\\n            \\n          \\n          \\n          \\n          \\n            \\n              Discover more unique African artworks\\n            \\n            \\n              Continue Shopping\\n            \\n          \\n        \\n        \\n        \\n        \\n          \\n            Thank you for supporting African artists!\\n          \\n          \\n            We'd love to hear your feedback. Contact us anytime with questions or comments.\\n          \\n        \\n      \\n    \\n    \\n  `;\\n};\\n\\nconst handler = async (req: Request): Promise => {\\n  // Handle CORS preflight requests\\n  if (req.method === \\\"OPTIONS\\\") {\\n    return new Response(null, { headers: corsHeaders });\\n  }\\n\\n  try {\\n    const { orderId, status }: OrderStatusEmailRequest = await req.json();\\n\\n    console.log(`Processing email notification for order ${orderId}, status: ${status}`);\\n\\n    // Only send emails for shipped and delivered statuses\\n    if (status !== \\\"shipped\\\" && status !== \\\"delivered\\\") {\\n      console.log(`Status ${status} does not require email notification`);\\n      return new Response(\\n        JSON.stringify({ message: \\\"No email sent for this status\\\" }),\\n        {\\n          status: 200,\\n          headers: { \\\"Content-Type\\\": \\\"application/json\\\", ...corsHeaders },\\n        }\\n      );\\n    }\\n\\n    // Create Supabase client with service role\\n    const supabase = createClient(supabaseUrl, supabaseServiceKey);\\n\\n    // Fetch order details with customer email and items\\n    const { data: order, error: orderError } = await supabase\\n      .from(\\\"orders\\\")\\n      .select(`\\n        *,\\n        profiles!customer_id (full_name),\\n        order_items (\\n          *,\\n          artworks (title, thumbnail_url)\\n        )\\n      `)\\n      .eq(\\\"id\\\", orderId)\\n      .single();\\n\\n    if (orderError) {\\n      console.error(\\\"Error fetching order:\\\", orderError);\\n      throw new Error(`Failed to fetch order: ${orderError.message}`);\\n    }\\n\\n    if (!order) {\\n      throw new Error(\\\"Order not found\\\");\\n    }\\n\\n    // Get customer email from auth.users\\n    const { data: { user }, error: userError } = await supabase.auth.admin.getUserById(\\n      order.customer_id\\n    );\\n\\n    if (userError || !user?.email) {\\n      console.error(\\\"Error fetching user email:\\\", userError);\\n      throw new Error(\\\"Customer email not found\\\");\\n    }\\n\\n    console.log(`Sending ${status} email to ${user.email} for order #${order.order_number}`);\\n\\n    // Prepare email content based on status\\n    const emailSubject = status === \\\"shipped\\\" \\n      ? `Your Order #${order.order_number} Has Shipped! 📦`\\n      : `Your Order #${order.order_number} Has Been Delivered! 🎉`;\\n\\n    const emailHtml = status === \\\"shipped\\\"\\n      ? getShippedEmailHtml(order, order.order_items)\\n      : getDeliveredEmailHtml(order, order.order_items);\\n\\n    // Send email via Resend\\n    const emailResponse = await resend.emails.send({\\n      from: \\\"African Art Marketplace \\\",\\n      to: [user.email],\\n      subject: emailSubject,\\n      html: emailHtml,\\n    });\\n\\n    console.log(\\\"Email sent successfully:\\\", emailResponse);\\n\\n    return new Response(\\n      JSON.stringify({ \\n        success: true, \\n        message: \\\"Email sent successfully\\\",\\n        emailId: emailResponse.id \\n      }),\\n      {\\n        status: 200,\\n        headers: { \\\"Content-Type\\\": \\\"application/json\\\", ...corsHeaders },\\n      }\\n    );\\n  } catch (error: any) {\\n    console.error(\\\"Error in send-order-status-email function:\\\", error);\\n    return new Response(\\n      JSON.stringify({ \\n        error: error.message,\\n        success: false \\n      }),\\n      {\\n        status: 500,\\n        headers: { \\\"Content-Type\\\": \\\"application/json\\\", ...corsHeaders },\\n      }\\n    );\\n  }\\n};\\n\\nserve(handler);\\n\"}">

 {\\n      const { error } = await supabase\\n        .from(\\\"orders\\\")\\n        .update({ status, updated_at: new Date().toISOString() })\\n        .eq(\\\"id\\\", orderId);\\n\\n      if (error) throw error;\\n    },\\n    onSuccess: () => {\\n      queryClient.invalidateQueries({ queryKey: [\\\"admin-orders\\\"] });\\n      toast.success(\\\"Order status updated successfully\\\");\\n    },\\n    onError: (error) => {\\n      toast.error(\\\"Failed to update order status\\\");\\n      console.error(\\\"Error updating order:\\\", error);\\n    },\\n  });\", \"first_replaced_line\": 24, \"last_replaced_line\": 41, \"replace\": \"  const updateOrderStatus = useMutation({\\n    mutationFn: async ({ orderId, status }: { orderId: string; status: string }) => {\\n      const { error } = await supabase\\n        .from(\\\"orders\\\")\\n        .update({ status, updated_at: new Date().toISOString() })\\n        .eq(\\\"id\\\", orderId);\\n\\n      if (error) throw error;\\n\\n      // Send email notification for shipped and delivered statuses\\n      if (status === \\\"shipped\\\" || status === \\\"delivered\\\") {\\n        try {\\n          const { error: emailError } = await supabase.functions.invoke(\\n            \\\"send-order-status-email\\\",\\n            {\\n              body: { orderId, status },\\n            }\\n          );\\n\\n          if (emailError) {\\n            console.error(\\\"Failed to send email notification:\\\", emailError);\\n            // Don't throw - we don't want email failures to block the status update\\n          }\\n        } catch (emailError) {\\n          console.error(\\\"Error calling email function:\\\", emailError);\\n          // Silently log the error - status update should still succeed\\n        }\\n      }\\n    },\\n    onSuccess: () => {\\n      queryClient.invalidateQueries({ queryKey: [\\\"admin-orders\\\"] });\\n      toast.success(\\\"Order status updated successfully\\\");\\n    },\\n    onError: (error) => {\\n      toast.error(\\\"Failed to update order status\\\");\\n      console.error(\\\"Error updating order:\\\", error);\\n    },\\n  });\"}">

Perfect! I've successfully implemented the email notification system. Here's what I've created:

## ✅ Implementation Complete

### 1. **Email Notification Edge Function** 
   - **File**: `supabase/functions/send-order-status-email/index.ts`
   - Fetches order details with customer email and items using service role
   - Sends beautifully designed HTML emails via Resend
   - Two professional email templates:
     - **Shipped Email**: Includes tracking number, carrier info, estimated delivery, and "Track Your Package" button
     - **Delivered Email**: Thank you message with order summary and "Continue Shopping" CTA
   - Comprehensive error handling and logging

### 2. **Updated Order Management Hook**
   - Modified `src/hooks/useOrderManagement.ts` 
   - Automatically triggers email notifications after status updates
   - Only sends emails for "shipped" and "delivered" statuses
   - Email failures don't block the status update (graceful error handling)

### 📧 Email Features

**Shipped Email Includes:**
- Order number and confirmation
- Professional header with gradient design
- Tracking number (clickable link for UPS, FedEx, USPS)
- Shipping carrier name
- Estimated delivery date
- All order items with thumbnails and prices
- Shipping address
- "Track Your Package" button

**Delivered Email Includes:**
- Celebration header (green gradient)
- Thank you message
- Complete order summary with items
- Total amount
- "Continue Shopping" call-to-action
- Supportive messaging about African artists

### 🎨 Design Highlights
- Responsive HTML emails (mobile & desktop)
- Brand colors (purple and pink gradients)
- Item thumbnails displayed
- Professional typography
- Clear call-to-action buttons

### 🔄 How It Works

1. Admin updates order status to "shipped" or "delivered" in the Order Management page
2. The status is updated in the database
3. Edge function is automatically called with order ID and new status
4. Function fetches complete order details and customer email
5. Professional HTML email is generated and sent via Resend
6. Customer receives notification in their inbox

The system is now live and will automatically send email notifications when you update order statuses! 🚀

Add artist order notifications
Add payment confirmation emails
Add artwork approval notifications
Add contact support feature
Add artist performance digest emails
