# NFC Emergency Card

An NFC card (NTAG216) that carries a person's emergency information **on the chip itself**.
When a responder taps it with a phone, a web page opens and shows the data. The page is only a
template. The data travels inside the link, never to a server.

> **Test with fake data only.** A locked card can never be wiped.

---

## How it works

```
 CARD (NTAG216)                          RESPONDER'S PHONE                    YOUR WEBSITE
 ┌──────────────────────────┐   tap     ┌───────────────────────┐  request   ┌───────────────┐
 │ URL record:              │ ───────>  │ opens the link in the │ ────────>  │ static page   │
 │ https://you.github.io/   │           │ browser               │ (no data   │ (empty        │
 │ nfc-card/r/#<data>       │           │                       │  sent)     │  template)    │
 │                          │           │ JavaScript decodes    │ <────────  │               │
 │ Text record (optional):  │           │ the part after "#",   │  index.    └───────────────┘
 │ plain-text fallback      │           │ fills the template,   │  html
 └──────────────────────────┘           │ clears the address bar│
                                        └───────────────────────┘
```

- The part of a link after `#` (the *fragment*) is **never sent to the server**. Your website never sees any patient data.
- Every tap loads the same template and fills it from that card's own data. Three cards from one family give three separate pages. Nothing is shared or cached between them.
- The page uses no cookies, no localStorage, no analytics, no service worker. The data lives in a JavaScript variable until the tab closes.
- Reloading the page shows "Sem dados legíveis". That's intentional: the data was removed from the address bar, so the responder taps the card again.

## Files

| File | What it is | Goes on GitHub? |
|---|---|---|
| `r/index.html` | The card page template (the page that fills itself from the card) | **Yes** |
| `index.html` | Placeholder for your marketing website (replace it) | **Yes** |
| `generate.py` | Turns a profile into the card's URL and checks it fits the chip | No, keep it on your computer |
| `sample-profile.json` | Fake example profile | No (use as a starting point) |
| `README.md` | This file | Optional |

> Never commit real people's profile files to a public repo. Only `index.html` and the `r/` folder
> need to be online. See "Website and card page on the same domain" below.

---

## Step 1: Create the GitHub repository

You need a free account at <https://github.com>. Below, `USERNAME` is your GitHub username.

1. Log in, click the **+** at the top right, then **New repository**.
2. **Repository name:** `nfc-card` (shorter is better, see the note on URL length below).
3. Choose **Public**. GitHub Pages is free only for public repos.
4. Leave the other options alone and click **Create repository**.
5. On the empty repo page, click the **uploading an existing file** link.
6. Drag in `index.html` **and the whole `r` folder** (and `README.md` if you want). Dragging
   folders works in Chrome, Edge and Firefox. If your browser won't take the folder, use
   **Add file > Create new file** instead, type `r/index.html` as the file name (typing the `/`
   creates the folder), paste the contents of that file, and commit.
7. Click **Commit changes**.

**Alternative, from the terminal** (run inside the project folder):

```bash
git init
git add index.html r README.md
git commit -m "NFC card page"
git branch -M main
git remote add origin https://github.com/USERNAME/nfc-card.git
git push -u origin main
```

## Step 2: Turn on GitHub Pages

1. In your repo, open **Settings** (top tab).
2. In the left sidebar, click **Pages**.
3. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
4. Set **Branch** to `main` and the folder to `/ (root)`, then click **Save**.
5. Wait 1 to 2 minutes and refresh the Pages settings. It will show:
   **"Your site is live at `https://USERNAME.github.io/nfc-card/`"**
6. Open that address. You will see the placeholder home page.
7. Now open the card page: `https://USERNAME.github.io/nfc-card/r/`. You should see
   *"Sem dados legíveis neste link…"*. That's correct, because there is no data in the link yet.

Your two addresses are now:

```
https://USERNAME.github.io/nfc-card/        <- your website (placeholder for now)
https://USERNAME.github.io/nfc-card/r/      <- the card page, this is what goes on the cards
```

Always write the trailing slash in the card link (`/r/`), so the phone never needs a redirect.

### A note on URL length

Every character in the link takes space on the chip. `https://` costs only 1 byte on the card
(NFC has a built-in shortcut for it), but the rest counts in full. A shorter repo name, domain
and path (`/r/` rather than `/read/`) mean more room for medical data.

## Step 3: Create the data for a card

Requires Python 3 (nothing to install). Run on your own computer, in the folder with `generate.py`.

1. Copy the sample and edit it. Use **fake data** for now:

   ```bash
   cp sample-profile.json card-a.json
   ```

2. Generate the link (replace `USERNAME` and the repo name):

   ```bash
   python3 generate.py card-a.json --base https://USERNAME.github.io/nfc-card/r/ --text
   ```

   Output looks like this:

   ```
   URL TO WRITE (record type: URL):
   https://USERNAME.github.io/nfc-card/r/#zJY9Bi8IwEIX_...

   TEXT TO WRITE (record type: Text, add it AFTER the URL record):
   NOME: Maria Teste Silva
   SANGUE: A+
   ...

   Bytes used on chip: 584 of 888 (NTAG216), 304 left
   OK, it fits.
   ```

   - `--text` adds the plain-text fallback record. Drop it if you need the space.
   - If it says **DOES NOT FIT**, shorten the notes or remove fields.

