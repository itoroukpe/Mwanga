
# Mwanga Marketplace for African Art - MVP Implementation Plan

## Project Overview
Building a faith-inspired e-commerce platform for African and diaspora artists to sell artwork globally, featuring a unique "Holy Calendar" widget with dual-date and triple-time display. This is a 12-week MVP with a $10,000 budget.

## Technology Stack Decision

Given the project requirements and Lovable's capabilities, I recommend:

**Frontend:** React with TypeScript (already set up in this project)
**Backend:** Supabase (Lovable Cloud) - provides:
- PostgreSQL database with automatic backups
- Authentication & user management
- Storage for artwork images
- Edge Functions for custom backend logic
- Built-in security and SSL

**Payments:** Stripe integration (Lovable has native Stripe support)
**CMS:** We'll build a lightweight headless CMS within the platform instead of Strapi, leveraging Supabase for content management
**Hosting:** Lovable's built-in deployment with SSL (already included)

## Database Schema Design

### Core Tables:

1. **user_roles** (Security-critical)
   - id (uuid, primary key)
   - user_id (uuid, references auth.users)
   - role (enum: 'admin', 'artist', 'customer')
   - Created with proper security definer functions to prevent privilege escalation

2. **profiles**
   - id (uuid, references auth.users)
   - full_name (text)
   - bio (text)
   - avatar_url (text)
   - country (text)
   - language_preference (enum: 'en', 'fr')
   - created_at, updated_at

3. **artist_profiles** (extends profile for artists)
   - id (uuid, references profiles)
   - artist_name (text)
   - artist_bio (text)
   - cultural_background (text)
   - is_approved (boolean)
   - approval_date (timestamp)
   - stripe_account_id (text, for future payouts)
   - commission_rate (decimal, default 0.70)

4. **artworks**
   - id (uuid)
   - artist_id (uuid, references artist_profiles)
   - title (text)
   - description (text)
   - story (text) - cultural storytelling element
   - category (enum: 'painting', 'sculpture', 'textile', 'jewelry', 'book', 'craft')
   - price (decimal)
   - currency (text, default 'USD')
   - image_urls (text array)
   - thumbnail_url (text)
   - dimensions (jsonb)
   - materials (text array)
   - origin_country (text)
   - is_approved (boolean)
   - is_active (boolean)
   - stock_quantity (integer)
   - created_at, updated_at
   - holy_calendar_date (text) - date created in Holy Calendar format

5. **orders**
   - id (uuid)
   - customer_id (uuid, references profiles)
   - order_number (text, unique)
   - status (enum: 'pending', 'paid', 'processing', 'shipped', 'delivered', 'cancelled')
   - total_amount (decimal)
   - currency (text)
   - stripe_payment_intent_id (text)
   - shipping_address (jsonb)
   - created_at, updated_at

6. **order_items**
   - id (uuid)
   - order_id (uuid, references orders)
   - artwork_id (uuid, references artworks)
   - artist_id (uuid, references artist_profiles)
   - quantity (integer)
   - price_at_purchase (decimal)
   - artist_share (decimal)
   - platform_share (decimal)

7. **blog_posts** (CMS content)
   - id (uuid)
   - author_id (uuid, references profiles)
   - title (text)
   - slug (text, unique)
   - content (text)
   - excerpt (text)
   - featured_image_url (text)
   - category (enum: 'artist_story', 'cultural_heritage', 'faith', 'news')
   - published (boolean)
   - published_at (timestamp)
   - created_at, updated_at

8. **cart_items**
   - id (uuid)
   - user_id (uuid, references auth.users)
   - artwork_id (uuid, references artworks)
   - quantity (integer)
   - created_at

9. **artist_earnings**
   - id (uuid)
   - artist_id (uuid, references artist_profiles)
   - order_item_id (uuid, references order_items)
   - amount (decimal)
   - status (enum: 'pending', 'paid')
   - payout_date (timestamp)

### Storage Buckets:
- **artwork-images** (public) - main artwork images
- **artwork-thumbnails** (public) - optimized thumbnails
- **profile-avatars** (public) - user avatars
- **blog-images** (public) - CMS images

