# Deploy guide — vesica.innercartography.one

Two files ship: `index.html` (the deck) and `llms.txt` (the agent payload). Plan: Vercel, early AM, via Claude Code.

## Files
- `vesica-deck.html`  → rename to `index.html` at project root
- `llms.txt`          → project root (served as-is at /llms.txt)

## Before you deploy — find/replace placeholders
In BOTH files, replace:
- `vesica.innercartography.one`  → your real deployed domain
- `https://innercartography.one` (Room to Cooperate link) → the published constellation URL
Also: drop the real QR image into slide 9 (replace the `.qr-slot` div with `<img>`), pointing at the constellation URL. Generate the QR last, once the URL is final.

## Option A — simplest (static, recommended for the talk)
```
vercel deploy --prod
```
That's it. The deck is at `/`, the agent payload at `/llms.txt`.
On the title slide you say: "curl vesica.innercartography.one/llms.txt".
Bulletproof, no server logic, works on venue wifi.

## Option B — bonus: true `curl <the deck URL>` returns markdown
So that `curl vesica.innercartography.one` (no /llms.txt) returns the agent payload,
while a browser at the same URL gets the deck. Vercel Edge Middleware:

Create `middleware.ts` at project root:
```ts
import { next, rewrite } from '@vercel/edge';
export const config = { matcher: '/' };

export default function middleware(req: Request) {
  const ua = (req.headers.get('user-agent') || '').toLowerCase();
  const accept = (req.headers.get('accept') || '').toLowerCase();
  // curl / wget / common agent fetchers, or explicitly asking for plain/markdown
  const wantsRaw =
    ua.includes('curl') || ua.includes('wget') || ua.includes('httpie') ||
    ua.includes('python-requests') || ua.includes('node-fetch') ||
    (accept.includes('text/plain') && !accept.includes('text/html')) ||
    accept.includes('text/markdown');
  if (wantsRaw) return rewrite(new URL('/llms.txt', req.url));
  return next();
}
```
Add to `package.json` (or let Claude Code init it):
```json
{ "dependencies": { "@vercel/edge": "^1.1.1" } }
```
Now: browser → deck; `curl` → markdown. Demo it live: `curl` your own URL on stage.

> Caveat: some link-preview bots also send non-HTML accepts; the UA checks keep it tight.
> If anything misbehaves, delete middleware.ts and fall back to Option A + /llms.txt.

## Sanity checks before bed
- [ ] `index.html` loads, scrolls, and `P` toggles present mode
- [ ] Counter reads 10 slides; arrow keys / clicker advance in present mode (your real fallback)
- [ ] Slide 2 (vesica) names co-regulation; slide 8 is the ROOM / "point is the pattern" slide
- [ ] /llms.txt resolves and reads clean
- [ ] domain + Room-to-Cooperate links replaced in both files
- [ ] QR on slide 10 points to the live constellation URL
- [ ] (Option B) `curl <url>` returns markdown, browser returns deck
