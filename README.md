# franklin.is

Personal contact card at [franklin.is](https://franklin.is). Static, hand-written,
no build step and no framework.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire page. Copy is in Icelandic (`lang="is"`). |
| `styles.css` | Single stylesheet. Palette lives in `:root` custom properties. |
| `favicon.svg` | Turtle icon, also the basis for the social card. |
| `og.svg` | Source for the Open Graph image — edit this, not the PNG. |
| `og.png` | 1200×630 social preview, generated from `og.svg`. |

## Local preview

`index.html` links the stylesheet as `/styles.css` (root-relative), which is correct
for Netlify but means opening the file directly gives an unstyled page. Serve it instead:

```sh
python3 -m http.server 8000
```

Then open <http://localhost:8000>.

## Regenerating the social card

Edit `og.svg`, then re-render. Requires `rsvg-convert` (`brew install librsvg`):

```sh
rsvg-convert -w 1200 -h 630 -o og.png og.svg
```

Keep it 1200×630. `og.png` must be committed — social scrapers fetch it from the live
site, and they do not render SVG, which is why the PNG exists at all. LinkedIn caches
previews aggressively; force a refresh with the
[Post Inspector](https://www.linkedin.com/post-inspector/).

## Swim counter

The card shows how many times I have been swimming in the current calendar
year. The number is fetched at page load, never hardcoded — if the fetch
fails the sentence falls back to "Ég veit ekki hve oft ég hef farið í sund á
árinu" rather than showing a stale count.

`netlify.toml` rewrites `/sund.json` to the public Sund snapshot at
`sund.talva.is`. Netlify fetches it server-side, so the browser sees a
same-origin request and the Sund site needs no CORS headers.

Note that `sund.talva.is` is split-horizon: on the tailnet it resolves to the
internal, access-code-gated app where `/state.json` does not exist. From the
public internet it is the read-only snapshot. Test it with the public path:

```sh
curl --resolve sund.talva.is:443:104.21.28.39 https://sund.talva.is/state.json
```

## Deployment

Deployed by Netlify from this repo. DNS is hosted at Cloudflare, delegated from ISNIC.

| Record | Value | Notes |
| --- | --- | --- |
| `franklin.is` A | `75.2.60.5` | Netlify's apex load balancer. Must be **DNS only** (grey cloud). |
| `www` | Netlify | Currently proxied (orange cloud). |

Two things that have bitten this domain before:

- **The apex A record must be exactly `75.2.60.5`.** It sat on `72.2.60.5` for months —
  a real address belonging to an unrelated ISP, so the apex silently timed out while
  Netlify showed "pending DNS verification" indefinitely.
- **Cloudflare's proxy hides the origin from Netlify.** A proxied record resolves to
  Cloudflare's IPs, so Netlify cannot verify the domain or issue a certificate. Set the
  record to DNS only until the certificate is issued.

Verify what the world actually sees, bypassing every cache:

```sh
dig @robin.ns.cloudflare.com franklin.is A +norecurse +short
```