### Row Level Security (RLS):
All tables will have comprehensive RLS policies ensuring:
- Artists can only edit their own artworks
- Customers can only view approved/active artworks
- Admins can moderate everything
- Role-based access using security definer functions

## Application Architecture

### Phase 1: Foundation & Authentication (Weeks 1-2)

**Features:**
1. Enable Supabase (Lovable Cloud)
2. User authentication system:
   - Email/password signup and login
   - OAuth support (Google) for easier onboarding
   - Email verification with proper redirects
   - Password reset functionality
3. User role selection during signup (Artist vs Customer)
4. Profile creation with automatic triggers
5. Responsive navigation with role-based menu items

**Pages:**
- `/` - Landing page with marketplace preview
- `/auth` - Combined login/signup page
- `/profile` - User profile management
- `/dashboard` - Role-specific dashboard redirect

### Phase 2: Artist Features (Weeks 3-4)

**Features:**
1. Artist Dashboard:
   - Upload artwork form with image preview
   - Multi-image upload support
   - Rich text editor for story/description
   - Category and metadata management
   - Submission for admin approval
2. Artist Portfolio Page:
   - Public-facing artist profile
   - Gallery of approved works
   - Bio and cultural background
3. Sales Analytics (basic):
   - Total sales revenue
   - Number of items sold
   - Pending earnings

**Pages:**
- `/artist/dashboard` - Artist control panel
- `/artist/upload` - Artwork upload form
- `/artist/my-works` - Manage uploaded artworks
- `/artist/earnings` - View sales and payouts
- `/artist/[id]` - Public artist profile

**Input Validation:**
- Use Zod schemas for all form inputs
- File size limits (max 10MB per image)
- Image format validation (JPEG, PNG, WEBP)
- XSS prevention on all text inputs

### Phase 3: Marketplace & Product Display (Weeks 5-6)

**Features:**
1. Marketplace Catalog:
   - Grid layout with artwork cards
   - Filter by category, price range, origin country
   - Search functionality (artwork title, artist name)
   - Sort options (newest, price, popularity)
2. Product Detail Page:
   - Image carousel with zoom
   - Artist information sidebar
   - Cultural story section
   - Holy Calendar date stamp
   - Add to cart functionality
   - Share buttons (social media)
3. Shopping Cart:
   - Add/remove items
   - Quantity management
   - Price calculation
   - Persistent cart (stored in database)

**Pages:**
- `/marketplace` - Main catalog page
- `/artwork/[id]` - Product detail page
- `/cart` - Shopping cart

**SEO Optimization:**
- Meta tags for each artwork
- Open Graph tags for social sharing
- Structured data for products
- Sitemap generation

### Phase 4: Checkout & Stripe Integration (Weeks 7-8)

**Features:**
1. Enable Stripe integration (Lovable native support)
2. Checkout flow:
   - Shipping address collection
   - Order review page
   - Stripe payment integration (cards, Apple Pay, Google Pay)
   - Order confirmation email
3. Order Management:
   - Order history for customers
   - Order tracking
   - Receipt generation
4. Edge Function for payment processing:
   - Webhook handler for Stripe events
   - Order creation on successful payment
   - Artist earnings calculation (70/30 split)
   - Email notifications

**Pages:**
- `/checkout` - Checkout page
- `/orders` - Order history
- `/order/[id]` - Order details

**Security:**
- PCI-DSS compliance via Stripe
- Never store card details
- Secure webhook signature verification
- Input validation on all payment data

### Phase 5: Admin Panel (Weeks 9-10)

**Features:**
1. Artist Moderation:
   - Pending artist applications list
   - Approve/reject artists
   - View artist details
2. Artwork Moderation:
   - Pending artwork submissions
   - Approve/reject artworks
   - Edit artwork details if needed
3. Order Management:
   - View all orders
   - Update order status
   - Process refunds (manual for MVP)
4. Analytics Dashboard:
   - Total sales metrics
   - Active artists/customers count
   - Revenue by category
   - Recent activity feed
5. Content Management:
   - Create/edit/delete blog posts
   - Rich text editor
   - Image uploads
   - Publish/unpublish posts

**Pages:**
- `/admin/dashboard` - Admin overview
- `/admin/artists` - Artist management
- `/admin/artworks` - Artwork moderation
- `/admin/orders` - Order management
- `/admin/blog` - CMS blog management
- `/admin/analytics` - Platform metrics

