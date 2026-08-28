# How a website address turns into a website — a DNS walkthrough

*Written for PF-04, in my own words, aimed at a non-technical teammate.*

Every website lives on a server somewhere, and every server has a numeric address called an **IP address** — something like `104.198.14.52`. Computers are happy typing that. People aren't. **DNS (the Domain Name System)** is the layer that lets us type `zoya.netlify.app` into a browser instead of a string of numbers, and have it quietly translate that name into the address a computer needs.

Think of it like a phone book for the internet, except it's not one book — it's a chain of people you ask, each one pointing you a little closer to the right number, until someone finally hands you the answer.

## The cast of characters

- **Resolver** — usually run by your internet provider (or a public one like Google's `8.8.8.8`). This is the "front desk" your device asks first: *"where is zoya.netlify.app?"*
- **Nameserver** — a server that actually holds the records for a domain. There's a hierarchy of these (root → `.app` → the domain's own nameserver), and the resolver climbs it step by step if it doesn't already know the answer.
- **Record** — an entry stored on a nameserver that maps a name to something useful. The two that matter here:
  - **A record** — maps a name directly to an IP address (e.g. `zoya.com → 104.198.14.52`).
  - **CNAME record** — maps a name to *another name*, not an IP address directly. For example, `www.zoya.com` could be a CNAME pointing to `zoya.netlify.app`, which itself resolves further down the chain. CNAMEs are how you connect your own custom domain to a host like Netlify: you don't get Netlify's IP address (it can change), you just say "whatever this name points to, follow it there."
- **Response** — the final answer that gets handed back down the chain to your browser: an IP address it can actually connect to.

## What happens when you type a web address

1. **You type it.** Say a teammate types `zoya.netlify.app` into their browser.
2. **The browser checks nearby first.** It looks in its own cache, then the device's cache, in case this name was looked up recently. If it finds a fresh answer, it skips straight to step 6.
3. **The resolver takes over.** If nothing is cached, the request goes to the resolver (typically run by the ISP or a public DNS service). The resolver checks its own cache too — if many people recently visited the same site, it may already know the answer.
4. **The resolver climbs the hierarchy, if it needs to.** If it's a fresh lookup, the resolver asks a **root nameserver** which nameserver handles `.app` domains, then asks the `.app` nameserver which nameserver handles `netlify.app`, and finally asks *that* nameserver for the specific record. This is a few quick round-trips, not a slow process — it usually takes milliseconds.
5. **The record is returned.** The nameserver responsible for the domain hands back the actual record — an A record (a direct IP address) or, if the name is set up as a CNAME, the resolver follows that pointer to another name and repeats the lookup until it lands on an A record with a real IP address.
6. **The response reaches the browser.** The resolver sends the final IP address back to the browser (and caches it briefly, so the next visitor's lookup is faster).
7. **The browser connects.** With the IP address in hand, the browser opens a secure connection (HTTPS) directly to the host's server and asks for the page. The server sends back the site, and it loads on screen.

All of that — steps 2 through 6 — typically happens in well under a second, before a person even notices a pause.

## Why this matters for hosting

When I renamed my site on Netlify to a clean URL like `zoya.netlify.app`, Netlify was managing the DNS record for that name for me — I didn't have to touch a nameserver directly. If I later buy my own domain (say, `zoyahulio.com`) and want it to point at the same Netlify site, I'd add a CNAME record at my domain registrar that points `zoyahulio.com` (or `www.zoyahulio.com`) at my Netlify site's address. Netlify also handles the HTTPS certificate automatically once that record is verified, which is why the padlock just appears without any extra setup on my end.

Understanding this chain means that if a domain doesn't work, I know where to look: is it the record itself (set up wrong), the nameserver (not yet propagated — DNS changes can take minutes to hours to spread globally), or something else entirely like the host's server being down.
