# Cloudflare notes - Zentos

## apex -> www is NOT handled in the repo
The site canonicalises to `https://www.zentos.com.au` (all 14 pages carry `rel=canonical` to www),
but Cloudflare Pages serves the apex too, on its own host, with a 200.

**A `_redirects` file cannot fix this.** Pages matches `_redirects` on the **path only** - it has no
host-based matching, so a rule like

    https://zentos.com.au/* https://www.zentos.com.au/:splat 301

is parsed and silently ignored. Verified 2026-09-07 on this site and on NPD Building Solutions, which
has carried exactly that rule for months and still answers 200 on both hosts.

**The real fix is a Cloudflare Redirect Rule** (dashboard: the zone -> Rules -> Redirect Rules), matching
`http.host eq "zentos.com.au"` and redirecting to `https://www.zentos.com.au` + the path, 301, preserving
the query string. Needs a Cloudflare API token or the dashboard; neither is on the build machine.

Until then `rel=canonical` carries it, which Google does honour - this is a tidiness and
signal-consolidation issue, not an indexing blocker.

## Cloudflare injects its own robots.txt
The served `/robots.txt` is Cloudflare's managed block (AI crawler `Disallow` rules plus
`Content-Signal: search=yes,ai-train=no,use=reference`) with our file appended after it. Googlebot is
allowed and our `Sitemap:` line is present, so this is fine - but do not be surprised that the served
file is longer than the one in the repo.
