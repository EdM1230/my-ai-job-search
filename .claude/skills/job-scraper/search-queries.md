# Search Queries for Job Scraper

<!-- SETUP: Customize these queries based on your skills, target roles, and location -->

## Installed portal CLIs (primary for `/scrape`)

`/scrape` discovers every portal skill under `.agents/skills/*/SKILL.md` and runs its CLI first. Shipped country-agnostic CLIs include `linkedin-search` and `freehire-search`; Danish demos and any skill you add with `/add-portal` are included the same way. You do **not** need a matching `site:` line below for those CLIs to run.

The `site:` query templates in this file are the **WebSearch fallback** — for portals without a CLI, company career pages, or when a CLI fails.

**Language scope:** write every query category in every language listed in your CLAUDE.md Languages table (typically 1-2, sometimes more). A posting requiring a language you have *not* declared, as a job condition, is excluded before scoring; a posting requiring a *higher level* than you declared in a language you *do* work in is flagged for your own judgment, not excluded — see `04-job-evaluation.md`'s Language Gate, the single source of truth for this rule. Translate each category's keywords rather than machine-translating word-for-word (e.g. "Frontend Developer" -> "Desarrollador Frontend", not a literal word-for-word translation) if you work in more than one language.

## Search Sites

Primary (Malaysian job boards - none of these has a CLI yet; scaffold one with `/add-portal`):

| Board | Status as of 2026-09-14 | How to read it |
|---|---|---|
| **linkedin.com/jobs** | **Working** - the main source so far | `linkedin-search` CLI, or the `jobs-guest` search endpoint on **`www.linkedin.com`** |
| **my.hiredly.com** (formerly WOBB) | **Working** - best source for Malaysian internships; the only dedicated SEO internships found came from here | Listings are JS-rendered, so page fetches return an empty shell. Use the **sitemap** instead (see below) |
| **my.jobstreet.com** | **Blocked by Cloudflare** | Serves a `Just a moment...` bot challenge to datacenter IPs. Do not attempt to defeat it. Works from a normal browser on a home connection |
| **malaysia.indeed.com** | **Untested - needs allowlisting** | See the Indeed note below |
| **internsheeps.com** / **ricebowl.my** | Untested | Malaysia-specific internship boards (optional) |

### Hiredly via sitemap (the method that works)

Hiredly's `robots.txt` advertises a sitemap and sets `Allow: /`. One request gets every live job URL, and the slugs encode company and title, so filtering happens offline with no further requests:

```bash
curl -sS https://my.hiredly.com/sitemap-jobs.xml -o hj.xml
grep -o "<loc>[^<]*</loc>" hj.xml | sed 's|</\?loc>||g' | sort -u > all.txt
grep -iE "job-.*(seo|search-engine)" all.txt
grep -iE "intern" all.txt | grep -iE "digital|marketing|seo|performance|web"
```

Keep the previous run's `all.txt` and `comm -13 old new` to get only what was added since - and a URL that has *dropped out* is a posting that has closed. This is how the Hypercharge Digital internship was caught closing between 09-11 and 09-14.

### Indeed Malaysia (added on request, 2026-09-14 - not yet verified)

Indeed's country domain is **`malaysia.indeed.com`** (`my.indeed.com` and `www.indeed.com` also resolve). As of 2026-09-14 **all three are denied by the cloud environment's network policy** - the proxy rejects CONNECT before the request leaves. To use Indeed from a cloud session, add to the environment's **Custom** allowed-domains list:

```text
malaysia.indeed.com
www.indeed.com
```

**Expect bot protection even then.** Indeed blocks datacenter IPs more aggressively than any other board on this list, so a cloud session may get a challenge page exactly like JobStreet's. If that happens, **do not try to defeat the challenge** - run the scrape from a local machine instead, where the residential IP and a real browser make it a non-issue. Check `https://malaysia.indeed.com/robots.txt` for a sitemap first: if Indeed publishes one, the Hiredly method above is cheaper and more reliable than fetching search pages.

> **Note:** the four shipped portal CLIs (Jobindex, Jobnet, JobBank, JobDanmark) are **Danish** and will return nothing useful for a KL search. Only `linkedin-search` and `freehire-search` are country-agnostic - and `freehire` is tech-first, so it returns little for marketing roles. Run `/add-portal` for JobStreet or Hiredly to get proper CLI coverage of this market.

> **Known environment issue:** in Anthropic-hosted cloud sessions, **Bun cannot complete TLS through the egress proxy** (CONNECT succeeds, the tunnelled handshake resets), which breaks the `linkedin-search` CLI. `curl` and WebFetch against the same endpoints work fine. Headless Chromium cannot reach any host at all. None of this applies when running locally.

Secondary (company career pages via Google):
- Direct Google searches with `site:` filters for known target companies

## Query Categories

Queries are grouped by priority. Write **each category in every language from your Languages table** (see Language scope above). Combine each query with your location terms (e.g. your city, region, or metro area) where the site supports it.

**Organize by function, not job title.** The same underlying work carries different titles across companies and markets (a "Data Scientist" role at one employer may be posted as "Insights Analyst" or "Data Consultant" at another). Name each priority category after the function it covers, and list several plausible job titles as query variants within that category rather than betting an entire priority tier on one exact title string.

All queries are written in English. Malay and Mandarin are on the Languages table, but Malaysian
postings for agency SEO/PPC roles are advertised in English almost without exception; a handful of
Mandarin variants are included in Priority 2 for Chinese-speaking agencies and clients in KL.

### Priority 1: SEO / PPC internships (primary target)

