# MyGrocer: Product & Build Plan (v0 draft)

> One-line pitch: give us your grocery list, and we show you the cheapest way to buy it (one store, several stores, pickup, or delivery), with drive time and coupons included, step by step.

Pricing target: **$5/month or $50/year**, on iOS and Android.

---

## 1. The hard truth first: data is the product

The app is simple to build. **Reliable, current, per-store prices are the hard part.** Everything else depends on them.

| Data we need | Realistic sources | Notes |
|---|---|---|
| Shelf prices per store | **Kroger Developer API** (Kroger, Ralphs, Fred Meyer, King Soopers, etc.: products, prices, store locations); **Walmart affiliate / Walmart.io APIs**; Instacart Developer Platform (shoppable lists, affiliate) | Most other chains (Target, Aldi, Costco, Publix, H-E-B) have **no public price API**. Scraping breaks their terms of service and gets blocked, so we don't build the business on it. |
| Coupons / digital deals | Kroger and Walmart where their APIs expose them; weekly circulars; user-entered | There is no universal coupon API. Start with "sale price" and add clipped-coupon support store by store. |
| Crowdsourced prices | Receipt scanning (photo, then OCR, then prices) from our own users | This is how apps like Basket fill gaps. It gets better as more people use the app, which is a moat. |
| Drive time / distance | Google Maps Routes API or Mapbox | Pay per request. Cache results. |
| Pickup / delivery fees | Store sites, Instacart, Walmart+, Amazon Fresh, Kroger delivery | Mostly published fee tables, so we maintain them as config and refresh periodically. |
| Amazon prices | Amazon Product Advertising API (Associates program) | Access requires an approved Associates account that keeps making sales. Treat as phase 2. |

**Recommendation:** launch the MVP in **one metro area** with **Kroger-family + Walmart** (both have real APIs), plus Instacart links for everything else. Prove people save money and will pay. Then expand.

---

## 2. MVP scope (what v1 does)

1. **Build a list.** Type, voice, or paste ("milk, eggs, 2 lb chicken breast"). We match each item to real products and ask simple questions when it's ambiguous ("Whole or 2%?").
2. **Set preferences once.** Home location, how far you'll drive, how much your time is worth (for example "a 15-minute detour must save me at least $5"), brand flexibility.
3. **See 3 clear options**, ranked:
   - **Cheapest single store**: "Walmart, $62.40, 8 min away"
   - **Best split** (2 stores max in v1): "Kroger + Walmart, $54.10, saves $8.30, adds 11 min"
   - **Easiest**: cheapest pickup/delivery, with fees and tips shown as one total
4. **Step-by-step trip mode.** A checklist per store in aisle-ish order, with directions to the next store and a running "you saved $X" total.
5. **Savings dashboard.** Lifetime and monthly savings. This is what justifies the $5/month.

Phase 2: coupon clipping, receipt scanning, Amazon/online comparison, 3+ store routes, household sharing, price-drop alerts, meal-plan-to-list.

---

## 3. The optimizer (plain English)

For each candidate plan (one store, any pair of stores, each delivery option):

```
total cost = item prices − coupons + delivery/pickup fees + tips
           + (extra drive minutes × user's value of time) + gas estimate
```

Pick the lowest **total cost**, but show the person the dollar and minute breakdown so they decide. With 2 stores max and ~10 nearby stores, that's ~55 combinations, which is instant to compute. Larger routes need a smarter search later.

---

## 4. Tech stack (recommended)

| Layer | Choice | Why |
|---|---|---|
| Mobile app | **React Native + Expo** (TypeScript) | One codebase for iPhone and Android. Expo's cloud build (EAS) means **no Mac required** to ship to the App Store. You can test on your own phone with the Expo Go app from day one. |
| Backend / database / auth | **Supabase** (Postgres) | Cheap, fast to start, scales well enough. |
| Price ingestion | Scheduled backend jobs calling store APIs, results cached in Postgres | Respects API rate limits and keeps the app fast. |
| Maps | Google Maps Platform or Mapbox | Drive times, store locations. |
| Subscriptions | **RevenueCat** on top of Apple/Google in-app purchases | Handles both stores' billing, free trials, and receipts. Apple and Google require their own billing for digital subscriptions. |
| Product matching | Rules + an LLM for fuzzy matching ("2 lb chicken breast" to specific products) | This is where AI helps most. |

### Money math on $5/month
Apple and Google take **15%** for small businesses (under $1M/yr), so you net about **$4.25/month** or **$42.50/year** per subscriber, before API, maps and hosting costs. Strongly consider a **free tier** (e.g. 1 optimized list per week, single store) with paid unlocking multi-store, delivery comparison, and coupons. Nobody pays $5 before they've seen it save them money.

---

## 5. Roadmap

| Phase | Outcome |
|---|---|
| 0. Setup (you + me) | Accounts created, API keys obtained, repo structured |
| 1. Clickable prototype | Full UI flow on your phone with **fake data**. Lets us nail the "anyone can do it" experience before paying for any data. |
| 2. Real data, 1 metro | Kroger + Walmart APIs wired in, real optimizer, drive times |
| 3. Paid beta | Subscriptions via RevenueCat, TestFlight + Google Play internal testing, 20–50 real users |
| 4. Store launch | App Store + Google Play submission, privacy policy, marketing site |
| 5. Expand | Coupons, receipt scanning, Amazon, more chains and cities |

---

## 6. What we need from the owner

**Decisions**
- [ ] Launch city / metro area
- [ ] Which stores matter most there
- [ ] Free tier, or paid-only with a free trial?
- [ ] Company/legal entity name for store listings (an LLC is typical)

**Accounts (only you can create these; they need your identity and payment)**
- [ ] Apple Developer Program: $99/year
- [ ] Google Play Console: $25 one-time
- [ ] Expo account (free)
- [ ] Kroger Developer account (free): developer.kroger.com
- [ ] Walmart affiliate / developer access
- [ ] Google Maps Platform or Mapbox (has free monthly credit)
- [ ] Supabase (free tier to start)
- [ ] RevenueCat (free until meaningful revenue)
- [ ] Later: Amazon Associates, Instacart Developer Platform

API keys go into the environment's secrets, **never** into the code or chat.

---

## 7. Open risks
- **Price coverage.** If we can't price a chain, we can't optimize it. Be upfront in-app about which stores are "live prices" vs "estimated".
- **Competition.** Basket, Flipp, Instacart, and store apps exist. Our angle is the **hand-holding**: one clear answer and a guided trip, not a price database.
- **API terms.** Some store APIs restrict commercial use or require affiliate links. Read each agreement before relying on it.
- **App review.** Apple requires a working subscription restore, a privacy policy, and account deletion.
