# TS Sport website

A mobile-friendly storefront for **TS Sport**, a South African shop selling soccer jerseys and football boots. Customers browse the range, pick a size and enquire on WhatsApp or by email. There is deliberately no cart, checkout, payment or customer account.

Products are managed in **Sanity Studio** (no code needed to add products, change prices, sizes or photos) and the site is hosted on **Netlify**, which rebuilds automatically whenever you publish, update or unpublish a product.

---

## 1. Open this first

Double-click **`site/index.html`**. It opens in your browser straight from your computer, with no install or internet connection needed (the fonts fall back to built-in ones when you are offline).

What you see is the **preview** version. It uses eight sample products (`site/catalogue.js`) with made-up prices and original placeholder illustrations. An orange banner says so, each card has a **Sample** label, and the WhatsApp/email enquiry buttons are switched off on sample items so nobody can enquire about a product that doesn't exist. Your real products from Sanity replace these automatically on the live site.

---

## 2. What's in the folder

```
TS-Sport-Website/
├── site/                     ← the storefront (open index.html)
│   ├── index.html            page layout and wording
│   ├── styles.css            colours, fonts, spacing (the design)
│   ├── app.js                filters, search, product pop-up, WhatsApp/email links
│   ├── config.js             ← your WhatsApp number and email address
│   ├── catalogue.js          sample products for the local preview
│   └── assets/               logo and placeholder illustrations
├── studio-files/             ← copy into your Sanity Studio
│   ├── schemaTypes/product.js    the product form (name, price, sizes, photos …)
│   ├── schemaTypes/index.js      registers the product form
│   └── sanity.config.example.js  reference only
├── scripts/
│   ├── build.mjs             builds the live site from Sanity into dist/
│   └── sample-data/          fake Sanity data for testing the build offline
├── tests/storefront.test.mjs optional automated browser checks
├── netlify.toml              tells Netlify how to build and publish
├── package.json              build shortcuts (no dependencies to install)
└── README.md                 this file
```

---

## 3. Change the contact details, design and wording

**Contact details — `site/config.js`.** Every WhatsApp and email button reads from this one file.

| Setting | Current value | Notes |
|---|---|---|
| `whatsappDisplay` | `072 068 0392` | What customers see on the page |
| `whatsappNumber` | `27720680392` | Used in the link: `27` + number without the first `0`, no spaces |
| `email` | `tshegomahlangu18@gmail.com` | Used for all email buttons |
| `enquiryGreeting` | `Hi TS Sport, I would like to enquire about` | Start of the prefilled message |

**Design — `site/styles.css`.** The colours and fonts are at the very top, inside `:root`. Change `--lime` to restyle every accent, `--bg` for the charcoal background, `--font-head` for headlines. The fonts (Barlow Condensed and Inter) are loaded from Google Fonts in `index.html`.

**Wording — `site/index.html`.** The hero headline ("Gear Up. Game On."), the "How to order" steps and the footer are plain text in this file. Search for the words you want to change.

**Logo — `site/assets/logo.svg`.** Replace it with your own logo file using the same name (SVG or PNG; if PNG, update the two `logo.svg` references in `index.html`).

---

## 4. Go live (one-time setup, about 30 minutes)

You'll need free accounts at **sanity.io**, **github.com** and **netlify.com**, plus **Node.js 18 or newer** installed on your computer (download the LTS version from nodejs.org).

### Step A — Create your Sanity Studio

1. Open a terminal (Command Prompt / PowerShell on Windows, Terminal on Mac) in a folder of your choice and run:
   ```
   npm create sanity@latest
   ```
2. Log in when asked, choose **Create new project**, name it `TS Sport`, accept the default dataset configuration (**production**, public), and choose the **Clean project with no predefined schema types** template. Choose **No** for TypeScript.
3. When it finishes, open the new Studio folder. Inside it is a folder called **`schemaTypes`**.
4. **Copy `studio-files/schemaTypes/product.js` and `studio-files/schemaTypes/index.js` into that `schemaTypes` folder**, replacing the `index.js` that's already there. (If your Studio made an `index.ts` instead, delete it and use the `index.js` from this project.)
5. In the Studio folder run `npm run dev` and open `http://localhost:3333`. You should see **Product** in the left menu.
6. To use the Studio from any device (including your phone), run `npx sanity deploy` and pick a name. Your Studio will live at `https://your-name.sanity.studio`.