These match the strongest and most desired direction: a paid SEO or PPC internship at a digital
or performance marketing agency in the Klang Valley, for the next semester break.

```
site:jobstreet.com.my "SEO intern" Kuala Lumpur
site:jobstreet.com.my "digital marketing intern" SEO Kuala Lumpur
site:jobstreet.com.my "PPC intern" OR "paid search intern" Kuala Lumpur
site:hiredly.com "SEO" OR "PPC" intern Kuala Lumpur
site:linkedin.com/jobs "SEO intern" Malaysia
site:linkedin.com/jobs "performance marketing intern" Kuala Lumpur
site:linkedin.com/jobs "search engine marketing intern" Malaysia
site:malaysia.indeed.com "SEO intern" Kuala Lumpur
site:malaysia.indeed.com "digital marketing intern" SEO Kuala Lumpur
site:malaysia.indeed.com "PPC" OR "paid search" intern Kuala Lumpur
```

### Priority 2: Technical SEO, CRO and paid media (domain expertise)

These match the documented domain expertise: technical SEO, page speed, landing page CRO and
conversion optimisation for paid traffic.

```
site:jobstreet.com.my "technical SEO" Kuala Lumpur OR Selangor
site:jobstreet.com.my "conversion rate optimisation" OR "CRO" Kuala Lumpur
site:jobstreet.com.my "landing page" optimisation intern Malaysia
site:linkedin.com/jobs "technical SEO" Kuala Lumpur Malaysia
site:linkedin.com/jobs "Google Ads" intern Kuala Lumpur
site:jobstreet.com.my 数码营销 实习 吉隆坡
site:linkedin.com/jobs SEO 实习生 马来西亚
site:malaysia.indeed.com "technical SEO" Kuala Lumpur
site:malaysia.indeed.com "conversion rate optimisation" OR "CRO" Kuala Lumpur
site:malaysia.indeed.com "Google Ads" intern Kuala Lumpur
```

### Priority 3: Adjacent marketing and analytics roles

Adjacent roles the profile could pivot into - analytics-leaning marketing, growth, and
web-development-with-marketing internships.

```
site:jobstreet.com.my "marketing analytics intern" Kuala Lumpur
site:jobstreet.com.my "growth marketing" intern Kuala Lumpur
site:jobstreet.com.my "web developer intern" WordPress Kuala Lumpur
site:hiredly.com "digital marketing" intern Kuala Lumpur
site:linkedin.com/jobs "marketing intern" analytics Kuala Lumpur
site:malaysia.indeed.com "marketing analytics intern" Kuala Lumpur
site:malaysia.indeed.com "web developer intern" WordPress Kuala Lumpur
```

### Priority 4: Broader digital marketing (wider net)

Wider net - general digital marketing internships where the SEO/PPC component may be unstated
in the title but present in the role.

```
site:jobstreet.com.my "digital marketing" internship Kuala Lumpur
site:jobstreet.com.my "e-commerce" marketing intern Klang Valley
site:linkedin.com/jobs "digital marketing" intern Malaysia
site:ricebowl.my marketing internship Kuala Lumpur
site:malaysia.indeed.com "digital marketing" internship Kuala Lumpur
site:malaysia.indeed.com "e-commerce" marketing intern Klang Valley
```

## Location Filter

When evaluating results, verify the job location is within reasonable commute distance from your home. Define acceptable areas:
Constraint: Klang Valley, onsite or hybrid. Commute beyond roughly an hour each way is a
deal-breaker, and KL traffic makes nominal distance a poor proxy - judge by transit time.

- **Ideal:** Kuala Lumpur city centre, Bangsar, Mid Valley, KL Sentral, Damansara Heights, Mont Kiara, Bukit Jalil - all on rail and well under an hour
- **Acceptable:** Petaling Jaya, Subang Jaya, Cyberjaya, Puchong, Shah Alam, Cheras, Ampang - reachable within about an hour, especially near an LRT/MRT/KTM station
- **Borderline:** Klang, Rawang, Kajang, Semenyih, Sepang/KLIA area - roughly an hour or more each way; flag for the user's judgment rather than auto-including
- **Too far:** anywhere outside the Klang Valley (Penang, Johor Bahru, Ipoh, East Malaysia), and Singapore - these require relocation, which is a stated deal-breaker
- **Remote / hybrid:** acceptable, and a hybrid role widens the "borderline" tier considerably - note the office-day requirement explicitly

## Language Filter

Your working languages and levels are in CLAUDE.md's Languages table. When filtering scraped results, apply `04-job-evaluation.md`'s Language Gate: a posting requiring a language you haven't declared at all is excluded; a posting requiring a higher level than you declared in a language you do work in is not excluded, flag it clearly instead (see `job-scraper/SKILL.md`'s Step 3 "Quick Fit Assessment" for how the flag surfaces in `/scrape` output). Postings simply *written* in a language you don't work in, that don't require it on the job, are fine.

## Date Filter

Only include jobs posted within the last 14 days, or with an application deadline that has not yet passed. If a posting date cannot be determined, include it but flag as "date unknown".

**Internship timing:** the target is a 3-4 month placement over the next university semester break.
Malaysian agencies commonly advertise internships 1-3 months ahead of the intake, so widen to 30
days when a scrape returns thin results, and always check the stated intake month against your
break dates before applying.

## Adapting Queries

If the user specifies a focus area, select queries from the matching category and also generate 2-3 custom queries for that focus. For example:
- "/scrape [focus_area]" -> relevant category queries + custom focus-specific queries
