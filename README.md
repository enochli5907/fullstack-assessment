# Bug Fixes Summary

This document outlines the bugs I found and fixed in the StackShop eCommerce application.

---

## Bug 1: App Crashes on Image Load

**The Problem:**
The app would crash with an error message about invalid image hostnames whenever certain products tried to load. Specifically, images from `images-na.ssl-images-amazon.com` weren't configured, making those products completely inaccessible.

**The Fix:**
Added the missing hostname to the Next.js image configuration in `next.config.ts`.

```typescript
images: {
  remotePatterns: [
    {
      protocol: 'https',
      hostname: 'm.media-amazon.com',
    },
    {
      protocol: 'https',
      hostname: 'images-na.ssl-images-amazon.com',  // Added this
    },
  ],
}
```

**Why This Approach:**
This is the standard Next.js way to handle external images. It's a simple config change that doesn't affect any existing functionality.

**Files Changed:**
- `next.config.ts` - Added new hostname to remotePatterns

---

## Bug 2: No Product Prices Displayed

**The Problem:**
Prices weren't showing up anywhere - not on the product cards and not on the detail pages. The data was there in the JSON, but the TypeScript interfaces were missing the `retailPrice` field, so it wasn't being rendered.

**The Fix:**
Added `retailPrice: number` to the Product interfaces in three files, then displayed the price with proper formatting (`$XX.XX`) in both the listing and detail views.

**Why This Approach:**
Keeping TypeScript interfaces in sync with the actual data is important for type safety. I used `.toFixed(2)` to ensure consistent currency formatting and placed the price prominently so users can see it immediately.

**Files Changed:**
- `lib/products.ts` - Added retailPrice to Product interface
- `app/page.tsx` - Display price on product cards
- `app/product/page.tsx` - Display price on detail page

---

## Bug 3: Clear Filters Doesn't Reset Dropdown

**The Problem:**
When you clicked "Clear Filters" after selecting a category, the filters would clear but the dropdown would still show the selected category instead of the "All Categories" placeholder. This was confusing because it looked like a filter was still active.

**The Fix:**
Changed how the category state is managed - using an empty string (`""`) instead of `undefined` to represent "no selection". Updated all the related checks throughout the code to use `!== ""` instead of checking for undefined.

**Why This Approach:**
The dropdown component needs an empty string (`""`) to display the placeholder text, not `undefined`. By using empty string consistently, the dropdown correctly shows "All Categories" when no category is selected.

**Files Changed:**
- `app/page.tsx` - Updated state management and conditional checks

---

## Bug 4: Subcategory Filter Shows Wrong Options

**The Problem:**
When you selected a category like "Headphones", the subcategory dropdown would show ALL subcategories from ALL categories instead of just the ones for Headphones. This made the filter pretty useless.

**The Fix:**
Added the `category` query parameter to the subcategories API call. The backend already supported filtering - the frontend just wasn't passing the parameter.

```typescript
// Before:
fetch(`/api/subcategories`)

// After:
fetch(`/api/subcategories?category=${encodeURIComponent(selectedCategory)}`)
```

**Why This Approach:**
One-line fix that leverages existing backend functionality. Used `encodeURIComponent()` to handle category names with special characters safely.

**Files Changed:**
- `app/page.tsx`

---

## Bug 5: Massive URLs with Product Data

**The Problem:**
Clicking on a product would create URLs over 1000 characters long because the entire product object was being serialized to JSON and stuffed into the query string. This exposed all the data in the address bar and could hit browser URL length limits.

Example: `/product?product={"stacklineSku":"E8ZVY2BP3","title":"Amazon Kindle...",...}`

**The Fix:**
Changed the approach to only pass the product SKU in the URL, then fetch the full product data on the detail page.

1. Updated the Link to use: `/product?sku=E8ZVY2BP3`
2. Modified the product page to read the SKU from query params
3. Look up the product from the imported JSON data

**Why This Approach:**
The SKU is a unique identifier for each product, making it perfect for use in the URL. This keeps URLs clean and short while still allowing me to look up the full product data from the JSON file using that SKU. It's a standard pattern that's simple and doesn't expose unnecessary data in the URL.

**Files Changed:**
- `app/page.tsx` - Updated Link href
- `app/product/page.tsx` - Read SKU from query params and fetch product

---

## Bug 6: Only 20 Products Accessible

**The Problem:**
The app has 500 products but only showed the first 20 with no way to see more. Even when using filters, you could only see the first 20 results of that filtered set. This meant most products were completely inaccessible.

**The Fix:**
Implemented a "Load More" button with offset-based pagination. Also added optional chaining (`?.`) to safely handle products that don't have images defined, which prevented crashes when loading products at higher offsets.

1. Added state for `offset`, `total`, and `loadingMore`
2. Modified the initial fetch to include `offset=0` and capture the total count
3. Created a `loadMore()` function that increments the offset by 20 and appends new products
4. Added a button that only shows when there are more products to load
5. Updated the product count to show "Showing X of Y products"
6. Added optional chaining for `imageUrls` to handle missing image data gracefully

**Why This Approach:**
"Load More" is a common, user-friendly pattern. It's simpler than traditional pagination with page numbers and works well with the existing API's offset parameter. Loading 20 at a time keeps performance good without overwhelming the user. The optional chaining prevents crashes when some products in the dataset don't have image URLs.

**Files Changed:**
- `app/page.tsx` - Added pagination state, Load More button, and optional chaining for images
