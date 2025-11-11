
## Artist Branding System Implementation

### Overview
Create a comprehensive branding system that allows artists to establish their unique brand identity, which will be visible across all their artworks, blog posts, and profile pages.

### 1. Database Schema Changes

**Add branding fields to `artist_profiles` table:**
```sql
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_logo_url TEXT;
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_color_primary TEXT DEFAULT '#8B5CF6';
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_color_secondary TEXT DEFAULT '#EC4899';
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_tagline TEXT;
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_website TEXT;
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_social_links JSONB DEFAULT '{}';
ALTER TABLE artist_profiles ADD COLUMN IF NOT EXISTS brand_slug TEXT UNIQUE;

-- Create index for brand slug lookups
CREATE INDEX IF NOT EXISTS idx_artist_profiles_brand_slug ON artist_profiles(brand_slug);

-- Add constraint to ensure slug is URL-friendly
ALTER TABLE artist_profiles ADD CONSTRAINT brand_slug_format 
  CHECK (brand_slug ~* '^[a-z0-9-]+$');
```

**Brand social links JSON structure:**
```json
{
  "instagram": "https://instagram.com/artist",
  "facebook": "https://facebook.com/artist",
  "twitter": "https://twitter.com/artist",
  "website": "https://artistwebsite.com",
  "tiktok": "https://tiktok.com/@artist"
}
```

### 2. Storage Bucket for Brand Logos

Create a new storage bucket `brand-logos` (public) for artist brand logos, separate from profile avatars.

### 3. Frontend Components Updates

#### A. **New Component: `BrandIdentity.tsx`**
A reusable component to display artist brand elements:
- Brand logo with fallback to avatar
- Brand colors as accent
- Brand tagline
- Social media links with icons
- Website link

#### B. **Update `ArtistCard.tsx`**
- Show brand logo instead of avatar
- Display brand tagline below name
- Apply brand colors as card accent/border
- Add small social media icons

#### C. **Update `src/pages/artist/ArtistProfile.tsx`**
- Display brand logo prominently (separate from personal avatar)
- Show brand tagline under artist name
- Display brand colors as theme accents throughout page
- Add social media links section
- Add website link button
- Use brand slug in URL (e.g., `/artist/@artist-name` instead of UUID)

#### D. **Update `ArtworkCard.tsx`**
- Show small brand logo badge on artwork thumbnails
- Display "by [Brand Name]" with brand colors
- Optional: Apply subtle brand color overlay on hover

#### E. **Update `BlogPostCard.tsx` and `AuthorBio.tsx`**
- Show brand logo for artist-authors
- Display brand tagline
- Add social media links
- Use brand colors for author card accents

### 4. Artist Dashboard - Brand Management

#### **New Page: `src/pages/artist/BrandSettings.tsx`**
A dedicated brand management interface with sections:

**Brand Identity Section:**
- Upload brand logo (image uploader similar to artwork upload)
- Set brand name (defaults to artist_name, can be different)
- Brand tagline input (max 100 chars)
- Brand slug input with real-time validation and preview

**Brand Colors Section:**
- Color pickers for primary and secondary brand colors
- Live preview of how colors appear on cards
- Reset to default button

**Online Presence Section:**
- Website URL input
- Social media links (Instagram, Facebook, Twitter, TikTok, LinkedIn)
- Validation for proper URLs

**Preview Section:**
- Live preview of how brand appears on:
  - Artwork cards
  - Blog post author cards
  - Profile page header

### 5. Hook Updates

#### **Update `src/hooks/useArtistProfile.ts`**
- Include brand fields in SELECT query
- Add validation for brand slug uniqueness
- Add mutation for updating brand settings

#### **New Hook: `src/hooks/useBrandSettings.ts`**
Dedicated hook for brand management:
- `uploadBrandLogo()` - Upload to brand-logos bucket
- `updateBrandColors()` - Update color scheme
- `updateBrandInfo()` - Update tagline, slug, website
- `updateSocialLinks()` - Update social media links
- `checkSlugAvailability()` - Real-time slug validation

### 6. URL Structure Enhancement

**Current:** `/artist/{uuid}`
**New:** `/artist/@{brand-slug}` with fallback to UUID

Update routing to handle both formats:
- Priority: brand_slug if exists
- Fallback: UUID for backward compatibility
- Redirect from UUID to slug for SEO

### 7. Artwork and Blog Post Integration

#### **Artworks Display:**
- Artwork detail page shows brand section (logo, tagline, social links)
- Gallery views show brand badge on thumbnails
- Search/filter by brand name

#### **Blog Posts Display:**
- Author bio section uses brand identity
- Related posts show brand context
- Blog post header can use brand colors as accent

### 8. Public Brand Pages

**New Route:** `/brand/@{brand-slug}` or `/artist/@{brand-slug}`
Enhanced artist profile page showcasing:
- Large brand logo and header with brand colors
- Brand story section (extended bio)
- All artworks gallery
- All blog posts/stories
- Social media feed preview (optional)
- Contact/inquiry section

### 9. SEO and Metadata

Update meta tags to include:
- Brand name in titles
- Brand logo as og:image
- Brand tagline as description
- Structured data for Artist/Organization

### 10. Admin Features

Add to admin dashboard:
- View all artist brands
- Approve brand logos (if moderation needed)
- Feature specific brands on homepage
- Brand analytics (views, engagement)

### Implementation Priority

**Phase 1 (Core):**
1. Database schema updates
2. Brand logo storage bucket
3. BrandSettings page for artists
4. Update ArtistProfile page to display brand

**Phase 2 (Integration):**
5. Update ArtworkCard and ArtistCard components
6. Update BlogPostCard and AuthorBio components
7. Add brand elements to artwork detail pages

**Phase 3 (Enhancement):**
8. Brand slug routing
9. Enhanced SEO/metadata
10. Admin brand management features

### Benefits

1. **For Artists:**
   - Professional brand presence
   - Consistent identity across platform
   - Increased recognition and memorability
   - Social media integration for growth

2. **For Platform:**
   - More engaging content
   - Better SEO with unique brand pages
   - Increased artist investment in platform
   - Professional marketplace appearance

3. **For Buyers:**
   - Easier to identify and remember favorite artists
   - More context about artist's brand story
   - Direct connection to artist's social presence
   - Professional buying experience

This implementation will transform your marketplace into a professional platform where artists can establish and showcase their unique brands. Each artist's work will be instantly recognizable through consistent branding elements, and buyers will have a richer, more engaging experience discovering and connecting with artists.

Would you like me to implement this plan? I can start with Phase 1 (database schema, storage bucket, and basic brand settings page) and then proceed with the integration phases.

Implement the plan