3. **Check the page before touching the card.** Send the long `URL TO WRITE` link to yourself
   (email, WhatsApp, notes app), open it on your phone and confirm the data shows up and the
   address bar ends up clean.

4. For three test cards, make `card-a.json`, `card-b.json`, `card-c.json` with different names
   and run the generator for each.

To double check a link, you can decode it again:

```bash
python3 generate.py --decode "https://USERNAME.github.io/nfc-card/r/#zJY9..."
```

### Profile fields

The card page and the plain-text fallback are in European Portuguese. Each field in the profile
JSON has a short key (to save bytes) and shows up under a Portuguese label:

| Key | Shown on the page as | Section |
|---|---|---|
| `n` | Nome | Pessoa |
| `b` | Data de nascimento | Pessoa |
| `s` | Sexo | Pessoa |
| `bt` | Grupo sanguíneo | Dados médicos |
| `al` | Alergias | Dados médicos |
| `md` | Medicação | Dados médicos |
| `cd` | Condições médicas | Dados médicos |
| `nt` | Notas | Dados médicos |
| `hi` | Seguradora | Seguro de saúde |
| `hp` | N.º da apólice | Seguro de saúde |
| `hn` | N.º de beneficiário | Seguro de saúde |
| `ins` | Seguradora | Seguro automóvel |
| `pol` | N.º da apólice | Seguro automóvel |
| `pl` | Matrícula | Seguro automóvel |
| `ice` | Contactos de emergência | list of `{"r": relação, "n": nome, "p": telefone}` |

To change the wording or the language, edit `SECTIONS` in `r/index.html` and `LABELS` in
`generate.py`. Accented letters take 2 bytes each on the card.

Leave out any field you don't want shown. Every byte counts, so keep it to the essentials.
The photo does **not** fit on the chip: print it on the card or holder.

## Step 4: Write the data to the card

Install **NFC Tools** (free, iOS and Android) by wakdev. Use an **Android phone for writing and
locking** if you have one, since it exposes the full NFC features.

1. Open **NFC Tools**, go to the **Write** tab, tap **Add a record**.
2. Choose **URL / URI**.
   - If there is a protocol drop-down, pick **https://** and paste everything *after* `https://`.
   - Otherwise paste the whole link.
3. (Optional) Tap **Add a record** again, choose **Text**, and paste the plain-text block.
   Make sure it comes **after** the URL record.
4. Look at the size shown at the bottom of the screen. It should be close to the "Bytes used" the
   generator reported, and below 888.
5. Tap **Write**, then hold the card against the back of the phone (near the top on most phones)
   until it confirms.

## Step 5: Verify the card

Do all of these **before locking**.

1. **Read it back.** In NFC Tools, **Read** tab, tap the card. Confirm the URL record (and text
   record) match what you wrote.
2. **Check with NXP TagInfo** (free, by NXP). It shows the chip type, memory used and lock status.
   A new card should show as unlocked.
3. **Tap it like a responder would.** Close all apps, unlock the phone, and tap the card.
   - **Android:** the page opens in the browser, or a prompt appears.
   - **iPhone:** a banner appears at the top. Tap it to open the page.
4. **Check the page:**
   - All the fields you entered appear.
   - The address bar shows only `…/nfc-card/r/` with no data after it.
   - The phone number link opens the dialler.
5. **Test on both Android and iPhone**, and on a second phone model if you can.
6. **Multi-card test.** Tap card A, then B, then C, one after another. Each must show its own data.
7. **Offline test.** Turn on airplane mode and tap the card. The page won't load, but if you
   added the text record, check what the phone shows for it. This is your fallback.

If an iPhone only shows the plain text and not the page (or the reverse), the order of records
matters. Rewrite with the **URL record first**.

## Step 6: Lock the card

> **Locking is permanent.** There is no undo. Do it on **one** test card first, and only after
> Step 5 passed on every device.

Two options (NFC Tools, **Other** tab):

| Option | What it does | Use it when |
|---|---|---|
| **Lock tag** | Card becomes permanently read-only. | Final cards you ship. Nobody can ever overwrite them, including you. |
| **Set password** | Writing now requires a password. Reading stays open. | You want to be able to update the card later. Write the password down: if you lose it, the card can't be changed again. |

**Prove the lock works:**

1. Lock (or set a password on) one card.
2. Try to write something new to it. **It must fail.**
3. Read it again. The original data must still be there.
4. With a phone that does **not** know the password, tap it. It must still open the page.
   (Responders need to read it freely.)

**Order for a real card:** finalise data → generator says it fits → write → read back and verify
on two phones → lock or set password → try to overwrite and confirm it fails.