**Find your project ID:** it's in your Studio's `sanity.config.js` (`projectId: '…'`) and at sanity.io/manage under your project. It looks like `abc123de`. You'll need it in Step C.

**Keep the dataset public.** The website reads published products without any password or token. Public means anyone can *read* published products (which are on your website anyway); only you can edit. Drafts are never public. You can check this at sanity.io/manage → your project → Datasets.

### Step B — Put this project on GitHub

1. Create a new repository on github.com (for example `ts-sport-website`). It can be private.
2. Upload the contents of this folder (everything except `dist/` and `node_modules/` if they exist). The easiest way without the command line is **Add file → Upload files** on GitHub, or use GitHub Desktop.

### Step C — Connect Netlify

1. On netlify.com choose **Add new site → Import an existing project**, pick GitHub and select your repository.
2. Netlify reads `netlify.toml` and fills in the build settings for you: build command `node scripts/build.mjs`, publish directory `dist`. Leave them as they are.
3. **Set your project ID.** Before deploying (or straight after, then redeploy), go to **Site configuration → Environment variables → Add a variable** and add:
   - Key **`SANITY_PROJECT_ID`**, value your project ID (e.g. `abc123de`).
   - Only if your dataset is *not* called `production`: key **`SANITY_DATASET`** with its name.
4. Click **Deploy**. When it says *Published*, open the site link. You'll see your real Sanity products (or "New stock coming soon" if you haven't added any yet). The orange sample banner will be gone.

> **Safety rule built in:** a production deploy will never show the sample products. If `SANITY_PROJECT_ID` is missing, or Sanity can't be reached during the build, the build stops with a clear message and Netlify keeps your previous live site online.

### Step D — Rebuild automatically when products change

This connects Sanity to Netlify so the website updates within a minute or two of you publishing, editing or unpublishing a product.

1. **Create a Netlify build hook.** In Netlify go to **Site configuration → Build & deploy → Continuous deployment → Build hooks → Add build hook**. Name it `Sanity`, branch `main`, save, and **copy the URL** it gives you (it starts with `https://api.netlify.com/build_hooks/`). Keep it private: anyone with it can trigger builds.
2. **Create a Sanity webhook.** Go to **sanity.io/manage → your project → API → Webhooks → Create webhook** and fill in:
   | Field | Value |
   |---|---|
   | Name | `Rebuild Netlify site` |
   | URL | the Netlify build hook URL you copied |
   | Dataset | `production` |
   | Trigger on | ✅ Create ✅ Update ✅ Delete |
   | Filter | `_type == "product"` |
   | Projection | leave empty |
   | Status | Enabled |
   | HTTP method | `POST` |
   | Trigger on drafts | leave **off** (only published changes rebuild the site) |
3. Save. Unpublishing a product counts as a *Delete* of the published version, so it triggers a rebuild too.

---

## 5. Add your first real jersey and boot

Open your Studio (`npm run dev` or your `.sanity.studio` address) and click **Product → + (Create)**.

**Your first jersey**

1. **Product name:** e.g. `2026 Home Jersey`.
2. **Category:** choose **Jerseys**.
3. **Product code (optional):** your own stock code, e.g. `TSJ-001`. Customers can search for it.
4. **Selling price (R):** numbers only, e.g. `649`. Leave it blank to show **Request a price**.
5. **Sizes available:** type `S`, press Enter, type `M`, press Enter, and so on.
6. **Availability:** In stock, Low stock or Out of stock.
7. **Main photo:** drag in your best photo. Click the **crop icon** on the photo to crop it and to drag the **focal point** (the circle) onto the jersey's badge or centre — the site keeps that part in view on square product cards. Add a short **Photo description**.
8. **Extra photos:** add back, detail and on-model photos. Customers can swipe through them.
9. **Description** and, optionally, switch on **Featured** to show it first.
10. Click **Publish** (bottom right). With the webhook set up, the live site updates in about a minute.

**Your first boot:** same steps, but choose **Boots** and use sizes like `UK 6`, `UK 7`, `UK 8`. A side-view main photo and a soleplate photo work well.

