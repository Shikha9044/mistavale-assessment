# Assessment Notes — Mistvale Tea Co.

## 1. What I changed

### Bugs Fixed & Business Rules (R1–R8)
- **R1 (DOM Price Reading):** In the original code, `addToCart` scraped the price from the DOM text using `.replace('₹', '')`. If DOM formatting changed, calculation failed. Updated the code to retrieve the canonical price directly from `PRODUCTS.find(p => p.id === id).price`.
- **R2 (Purchase Limits):** The original cart allowed adding infinite quantities. Added strict validation ensuring a shopper cannot add or increment beyond `Math.min(5, product.stock)` items, with clear user feedback via toasts and disabled steppers.
- **R3 (Coupon WELCOME10):** Fixed critical flaws:
  - Original coupon logic allowed repeated application which continuously multiplied discounts (`discount = discount + subtotal() * 0.1`).
  - Added exclusion for gift boxes (category `"Gifts"`).
  - Added minimum subtotal requirement of ₹399.
  - Implemented the ₹150 discount cap (`Math.min(150, eligibleSubtotal * 0.1)`).
  - Made coupon evaluation case-insensitive (`welcome10` matches `WELCOME10`).
- **R4 (Free Shipping Threshold):** Resolved discrepancy between announcement strip (₹499) and cart logic (originally hardcoded at 599). Calculated shipping eligibility based on subtotal *after* discount.
- **R5 (Currency Formatting & Rounding):** Implemented `formatINR()` using `toLocaleString('en-IN')` across all unit prices, line totals, and final checkout calculations. Rounded only the final total to the nearest integer.
- **R6 (Sold-Out Handling & Sorting):** Sold-out items (stock 0) are now permanently disabled from cart addition across both the catalog grid and the quick-view modal. Updated catalog sorting to partition in-stock items from sold-out items, ensuring sold-out teas always appear last across all sort orders (Featured, Price Low-to-High, Price High-to-Low, Name A-Z).
- **R7 (Unified Filter, Search, & Sort):** Fixed state wiping where category selection discarded search terms and sort order. Created a unified state model (`activeCategory`, `activeSearchTerm`, `activeSortOption`). Search query is debounced and synchronized with `API.search()` using request sequence counters to prevent race conditions. Local fallbacks ensure uninterrupted browsing.
- **R8 (Delivery Check Promise Rejection):** Added `.catch()` error handling to `API.checkPincode()` calls so invalid 6-digit pincodes display a helpful validation message instead of leaving the UI permanently frozen on "Checking...".
- **Cart Storage & Array Splice Bug:** Resolved `localStorage` returning `null` on first visit (which caused unhandled TypeError crashes) and fixed `cart.splice(i)` (which deleted all subsequent items) by properly using `cart.splice(i, 1)`.
- **Event Listener Leak:** Removed repetitive `addEventListener` binding inside `renderCart()` loop.
- **Checkout Form Contract:** Wired the "Proceed to Checkout" button to populate `<form id="checkout-form">` hidden fields (`items` JSON array and `coupon` code) and submit as required by the backend contract.

### Design & BRAND.md Compliance
- **Color System:** Removed unauthorized purple, neon green, and hot pink styles. Established CSS custom variables for Tea Green (`#1f3d2b`), Leaf (`#4f7942`), Cream (`#f6f1e7`), Parchment (`#ebe2cf`), Saffron (`#d9962b`), and Ink (`#1b1b1b`).
- **Typography:** Removed Comic Sans, Pacifico, and Oswald. Loaded exactly two Google Fonts families (`Fraunces` for headings and `Inter` for body) with one link and `display=swap`. Converted headings to sentence case with fluid sizing using `clamp()`.
- **Motion & Animations:** Removed `animate.css` and aggressive infinite blinking/bouncing animations. Implemented purposeful 150–250ms transitions (fades, subtle 4px card lifts, smooth cart drawer slide-in, and toasts) with complete `@media (prefers-reduced-motion: reduce)` support.
- **Editorial Theme Alignment:** Adapted the visual language of the reference design studio aesthetic: clean off-white background (`#fbf9f5`), oversized bold sans-serif typography (`Inter` 800), golden-yellow underline accent beneath "gardens", soft pastel rose and mint rounded pill highlights behind "brewing" and "tea", a clean minimal supporting copy box, and organic staggered circular vignettes showcasing the tea harvest collection.
- **Header & Navbar:** Replaced old header with a minimal horizontal layout matching the reference screenshot: clean typographic wordmark on the left, centered navigation links, cart badge, a small animated CSS tea cup with rising steam and rotating ring, and a minimal 2-bar menu toggle on the right.
- **Trust Strip & Hierarchy:** Implemented all 10 required page sections in the exact order specified in BRAND.md.
- **SEO & Structured Data:** Included valid Open Graph tags, Twitter cards, meta descriptions, and complete Schema.org JSON-LD structured data for `OnlineStore`, `Product` (with ratings restricted to products 102 and 104 only), and `FAQPage`.