---

## Website and card page on the same domain

You can host the marketing site and the card page together. Same repo, same domain, two places:

```
yourdomain.com/            <- marketing website  (index.html at the repo root)
yourdomain.com/r/          <- card page          (r/index.html), this is what the cards open
```

Static hosting serves each folder's `index.html`, so no extra setup is needed. The card link
becomes `https://yourdomain.com/r/#<data>`.

Things to keep in mind:

- **Keep `r/index.html` separate and untouched.** Build the marketing site however you like at the
  root (any framework, images, fonts, analytics), but don't change the card page unless you mean to.
- **Analytics and third-party scripts only on the marketing pages, never on `/r/`.** They could
  capture the full URL, including the data after `#`. The card page's built-in Content-Security-Policy
  blocks them there anyway, as a safety net.
- **Same origin means shared trust.** Anything that can change files in this repo can change what
  the cards show. Protect the GitHub account with 2-factor authentication and treat changes to
  `r/` as sensitive.
- Don't link to `/r/` from the marketing site with real data in the link. A demo link should use
  fake data only.
- **Stricter isolation option:** serve the card page from its own subdomain (for example
  `r.yourdomain.com`) with a separate repo. It's cleaner, but the longer link costs a few
  more bytes on every card.
- **If you change the path later, old cards break.** Pick `/r/` now and never move it. The cards
  already written will keep pointing there.

## Moving to a real domain

For the tests, `USERNAME.github.io` is fine. For cards you give to other people, buy a domain
**first**, because the link written on a card can't be changed once the card is locked.

- **GitHub Pages does not sell domains.** You buy one at a registrar and point it at GitHub
  Pages. In your repo go to **Settings > Pages > Custom domain**, enter the domain, and tick
  **Enforce HTTPS**. At your registrar, create a `CNAME` record pointing to `USERNAME.github.io`
  (for a bare domain, use the A records listed in GitHub's guide).
- **Netlify** can sell you a domain directly and host the page in the same dashboard.
- **Cloudflare Registrar** sells domains at wholesale cost, and Cloudflare Pages can host the page.
- Pick a **short domain**. Each character costs a byte on every card.
- **Turn on auto-renew and register for several years.** If the domain ever lapses, someone else
  could register it and serve a malicious page to every card already in circulation. This is the
  biggest long-term risk of the whole design. Keep it renewed for as long as any card exists.
- **Headers:** GitHub Pages can't set custom HTTP headers. That is fine, since the page holds no
  data, so caching it is harmless. If you want `Cache-Control: no-store`, host on Netlify or
  Cloudflare Pages and add a `_headers` file.

Links:
[GitHub: about custom domains](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages) ·
[Netlify: register and buy a domain](https://docs.netlify.com/domains/configure-domains/register-and-buy-a-domain/) ·
[Cloudflare Registrar](https://www.cloudflare.com/application-services/products/registrar/buy-com-domains)

## Security and privacy notes

- **Open read, by design.** Anyone who can tap the card can read what's on it, like a medical
  alert bracelet. State this clearly to buyers and let them choose which fields to include.
- **Medical data is sensitive.** In the EU it is health data under GDPR. Before selling cards
  with real data, get proper advice on your obligations, including who writes the data on the
  cards.
- The page treats all card data as untrusted text (it never inserts it as HTML), strips the phone
  link to digits and `+` only, and ships a strict Content-Security-Policy. Anyone can craft a link
  to your page, so keep it that way if you edit the code.
- Don't add analytics, fonts, CDNs or any third-party script. Many capture the full URL,
  including the data after `#`.
- Browsers may still record the original link (data included) in their history before the page
  clears it. Describe it honestly: *"nothing is stored by our service and the page saves nothing
  on the phone"*, not *"nothing is stored anywhere"*.
- **Browser support:** the page needs a recent browser (roughly Chrome 103, Safari 16.4,
  Firefox 113 or newer) to decompress the data. Older phones show "Sem dados legíveis", which is
  why the plain-text record on the card matters.
- **MIFARE Classic cards are not suitable.** If TagInfo shows MIFARE Classic rather than NTAG,
  keep it for experiments only. Buy NTAG216 for the product.

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| Page shows "Sem dados legíveis" after tapping | The link on the card is wrong or truncated. Read the card in NFC Tools and compare to the generator output. Also check you didn't reload the page. |
| Page is blank or 404 | Pages isn't live yet (wait 2 minutes), or the link doesn't match your repo name or the `/r/` path. Open `…/nfc-card/r/` in a browser to check. |
| Write fails with "not enough space" | Over 888 bytes. Run `generate.py` without `--text`, or shorten fields. |
| Link has `https://https://` | You pasted a full link while the protocol drop-down was already on `https://`. Remove one. |
| iPhone doesn't react | Hold the card near the top back edge of the phone, unlock the screen, and make sure the URL record is first. |
| Can't write to a card anymore | It has been locked or password-protected. Locked cards are permanent. |