**Security:**
- Strict role-based access control
- Separate user_roles table with RLS
- Security definer functions for role checks
- Admin activity logging

### Phase 6: Holy Calendar Widget (Weeks 9-10)

**Features:**
1. Calendar Algorithm:
   - Gregorian to Holy Calendar conversion
   - 360-day year (12 months × 30 days)
   - Month names: Nisani, Ziv, Siwani, Zantiko, Dioskuro, Eluli, Etanimu, Buli, Kislevu, Tebethi, Shebati, Adari
   - Day names: Anzia, Fuatia, Eneza, Angaza, Fungo, Sathoni, Sabato

2. Triple Time Display:
   - Human Time (12hr day + 12hr night, starts 4:45 AM)
   - Elected Time (4 parts × 6 hours)
   - Angelic Time (3 parts × 8 hours)

3. Widget Components:
   - Homepage display widget
   - Dual-date display
   - Real-time clock updates
   - Interactive converter modal
   - Educational tooltips

4. Integration Points:
   - Display on homepage
   - Show on artwork pages (creation date)
   - Include in artist profiles
   - Footer widget on all pages

**Implementation:**
- React component with custom hooks
- Date calculation utilities
- LocalStorage for timezone preferences
- Responsive design for mobile
---
**Note: For MVP, skip this and note as Phase 6 enhancement.**
---
### Phase 7: Blog & Storytelling (Weeks 9-10)

**Features:**
1. Blog Listing Page:
   - Featured posts
   - Category filters
   - Pagination
2. Blog Post Page:
   - Rich content display
   - Author information
   - Related posts
   - Social sharing
3. Artist Stories Integration:
   - Artist spotlight posts
   - Cultural heritage articles
   - Faith-based content

**Pages:**
- `/blog` - Blog listing
- `/blog/[slug]` - Individual post
- `/blog/category/[category]` - Category archive

### Phase 8: Multi-language Support (Week 11)

**Features:**
1. i18n setup for English and French
2. Language selector in navigation
3. Translated UI strings
4. Language preference saved in profile

**Implementation:**
- react-i18next library
- Translation files for EN/FR
- Dynamic content translation (later phase)
- RTL support preparation

### Phase 9: Testing & Polish (Week 11-12)

**Activities:**
1. Cross-browser testing (Chrome, Firefox, Safari, Edge)
2. Mobile responsiveness testing
3. Performance optimization:
   - Image lazy loading
   - Code splitting
   - CDN configuration
4. Security audit:
   - RLS policy review
   - Input validation testing
   - XSS prevention verification
5. User acceptance testing with sample data:
   - 5 test artists
   - 20 sample artworks
   - Test transactions
6. Bug fixes and refinements
7. Documentation:
   - Admin user guide
   - Artist onboarding guide
   - API documentation

### Phase 10: Deployment & Launch (Week 12)

**Activities:**
1. Production deployment via Lovable
2. Environment variables configuration
3. Stripe production mode setup
4. Database migration to production
5. SSL verification
6. Domain setup (if custom domain)
7. Monitoring setup
8. Backup verification
9. Launch checklist completion
10. Handover documentation

## Design Aesthetic

**Visual Theme: "Afro-Modern Faith-Inspired"**

