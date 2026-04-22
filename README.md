# PitchBox website pages

Three static HTML pages that need to live on `divergeix.com`:

| File | Public URL | Purpose |
|------|------------|---------|
| `pricing.html` | `https://divergeix.com/pitchbox/pricing` | Public pricing page; linked from the extension's "View pricing page" button. |
| `checkout.html` | `https://divergeix.com/pitchbox/checkout` | Razorpay Checkout launcher. The extension opens this in a new tab with order details in URL params. |
| `success.html`  | `https://divergeix.com/pitchbox/success`  | Post-payment thank you. Tells the user to return to the extension and click "Refresh plan". |

## How to deploy

These are plain static HTML — drop them anywhere that serves your `divergeix.com` site (your existing webhost, Netlify, Vercel, GitHub Pages, Azure Static Web Apps).

Whatever your hosting is, the URL pattern needs to match the three rows above (or you'll need to update `BILLING_CONFIG.pricingPageUrl` in `extension/src/lib/billing.ts` and rebuild the extension).

If your host requires file extensions in URLs (Apache, plain nginx), either:
- Configure rewrites: `/pitchbox/pricing → /pitchbox/pricing.html`
- Or rename: `pricing.html → /pitchbox/pricing/index.html`

## How the flow connects

```
[Extension Settings → Upgrade to Pro]
  ↓ POST /api/payment/create-order (auth'd)
[Backend creates Razorpay order, returns orderId + keyId]
  ↓ chrome.tabs.create(checkout.html?orderId=…&keyId=…&amount=…&email=…)
[checkout.html loads Razorpay Checkout.js, opens widget]
  ↓ User pays
  ↓ Razorpay → fires webhook → backend marks user pro server-side
  ↓ Razorpay → returns success to checkout.html
  ↓ Redirect to success.html
[User reads instructions, returns to extension]
  ↓ Settings → Refresh plan from server (re-login)
[Extension fetches new JWT with plan=pro]
```

The website pages do NOT need to call `/api/payment/verify` directly — the webhook handles the server-side upgrade. This is by design: the user's JWT lives in the extension and shouldn't be passed through URL params to a hosted page.

## Branding / customization

The pages use inline CSS that matches the extension's dark theme. To rebrand for divergeix.com:
- Update the `<title>` and meta description
- Swap the logo mark (the `P` in a purple square)
- Adjust the `--accent` color (currently `#6366f1` indigo)

Keep `checkout.html` mostly untouched — the Razorpay Checkout flow depends on the URL parameter format.

## Testing locally

```bash
# From the website/pitchbox folder:
python3 -m http.server 8000
```

Then visit:
- http://localhost:8000/pricing.html
- http://localhost:8000/checkout.html?orderId=order_test&keyId=rzp_test_xxx&amount=49900&email=test@example.com
- http://localhost:8000/success.html?paymentId=pay_test&email=test@example.com