**Use your own photos** (or ones the supplier has given you permission to use) rather than copying photos or descriptions from other retailers' websites.

### How the product rules work

| You do this in Sanity | The website does this |
|---|---|
| Leave **price** blank | Shows **Request a price**; the WhatsApp message asks for the price |
| Set **Out of stock** | Product stays visible with a greyed photo, but has **no** enquiry buttons |
| **Unpublish** or delete a product | Removed from the website after the next build (automatic with the webhook) |
| Edit and **Publish** | Changes go live after the next build |
| Save without publishing (draft) | Nothing changes on the website |
| Add only **one** size | That size is preselected for the customer |
| Switch on **Featured** | Shown first in the catalogue |

When a customer taps **Enquire on WhatsApp**, WhatsApp opens a chat with **072 068 0392** with a ready-made message such as:

> Hi TS Sport, I would like to enquire about the 2026 Home Jersey (code TSJ-001) in size M. Price shown on the website: R 649. Is it available?

If the product has sizes, the customer must choose one first, so every message includes the size.

---

## 6. Build commands (for developers)

| Command | What it does |
|---|---|
| `node scripts/build.mjs` | Builds `dist/`. With `SANITY_PROJECT_ID` set it uses your live Sanity products; without it (on your own computer) it copies the demo preview |
| `node scripts/build.mjs --sample` | Builds `dist/` from `scripts/sample-data/sanity-sample.json`, so you can test the Sanity path offline. Refused on production deploys |
| `SANITY_PROJECT_ID=abc123de node scripts/build.mjs` | Test a real Sanity build on your computer (Mac/Linux; on Windows PowerShell use `$env:SANITY_PROJECT_ID="abc123de"; node scripts/build.mjs`) |

After building, open `dist/index.html` to preview the result.

**Optional automated tests** (real browser checks of filters, search, gallery, sizes, WhatsApp/email links, demo and out-of-stock rules, mobile layout, and the sample Sanity build):

```
npm install --no-save playwright
npx playwright install chromium
node scripts/build.mjs --sample
node tests/storefront.test.mjs
```

---

## 7. What has been tested, and what needs your live accounts

**Tested and passing (26 automated browser checks in Chrome, plus build checks):** the local preview opens straight from `site/index.html` with no errors; Browse Jerseys / Browse Boots filter and scroll to the catalogue; header links, All Gear / Jerseys / Boots chips and `index.html#boots` links work; search matches names, product codes and sizes and combines with the category filter; empty results show a message and a reset button; the photo gallery works with arrows, thumbnails and the keyboard; one size is selected at a time; the WhatsApp link goes to `wa.me/27720680392` with the product name, code, chosen size and price prefilled, and clicking it opens that address; the email button goes to `tshegomahlangu18@gmail.com` with the product and size in the subject; sample products and out-of-stock products have no enquiry buttons; no sideways scrolling at 390 px and 320 px widths; product text is always shown as text (never run as code). The build was tested with sample Sanity data (crop and focal-point image URLs, blank price, decimal price, duplicate sizes cleaned up, drafts and unnamed items excluded) and refuses to publish demo or sample data on a production deploy. The product schema passes Sanity's own schema validator, and the product query was run with Sanity's GROQ engine against the sample data.

**Needs your live accounts (not possible to test without them):** fetching products from your real Sanity project, Sanity's image CDN serving your actual photos, a real Netlify deploy, and the Sanity webhook triggering the Netlify build hook. Nothing has been deployed.

---

## 8. Troubleshooting

- **Netlify build says "SANITY_PROJECT_ID is not set".** Add it under Site configuration → Environment variables, then Deploys → Trigger deploy → Deploy site.
- **Build says "Sanity answered 401/403/404".** Check the project ID has no spaces, the dataset name is right, and the dataset is public.
- **Site shows "New stock coming soon".** The build worked but found no *published* products. Open the product in Studio and click Publish.
- **I published but the site didn't change.** Check Netlify → Deploys for a new build. If none started, check the webhook URL and that it's Enabled in sanity.io/manage → API → Webhooks (it has an attempts log).
- **WhatsApp opens the wrong number.** Fix `whatsappNumber` in `site/config.js`: `27` followed by the number without the leading `0`, digits only.
