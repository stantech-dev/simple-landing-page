# Gvoi early-access sign-up page

A single-page early-access sign-up for Gvoi, a speech-to-notes and text-to-speech tool for East African languages. It collects a name and email (plus main use and language) with consent, and stores them in Supabase so you can measure interest before launch.

## How it works

- Static page: HTML, CSS and JavaScript only. No backend.
- The browser sends each sign-up straight to Supabase using the **publishable** key.
- Row-level security is on, with an **insert-only** policy for the public. Nobody can read the data with the public key. You review signups in the Supabase dashboard.

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole page, including the Terms and Privacy pop-ups and the `CONFIG` block |
| `logo.png` | Header logo and browser tab icon |
| `waitlist.sql` | Creates the `waitlist_signups` table |
| `waitlist.py` | Not used. An optional Flask endpoint, only if you later want server-side rate limiting |

## Setup

1. **Create the table.** In Supabase, open SQL Editor and run `waitlist.sql`.
2. **Add the insert policy and make both dropdowns required** (run as a new query):
   ```sql
   grant insert on public.waitlist_signups to anon;

   create policy "public can sign up"
     on public.waitlist_signups
     for insert
     to anon
     with check (consent = true);

   alter table public.waitlist_signups
     alter column use_case set not null,
     alter column language set not null;
   ```
3. **Fill in the `CONFIG` block** near the bottom of `index.html`: `PRODUCT_NAME`, `OWNER_NAME`, `CONTACT_EMAIL`, `SUPABASE_URL`, `SUPABASE_KEY`.
4. **Test.** Open the page, sign up once, and check the row in Table Editor. Check the browser console (F12) for red errors.
5. **Deploy.** Upload `index.html` and `logo.png` together to Netlify, Cloudflare Pages, Vercel or GitHub Pages.

## Reviewing signups

Run in the SQL Editor, or export CSV from Table Editor:

```sql
select count(*) from public.waitlist_signups;
select source, count(*) from public.waitlist_signups group by source order by 2 desc;
select use_case, language, count(*) from public.waitlist_signups group by 1, 2 order by 3 desc;
```

Track channels with links like `yourpage.com/?src=whatsapp`. The `src` value is saved with each signup.

Delete someone on request:

```sql
delete from public.waitlist_signups where lower(email) = lower('person@example.com');
```

## Rules to keep

- **Never put the secret or `service_role` key in the page.** Only the publishable (or legacy `anon`) key goes in `index.html`.
- **Keep the wording honest.** The product isn't live yet, so the page should say "launching soon" and "early access", not imply a working account.
- **Consent is required** in the form and enforced by the database. The checkbox text must match what you actually do with the data.
- If you edit the footer Contact link, keep the script and the HTML in agreement. A missing element makes the script crash and stops the form from working.

## Before real traffic

- [ ] Check whether you must register with Kenya's ODPC as a data controller, and confirm cross-border storage rules (Supabase servers may be outside Kenya).
- [ ] Read and adjust the Terms and Privacy text, including the retention period and owner details.
- [ ] Make sure text on the page is dark enough to read on the light blue background, and the button label is readable.
- [ ] Add a demo audio clip under the hero once you have a sample voice you're happy with.
- [ ] Send a test signup from a phone.

## Known limits

- Anyone technical could send junk sign-ups with the public key. The unique-email rule and column checks limit the damage. If spam appears, add Cloudflare Turnstile.
- A duplicate email returns an "already signed up" message, which reveals that the address is on the list. Acceptable for a waitlist.