**Color Palette:**
- Primary: Rich Terracotta (#D2691E) - African earth tones
- Secondary: Deep Indigo (#4B0082) - spiritual depth
- Accent: Gold (#FFD700) - celebration and excellence
- Neutral: Warm beige backgrounds (#F5F5DC)
- Text: Deep charcoal (#2C2C2C)

**Typography:**
- Headers: Playfair Display (elegant, traditional)
- Body: Inter (modern, readable)

**UI Elements:**
- Rounded corners (8px-12px)
- Subtle shadows and depth
- Cultural patterns as decorative elements
- Generous whitespace
- High-quality imagery focus
- Faith symbols integrated tastefully

## Key Features Summary

### MVP Deliverables:
✅ User authentication (email/password + Google OAuth)
✅ Role-based access (Artist, Customer, Admin)
✅ Artist dashboard with artwork upload
✅ Admin moderation for artists and artworks
✅ Marketplace catalog with search/filters
✅ Product detail pages with cultural storytelling
✅ Shopping cart functionality
✅ Stripe checkout integration
✅ Order management system
✅ Holy Calendar widget with dual-date & triple-time
✅ Blog/CMS for cultural content
✅ Artist earnings tracking (70/30 split)
✅ Mobile-responsive design
✅ Multi-language support (EN/FR)
✅ Admin analytics dashboard
✅ Secure hosting with SSL

### Post-MVP Enhancements (Phase 2):
- Stripe Connect for automated artist payouts
- Mobile app (React Native)
- Customer reviews and ratings
- Wishlist functionality
- Print-on-demand integration (Printful)
- Advanced shipping integration
- Swahili language support
- Holy Calendar API service
- Newsletter system
- Affiliate/ambassador program
- Advanced analytics
- Cryptocurrency payment option

## Security Measures

1. **Authentication & Authorization:**
   - Supabase Auth with email verification
   - Secure password hashing (bcrypt)
   - JWT token-based sessions
   - Role-based access control with separate user_roles table
   - Security definer functions to prevent privilege escalation

2. **Payment Security:**
   - PCI-DSS compliance via Stripe
   - No card data stored locally
   - Webhook signature verification
   - Secure API key management via Lovable secrets

3. **Data Protection:**
   - RLS policies on all tables
   - Input validation using Zod schemas
   - XSS prevention on all user inputs
   - SQL injection prevention (parameterized queries)
   - CSRF protection

4. **Infrastructure:**
   - SSL/TLS encryption (automatic via Lovable)
   - Database backups (automatic via Supabase)
   - Environment variables for secrets
   - Rate limiting on API endpoints

## Success Metrics (First 3 Months)

- ✅ ≥10 artist sign-ups
- ✅ ≥20 artworks listed
- ✅ ≥20 buyer registrations
- ✅ ≥50 successful transactions
- ✅ 99.5% uptime
- Average page load time <3 seconds
- Mobile traffic >40%
- Repeat customer rate >15%

## Implementation Steps

### Immediate Next Steps:

1. **Enable Lovable Cloud (Supabase)**
   - Set up PostgreSQL database
   - Configure authentication

2. **Enable Stripe Integration**
   - Collect Stripe API keys via secure form
   - Configure webhook endpoints

3. **Database Schema Creation**
   - Run migrations for all core tables
   - Set up RLS policies
   - Create storage buckets

4. **Phase 1 Development Begin**
   - Build landing page
   - Implement authentication
   - Create role selection flow

## Project Management

**Communication:**
- Bi-weekly sprint reviews
- Weekly progress updates via project dashboard
- Issue tracking via GitHub Issues
- Documentation in project README

**Version Control:**
- GitHub repository (already connected via Lovable)
- Feature branch workflow
- Code reviews before merging

**Testing Strategy:**
- Unit tests for utility functions
- Integration tests for API endpoints
- E2E tests for critical user flows
- Manual UAT for UI/UX

## Budget Alignment

The implementation leverages Lovable's built-in features:
- ✅ Hosting & SSL: Included in Lovable
- ✅ PostgreSQL database: Lovable Cloud (Supabase)
- ✅ Storage: Lovable Cloud (Supabase Storage)
- ✅ Authentication: Lovable Cloud (Supabase Auth)
- ✅ Stripe Integration: Native Lovable support
- ✅ Deployment: Automated via Lovable

This approach provides enterprise-grade infrastructure while staying within budget.

## Risk Mitigation

**Technical Risks:**
- Risk: Complex Holy Calendar algorithm
  - Mitigation: Develop and test algorithm independently first, extensive unit testing

**Business Risks:**
- Risk: Low artist adoption
  - Mitigation: Seed with 5-10 initial artists, ambassador program

- Risk: Payment processing issues
  - Mitigation: Extensive Stripe testing, sandbox environment, clear error handling

**Security Risks:**
- Risk: Data breach or payment fraud
  - Mitigation: RLS policies, Stripe Radar, security audit, regular updates

---
###API Required
1. #Resend Email
API Key for Resend Email

2. Twilio
3. STRIPE
4. FedEX
