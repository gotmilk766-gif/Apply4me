# Firebase App Check setup — Primespheres

App Check blocks requests that are not coming from your real web app
(scrapers, stolen API keys, random scripts). Client code is already wired;
you only need Console + a reCAPTCHA key.

---

## 1. Create a reCAPTCHA v3 key

1. Open [Google reCAPTCHA Admin](https://www.google.com/recaptcha/admin).
2. **Create** a new key:
   - Label: `Primespheres Web`
   - Type: **reCAPTCHA v3**
   - Domains:
     - your production domain (e.g. `primespheres.com`)
     - `localhost` (for local testing)
     - any Firebase Hosting domain (`primespheres.web.app`, `primespheres.firebaseapp.com`)
3. Accept terms → **Submit**.
4. Copy:
   - **Site key** (public) → used in `index.html`
   - **Secret key** → used only in Firebase Console (never in client code)

---

## 2. Register the app in Firebase App Check

1. [Firebase Console](https://console.firebase.google.com) → project **primespheres**
2. **Build** → **App Check**
3. Select your **Web** app
4. Provider: **reCAPTCHA v3**
5. Paste the **secret key** from step 1
6. Save

Optional: under **Apps**, confirm the correct `appId` matches  
`1:195779179440:web:925b0550a617e606039c75`.

---

## 3. Put the site key in the client

In `index.html`:

```js
const RECAPTCHA_V3_SITE_KEY = '6Lc...your-site-key...';
const APP_CHECK_DEBUG = false;
```

Redeploy / refresh the site. Console should show:

```
✅ App Check activated (reCAPTCHA v3)
✅ Firebase initialized
```

---

## 4. Monitor before enforcing

1. App Check console → **Metrics** / **APIs**
2. Use the live site for a day or two
3. Confirm legitimate traffic shows **Verified** tokens
4. Investigate any unexplained **Unverified** spikes

**Do not enforce yet** if you still have old clients without App Check.

---

## 5. Enforce (the important step)

In App Check → **APIs** (or product-specific toggles):

| Product   | Enforce when ready |
|-----------|--------------------|
| Cloud Firestore | Yes |
| Cloud Storage   | Yes |
| Authentication  | Optional (recommended) |

After enforce, requests **without** a valid App Check token are rejected
(even if security rules would allow them).

---

## 6. Local development / debug tokens

When `localhost` is detected (or `APP_CHECK_DEBUG = true`), the app sets:

```js
self.FIREBASE_APPCHECK_DEBUG_TOKEN = true;
```

Open DevTools → Console once; Firebase logs a **debug token UUID**.

1. Firebase Console → App Check → **Manage debug tokens**
2. Add that UUID
3. Reload localhost — requests are treated as verified

Never ship a hard-coded debug token in production builds.

---

## 7. Troubleshooting

| Symptom | Fix |
|---------|-----|
| `App Check skipped` in console | `RECAPTCHA_V3_SITE_KEY` is still empty |
| `403` / `app-check-token-error` after enforce | Site key mismatch, domain not registered, or old cached client |
| Works in prod, fails on localhost | Add debug token **or** register `localhost` on the reCAPTCHA key |
| reCAPTCHA badge / errors | Domain must be listed on the reCAPTCHA key; allow third-party cookies if blocked |
| Metrics all “Unverified” | Client not activating App Check, or wrong site key |

---

## 8. Security notes

- Site key is **public** (safe in client code).
- Secret key stays **only** in Firebase Console / server config.
- App Check complements security rules — it does not replace them.
- For stricter bot protection later, consider **reCAPTCHA Enterprise**.

---

## Checklist

- [ ] reCAPTCHA v3 key created (prod + localhost domains)
- [ ] App registered in Firebase App Check with secret key
- [ ] `RECAPTCHA_V3_SITE_KEY` set in `index.html`
- [ ] Console shows App Check activated
- [ ] Metrics show verified traffic
- [ ] Debug token registered for localhost (if needed)
- [ ] Enforce enabled on Firestore + Storage
- [ ] Smoke-test: browse jobs, login, post job, Apply4Me submit