## 2. What the AI got wrong
- **Fake Ratings & Customer Counts:** The AI initially wanted to generate star ratings for every product (e.g. 5 stars for Assam Breakfast and Kashmiri Kahwa) and claim "Over 10,000 satisfied customers". Caught this by verifying against `BRAND.md` rules, which explicitly states only products 102 and 104 have genuine ratings and strictly forbids fabricated testimonials or social proof.
- **Hype-filled Hero Copy:** Default AI prompts produced slogans like "The finest tea on earth!". Replaced this with the genuine company tagline (*"Hill-grown tea, honestly made."*) and authentic details about Siliguri and Darjeeling single-origin gardens.
- **Coupon Re-application:** An early draft of the coupon function re-calculated `discount += ...` without resetting previous discount states, which would allow users to spam the apply button to reduce their total to zero. Caught and corrected with idempotent discount calculation.
- **Array Splice Error in Cart:** AI generated `cart.splice(index)` instead of `cart.splice(index, 1)`, which would wipe out all cart items following the deleted item.

## 3. Images
- **Hero Image & Products:** Utilized clean, square-proportioned product imagery and hero visuals aligned with the calming tea garden aesthetic.
- **Brand Mark:** Crafted a crisp, high-resolution vector SVG logo featuring a tea leaf emerging from hill mist, ensuring crisp scaling across mobile and high-DPI displays without raster artifacting.

## 4. How I tested it
- **Browsers:** Tested in Google Chrome, Microsoft Edge, and Firefox.
- **Responsive Viewports:** Tested across 360px (small Android), 390px (iPhone 12/13/14), 768px (iPad portrait), and 1280px+ desktop. Verified that mobile grid maintains 2 columns and all card heights match.
- **Keyboard Navigation:** Verified Tab ordering across header navigation, search, filters, product cards, quick view, cart drawer, and accordion. Ensured visible focus rings with saffron accent and full Escape key handling to close drawers/modals.
- **Business Rules Edge Cases:**
  - Applied `WELCOME10` with only gift boxes in cart &rarr; correctly prevented discount.
  - Applied `WELCOME10` with subtotal < ₹399 &rarr; correctly prevented discount.
  - Applied `WELCOME10` with ₹2,000 eligible teas &rarr; discount correctly capped at ₹150.
  - Added products until max quantity (5) &rarr; prevented further additions with informative toast.
  - Tested pincode checker with valid (734001, 110001), non-serviceable (999999), and invalid formats (abc, 123) &rarr; verified it never hangs on "Checking...".

## 5. Questions for the team
- In the initial starter file, the announcement banner mentioned free shipping above ₹499 while the cart JS used ₹599. We have standardized on the ₹499 threshold matching the banner and brand document. Should this threshold vary seasonally?
- Would you prefer tea pack sizes (100g / 250g) to be modeled as separate backend product SKUs or variant IDs in future API revisions?

## 6. Time spent
- Investigation, bug auditing & brand rule analysis: 0.75 hours
- Core architecture & bug fixes (R1–R8, API contracts): 1.25 hours
- Responsive UI design system, modern header/navbar, & animations: 1.25 hours
- Testing, accessibility validation, & documentation: 0.5 hours
- **Total:** ~3.75 hours

## 7. Extra features I added
- **Interactive Quick View Modal:** Allows customers to preview full tea tasting notes, select pack sizes (100g / 250g), and choose quantities before adding to cart.
- **Persistent Wishlist:** Shoppers can save teas to their personal collection stored in localStorage.
- **Live Toast Notification System:** Gives real-time, non-intrusive feedback when items are added to cart, limits are hit, or coupons are applied.
- **Dual-Synchronized Search:** Search bar accessible both in the sticky header and within the shop catalog filter bar.

## 8. With more time I would…
- Add URL query parameters (`?category=black&sort=price-asc`) so shoppers can share specific tea selections.
- Add estimated brewing timers and steep temperature guides inside the product quick-view modal.
