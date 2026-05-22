# tweeter — curated Twitter sources for a macro/geo/Brazil intelligence agent

> ~185 Twitter (X) accounts feeding a multilingual financial-intelligence pipeline.
> Annotated with **who** each user is, **why** they're tracked, and **how** the
> system filters their tweets before storage.

This is the live follow list behind the M3xA intelligence system. Sibling repos:

- [`m3xa-core`](https://github.com/prcodex/m3xa-core) — the House pattern (architecture, didactic)
- [`chunking_vector`](https://github.com/prcodex/chunking_vector) — the v5 RAG spec (Bedrock-native, anonymized)
- [`M3XABR_NEW`](https://github.com/prcodex/M3XABR_NEW) — the expertise-composition slice

Where the architecture docs use anonymized aliases (`Bank1`, `Expert1`, `Podcast1`), **this repo uses real handles**. Twitter accounts are public follows, not paid subscriptions — the curation is the value, and aliasing it would defeat the point.

## Table of contents

- [How to read this list](#how-to-read-this-list)
- [A. Brazil domain — PT-classified accounts](#a-brazil-domain--pt-classified-accounts)
- [B. Macro / Geo domain — EN-classified accounts](#b-macro--geo-domain--en-classified-accounts)
  - [B.1 Macro — rates, FX, fiscal, central banks, markets](#b1-macro--rates-fx-fiscal-central-banks-markets)
  - [B.2 Geo — great-power, conflicts, regional](#b2-geo--great-power-conflicts-regional)
- [Filtering process — trash filter + AI ranking](#filtering-process--trash-filter--ai-ranking)
- [twitterapi.io vs Twitter/X API — why we use twitterapi.io](#twitterapiio-vs-twitterx-api--why-we-use-twitterapiio)
- [Stats](#stats)
- [How to reproduce](#how-to-reproduce)

---

## How to read this list

Each account row is annotated:

| Field | Meaning |
|---|---|
| Handle | `@username` on X |
| Role | What they cover |
| Why tracked | Signal density, unique access, contrarian framing, breaking-news lead |
| Weight | Configured retrieval-boost weight (0.5–1.0) — higher = stronger boost when their content matches a query |
| VIP / junk-bypass | Whether the AI trash filter is skipped for this account (true = always store, no junk filtering) |
| Notes | Cross-references to related accounts, quirks, fallbacks |

**The full list (~185 accounts) is dynamic via the Sources Manager service** (port 8551 in the live system). What follows is the high-signal subset with rich annotations. Lower-signal accounts in the long tail are tracked but not individually documented here.

---

## A. Brazil domain — PT-classified accounts

> Storage filter: `domain = 'brazil'` in LanceDB. ~55 accounts total in the live system.
> Bot: `@M3xabr_bot`. Routing: `/api/rag` with `filters=['brazil']`.

These are **journalists and political columnists** writing in Portuguese — judicial, electoral, fiscal, BCB-watcher coverage. Brazilian banking handles (Itaú, XP) live in research-note emails rather than Twitter; only a handful of Brazilian bank-research X accounts make this list.

### Political columnists & analysts

#### `@danielalimaaa` — Daniela Lima (UOL / GloboNews)
- **Role:** Brasília political columnist; STF and Esplanada coverage
- **Why tracked:** Strong direct-source access; consistent on cabinet-shuffle / coalition signals
- **Weight:** 1.0
- **Notes:** Cross-referenced with her UOL columns scraper (`uol_columnists`, cron `5 11,23 * * *`)

#### `@JosiasDeSouz` — Josias de Souza (UOL / Folha alum)
- **Role:** Senior political columnist; analytical voice, not breaking news
- **Why tracked:** Long-form framing of political-economy events; useful for the "what does this mean" layer
- **Weight:** 1.0
- **Notes:** UOL scraper also pulls his column body

#### `@reinaldoazeved2` — Reinaldo Azevedo
- **Role:** Brazilian political commentator (UOL)
- **Why tracked:** Different lens from Daniela / Josias; useful for both-sides synthesis on contentious topics
- **Weight:** 0.8

#### `@veramagalhaes` — Vera Magalhães
- **Role:** Political columnist (CNN Brasil); Brasília + STF
- **Why tracked:** Direct quotes from political actors; high signal on STF rulings
- **Weight:** 1.0

#### `@bernardomello` — Bernardo Mello Franco (O Globo)
- **Role:** Political columnist; centrist framing
- **Why tracked:** Cross-spectrum coverage diversity
- **Weight:** 0.8

#### `@andrezamatais` — Andreza Matais
- **Role:** Brasília political reporter (Estadão); breaking
- **Why tracked:** Early on cabinet/coalition shifts
- **Weight:** 1.0

#### `@jamilchade` — Jamil Chade
- **Role:** International correspondent (UOL), Brasil-Geneva axis
- **Why tracked:** Brazil-foreign-policy intersection; rare in the BR pundit set
- **Weight:** 0.8

#### `@FelipeRecondo` — Felipe Recondo (JOTA)
- **Role:** STF / judicial coverage; co-author of "Os Onze"
- **Why tracked:** Pre-eminent STF watcher; tracked alongside his "Recondo e os Onze" Substack newsletter (`recondo_substack`, cron `30 */4 * * *`)
- **Weight:** 1.0
- **Notes:** Substack body is the primary signal; Twitter adds real-time flags

### Brazilian bank research (limited Twitter presence)

Most Brazilian bank research arrives via email research notes (Itaú Brazil, XP Análise Política, XP Sentinel) rather than Twitter. The Twitter follow list for Brazilian banks is intentionally thin — corporate accounts produce mostly marketing.

### Notes on the Brazil follow list

- **Full set:** ~55 PT-classified accounts in the live `source_classifications.db`, queryable via `GET 8551/api/twitter-accounts?classification=brazil`.
- **What's not here:** Most general-news Brazilian Twitter (Folha, Valor, Antagonista corporate accounts) — those are covered by their RSS / web scrapers, which have richer extraction (full article body) than the 280-char tweet would provide.
- **Crossover:** Some accounts (e.g., political columnists) appear in both Twitter and email/web feeds. The pipeline dedupes by URL hash at the retriever, so cross-source duplication is bounded.

---

## B. Macro / Geo domain — EN-classified accounts

> Storage filter: `domain = 'macro'` in LanceDB. ~130 accounts total.
> Bot: `@M3xA_bot`. Routing: `/api/rag` with `filters=['macro']`.

The macro/geo list splits roughly into:
- **B.1 Macro** (~95 accounts): rates, FX, central banks, fiscal, equity markets
- **B.2 Geo** (~35 accounts): great-power competition, conflicts, regional (Iran, China, Russia, Middle East)

The split is editorial, not enforced by the DB — both share the `macro` domain. The B.1/B.2 distinction below is to make the curation legible.

---

### B.1 Macro — rates, FX, fiscal, central banks, markets

#### Real-time wires

##### `@DeItaone` — Walter Bloomberg (real-time BBG headlines)
- **Role:** Aggregates Bloomberg terminal headlines in real time
- **Why tracked:** Fastest non-terminal access to BBG breaking; ~100+ headlines/day
- **Weight:** 1.0  · **VIP (junk_bypass=true)** — every tweet stored unscored
- **Notes:** Has its own dedicated pipeline (`deltaone_timeline.py` on R6G, cron `4,24,44 * * * *`). Generates T+0 / T+5 market-reaction snapshots for 17 tickers per headline. Single source of truth in `entity_deltaone.md`.

##### `@FirstSquawk` — First Squawk (real-time financial newswire)
- **Role:** Cross-market real-time wire — central bank speakers, geopolitical breaking, economic data prints
- **Why tracked:** Often beats DeItaone on geopolitical breaking
- **Weight:** 1.0  · **VIP**

##### `@financialjuice` — FinancialJuice
- **Role:** Real-time market headlines aggregator
- **Why tracked:** Complements DeItaone + FirstSquawk; covers wires the others miss
- **Weight:** 0.8

##### `@zerohedge` — ZeroHedge
- **Role:** Markets-focused contrarian aggregator
- **Why tracked:** Headlines often surface stories before mainstream finance press; treat with skepticism on framing
- **Weight:** 0.5

#### Independent macro analysts (Substack-backed)

##### `@SetserBrad` — Brad Setser (CFR)
- **Role:** EM sovereign credit, capital flows, China economic data
- **Why tracked:** Premier voice on sovereign capital accounts; cross-references his blog (Council on Foreign Relations Follow the Money)
- **Weight:** 1.0

##### `@michaelxpettis` — Michael Pettis (Peking University)
- **Role:** China macro, global imbalances, fiscal-monetary interaction
- **Why tracked:** Long-arc framing for the China story; pairs well with Setser
- **Weight:** 1.0

##### `@josephpolitano` — Joey Politano (Apricitas)
- **Role:** US economic data interpretation
- **Why tracked:** Best-in-class print-day analysis (CPI, PCE, NFP); his Aprecitas blog is also scraped (`aprecitas` cron `25 */4 * * *`)
- **Weight:** 0.8

##### `@claudiasahm` — Claudia Sahm (former Fed)
- **Role:** US labor market, recession indicators, Fed reaction function
- **Why tracked:** Named for the Sahm Rule; signal-dense on recession probability
- **Weight:** 0.8

##### `@elerianm` — Mohamed El-Erian (Allianz / Queens')
- **Role:** Macro framing, Fed-watch, global commentary
- **Why tracked:** High-visibility voice; useful for tracking consensus shifts
- **Weight:** 0.8

##### `@truthgundlach` — Jeffrey Gundlach (DoubleLine)
- **Role:** Rates, credit, equity strategy
- **Why tracked:** Tier-1 fund-manager voice; quarterly conference calls supplement Twitter
- **Weight:** 1.0

##### `@econguyrosie` — David Rosenberg (Rosenberg Research)
- **Role:** Bearish macro, recession-skewed framing
- **Why tracked:** Counterweight to consensus optimism; useful for both-sides positioning
- **Weight:** 0.8

##### `@macroalf` — Alfonso Peccatiello (The Macro Compass)
- **Role:** Macro framework + market positioning
- **Why tracked:** Strong on rate-cycle framing and cross-asset positioning
- **Weight:** 1.0

##### `@stephen_roach` — Stephen Roach (Yale, ex-Morgan Stanley)
- **Role:** China-US macro, long-arc framing
- **Why tracked:** Historical-context lens (e.g., revisiting Japan analogies); his Substack also scraped
- **Weight:** 0.8

##### `@nfergus` — Niall Ferguson (Stanford, Greenmantle)
- **Role:** Historical-economic framing of current events
- **Why tracked:** Greenmantle research note arrives via Substack scraper; Twitter adds quick takes
- **Weight:** 0.8

##### `@jnordvig` — Jens Nordvig (Exante Data)
- **Role:** FX, EM, capital flows
- **Why tracked:** Tier-1 FX strategist; signal-dense on currency moves
- **Weight:** 1.0

##### `@robin_j_brooks` — Robin Brooks (Brookings, ex-IIF)
- **Role:** EM capital flows, sanctions impact, sovereign analysis
- **Why tracked:** Distinctive data-driven framing on EM dislocations
- **Weight:** 0.8

##### `@charliebilello` — Charlie Bilello (Creative Planning)
- **Role:** Markets data visualization, equity factors, rates
- **Why tracked:** High-quality charts and historical-context tweets
- **Weight:** 1.0

##### `@gave_vincent` — Vincent Gave (Gavekal)
- **Role:** Asia macro, global imbalances, long-arc framing
- **Why tracked:** Gavekal research notes via Drive PDFs; Twitter adds real-time
- **Weight:** 0.8

##### `@pantheonmacro` — Pantheon Macroeconomics
- **Role:** Independent macro research shop (US-focused)
- **Why tracked:** Tight, data-driven calls
- **Weight:** 1.0

##### `@johnauthers` — John Authers (Bloomberg Opinion)
- **Role:** Markets columnist, macro framing
- **Why tracked:** "Points of Return" newsletter equivalent in tweet form
- **Weight:** 0.8

##### `@fedguy12` — Joseph Wang ("Fed Guy")
- **Role:** Fed plumbing, repo markets, reserve management
- **Why tracked:** Best plumbing-side Fed analyst on X
- **Weight:** 1.0

#### Sell-side bank corporate / research

##### `@gs_research`, `@gs_markets` — Goldman Sachs research / markets desks
- **Role:** GS research summary tweets + market commentary
- **Why tracked:** Public-side of GS coverage; full research notes via email
- **Weight:** (per `entity_goldman_sachs.md`)

##### `@tonyp` — Tony Pasquariello (Goldman, hedge fund coverage)
- **Role:** Hedge fund flows, positioning, market structure
- **Why tracked:** Tier-1 voice on positioning; signal-dense
- **Weight:** 1.0  · **VIP**

##### `@jpmorgan` — JP Morgan corporate
- **Role:** Corporate news, research summaries
- **Why tracked:** Cover for macro-routing queries; marketing noise filtered
- **Weight:** 0.5

##### `@bcaresearch` — BCA Research
- **Role:** Independent macro research shop
- **Why tracked:** Their dashboard is also scraped (`bca_dashboard` cron `10 */6 * * *`)
- **Weight:** 1.0

##### `@anz_research` — ANZ Bank Research
- **Role:** Asia-Pacific macro, commodities, EM Asia
- **Why tracked:** ANZ "5 in 5" newsletter also arrives via Gmail
- **Weight:** 0.8

#### Wire services + outlets

##### `@bloomberg`, `@business`
- **Role:** Bloomberg corporate
- **Why tracked:** Catches BBG features that DeItaone's headline-only feed misses
- **Weight:** 0.8

##### `@reuters`
- **Role:** Reuters corporate
- **Why tracked:** Reuters articles also scraped via gold daemon (`run_rss_collector.sh` worker w3)
- **Weight:** 0.8

##### `@wsj`, `@wsjmarkets`
- **Role:** WSJ corporate + markets desk
- **Weight:** 0.8

##### `@ft`, `@FTMarkets`, `@FTAlphaville`, `@FTCompanies`, `@FinancialTimes`
- **Role:** FT cluster — corporate, markets desk, Alphaville (blog), companies, broader
- **Why tracked:** Many FT pieces are paywall-gated; Twitter excerpts often carry the gist
- **Weight:** 0.8
- **Notes:** Dedicated `ft_twitter_collector.py` pipeline (cron `*/30 * * * *`)

##### `@nicktimiraos` — Nick Timiraos (WSJ Fed reporter)
- **Role:** Fed reporter; closest to Fed-trial-balloon signaling
- **Why tracked:** When Timiraos tweets a "Fed expected to..." piece, it's often quasi-official
- **Weight:** 1.0  · **VIP**

##### `@yarotrof` — Yaroslav Trofimov (WSJ)
- **Role:** Geopolitical correspondent; war, sanctions, alliance politics
- **Why tracked:** Direct-source access in conflict zones
- **Weight:** 1.0  · **VIP**

##### `@javierblas` — Javier Blas (Bloomberg)
- **Role:** Commodities (oil, gas, metals), energy geopolitics
- **Why tracked:** Top voice on physical-commodity markets
- **Weight:** 1.0  · **VIP**

##### `@raghavanreports` — Anita Raghavan
- **Role:** Wall Street / finance investigative reporter
- **Weight:** 1.0  · **VIP**

#### Officials and policy voices

##### `@secscottbessent` — Scott Bessent (Treasury Secretary)
- **Role:** US Treasury communications
- **Why tracked:** Official Treasury statements; market-moving
- **Weight:** 1.0  · **VIP**

##### `@howardlutnick` — Howard Lutnick (Commerce Secretary)
- **Role:** US Commerce communications
- **Weight:** 1.0  · **VIP**

##### `@ericsrosengren` — Eric Rosengren (former Fed)
- **Role:** Former Boston Fed; rate-cycle commentary
- **Weight:** 1.0

---

### B.2 Geo — great-power, conflicts, regional

> The geopolitical follow list. Heavier on independent analysts than on official accounts — official accounts are tracked via wire scrapers (Xinhua, Anadolu, NYT Iran).

#### Iran / Gulf / Middle East

##### `@s_m_marandi` — Seyed Mohammad Marandi (University of Tehran)
- **Role:** Iranian perspective; closely tied to Iranian official framing
- **Why tracked:** **Both-sides rule** — geopolitical synthesis is incomplete without the Iranian framing. The pipeline auto-translates Arabic content where needed (Marandi tweets in English).
- **Weight:** 1.0

##### `@anasalhajji` — Anas Alhajji (energy expert)
- **Role:** Oil markets, OPEC+, energy geopolitics
- **Why tracked:** Best-in-class on Middle East energy policy; auto-translation for Arabic content
- **Weight:** 1.0
- **Notes:** Entity registry has dedicated boost rule (`entity_anas_alhajji.md`); 140+ tweets indexed

##### `@vali_nasr` — Vali Nasr (Johns Hopkins / Iran)
- **Role:** Iran politics, US-Iran relations
- **Why tracked:** Tier-1 academic on Iran-region politics
- **Weight:** 1.0  · **VIP**

##### `@alivaez` — Ali Vaez (Crisis Group)
- **Role:** Iran nuclear file, diplomacy track
- **Why tracked:** Direct lines to Tehran + Vienna negotiating circles
- **Weight:** 1.0

##### `@AnnaLeaJacobs` — Anna-Lea Jacobs (ICG / Crisis Group)
- **Role:** Iran-region analyst, security-track
- **Why tracked:** Complements Vaez on the security/political side
- **Weight:** 1.0

##### `@professorpape` — Robert Pape (University of Chicago)
- **Role:** Strategic studies, air campaigns, deterrence
- **Why tracked:** Academic depth on conflict dynamics
- **Weight:** 1.0
- **Notes:** Co-tracked via Substack scraper (entity rule: deduplicate Twitter ↔ Substack)

#### Great-power competition

##### `@ianbremmer` — Ian Bremmer (Eurasia Group)
- **Role:** Global political risk
- **Why tracked:** Tier-1 voice on great-power competition; useful for "top of mind" framing
- **Weight:** 1.0  · **VIP**
- **Notes:** Dedicated `bremmer_fetcher.py` pipeline (cron `5,35 * * * *`) with Groq Whisper for his video content

##### `@fukuyama` — Francis Fukuyama
- **Role:** Long-arc political-economy framing
- **Why tracked:** Slow but high-signal; pairs well with Ferguson + Pettis
- **Weight:** 0.8

##### `@thewarzonewire` — The War Zone (TWZ)
- **Role:** Defense / military reporting
- **Why tracked:** Hard-news on military movements; scraper also pulls full TWZ articles (`twz_gateway` cron `40 */2 * * *`)
- **Weight:** 1.0

#### Conflict / OSINT

The OSINT follow list (low-volume per-account, structured event reporting) is intentionally thin on Twitter — the pipeline's structured Iran scrapers (`iran_proxies_collector`, `iran_intel_scraper`, `hormuz_monitor`) cover the OSINT space better than ad-hoc Twitter follows.

---

## Filtering process — trash filter + AI ranking

```mermaid
flowchart LR
    SRC[Sources Manager · port 8551<br/>~185 accounts] -. /api/twitter-accounts .-> XSCR

    TWIO[twitterapi.io<br/>advanced_search +<br/>user/last_tweets] --> XSCR[xscraper_gateway.py<br/>cron 2,22,42 * * * *]

    XSCR --> JUNK{Junk filter<br/>rule-based}
    JUNK -- pass --> AISCORE[AI scoring<br/>Haiku 4.5<br/>1-10 relevance]
    JUNK -- drop --> X1[(dropped)]

    AISCORE --> VIP{VIP account?}
    VIP -- yes --> STORE[Store unscored<br/>has_vector=1.0]
    VIP -- no --> THRESH{Score ≥ threshold?}
    THRESH -- yes --> STORE
    THRESH -- no --> X2[(dropped)]

    STORE -. HTTP POST .-> R6G[R6G :8546<br/>/api/gateway-insert]
    R6G --> VOY[Voyage-3-large<br/>2048d embed]
    VOY --> LDB[(LanceDB unified_feed)]

    classDef gw fill:#1e3a8a,stroke:#3b82f6,color:#fff
    classDef r6 fill:#14532d,stroke:#22c55e,color:#fff
    classDef ext fill:#374151,stroke:#9ca3af,color:#fff
    classDef store fill:#581c87,stroke:#a855f7,color:#fff
    classDef drop fill:#7f1d1d,stroke:#ef4444,color:#fff
    classDef decision fill:#7c2d12,stroke:#ea580c,color:#fff

    class TWIO ext
    class SRC,XSCR,AISCORE,STORE gw
    class R6G,VOY r6
    class LDB store
    class X1,X2 drop
    class JUNK,VIP,THRESH decision
```

### Stage 1 — Collection (every 20 min)

A cron-driven scraper (`xscraper_gateway.py`, schedule `2,22,42 * * * *`) reads the live account list from the Sources Manager (`GET 8551/api/twitter-accounts`) and pulls each account's recent tweets via [twitterapi.io](https://twitterapi.io)'s `advanced_search` and `user/last_tweets` endpoints.

The choice of twitterapi.io over Twitter's official API is covered in [the next section](#twitterapiio-vs-twitterx-api--why-we-use-twitterapiio).

### Stage 2 — Trash filter (rule-based)

A rule-based pass drops tweets that don't carry signal regardless of who tweeted them:

- Pure retweets with no commentary (when the original is already in the index)
- Replies that are just `@-mentions` with no content
- Promotional patterns (giveaways, "buy my course", crypto-spam patterns)
- Length floor (<20 chars for non-VIP accounts)
- Recognized boilerplate ("Follow me for more...", "Thread 🧵 1/N" headers without bodies)

These rules are conservative — they err on the side of letting marginal content through to the AI ranker rather than dropping signal.

### Stage 3 — AI ranking (Haiku)

What survives Stage 2 goes through a Haiku-4.5 scoring pass:

```
Given a tweet from {handle}, who covers {classification}, rate the tweet's
relevance to a {macro/geo/brazil} intelligence brief on a 1-10 scale.
Consider: information density, novelty, attribution clarity, market relevance.
```

Tweets scoring **≥6** are kept; **<6** is dropped. The threshold is conservative — better to keep marginal content (it gets re-ranked at retrieval time anyway via institution boost + entity boost) than to discard a piece the user later asks about.

Cost: ~$18/month for Haiku scoring across ~5K candidate tweets/day.

### Stage 4 — VIP bypass

A small set of accounts (~20) is marked `junk_bypass=true` in the entity registry. For these, **every tweet is stored, unscored**:

- Real-time wires where speed matters more than per-tweet relevance: `@DeItaone`, `@FirstSquawk`
- Tier-1 voices where missing one tweet costs more than indexing a marginal one: `@ianbremmer`, `@vali_nasr`, `@yarotrof`, `@javierblas`, `@nicktimiraos`, `@tonyp`, `@secscottbessent`, `@howardlutnick`, `@raghavanreports`

VIPs use a `vip_scored` status flag in the DB so downstream code knows the tweet was stored without scoring. The retrieval boost layer still applies the account's weight.

### Stage 5 — Storage with retrieval weights

Surviving tweets are POSTed to the R6G ingestion endpoint (`/api/gateway-insert`), embedded with Voyage-3-large (2048d), and stored in LanceDB's `unified_feed` table with:

- `source` = handle
- `domain` = `macro` or `brazil`
- `has_vector = 1.0` (critical — see [m3xa-core lessons](https://github.com/prcodex/m3xa-core/blob/main/LESSONS.md))
- `created_at`, `text`, plus the AI score (if scored)

At query time, the retriever applies:

1. **Vector similarity** (Voyage embedding cosine)
2. **Entity boost** — when the query mentions a handle or institution that maps to this account, force-inject top tweets from this source into the synthesis context
3. **Source-class boost** — accounts with high `priority` get a small additive boost
4. **Time decay** — newer tweets weighted higher unless the query asks historical

See [`m3xa-core/concepts/institution_boost.md`](https://github.com/prcodex/m3xa-core/blob/main/concepts/institution_boost.md) for the boost layer's full design.

---

## twitterapi.io vs Twitter/X API — why we use twitterapi.io

> Short version: official X API pricing makes a 185-account pipeline economically irrational. twitterapi.io is a third-party that scrapes the public X surface and exposes a stable API at a fraction of the cost.

### Side-by-side

| Aspect | Official X API (v2) | twitterapi.io |
|---|---|---|
| **Pricing** (as of 2026) | $200/mo Basic, $5000/mo Pro, free tier basically unusable for ingestion | **$30/mo flat** |
| **Tweet pulls/month** | Basic: ~10K reads/mo. Pro: 1M reads/mo. | Effectively unlimited at $30/mo |
| **Authentication** | OAuth 2.0 + bearer tokens, app + user contexts | Single API key in header |
| **Endpoint shape** | RESTful, `GET /2/users/:id/tweets`, complex paging | RESTful, simpler: `advanced_search`, `user/last_tweets` |
| **Time-window search** | Native `start_time` / `end_time` params | `since:` / `until:` operators in `advanced_search` query string (works as of mid-2026; see operational concern below) |
| **Rate limits** | Strict per-window; bursts get hard-throttled | Generous; in practice not the bottleneck |
| **Reliability** | Enterprise SLA at Pro tier; basic has no SLA | No SLA; depends on continued X scraping access |
| **Stability of contract** | Anchored to X's official roadmap; breaking changes when X deprecates v1.1 endpoints, etc. | Depends on twitterapi.io maintaining its scraping infrastructure; lower stability ceiling, but lower price floor |
| **Best for** | Building products that need official partnership / SLA | Internal pipelines where cost matters and downtime is recoverable |

### What we use it for, concretely

| Endpoint | Used by | Purpose |
|---|---|---|
| `advanced_search` | `xscraper_gateway.py`, `bremmer_fetcher.py` | Time-windowed query: "tweets from @{handle} between {since} and {until}" |
| `user/last_tweets` | `deltaone_timeline.py` | Pull the most recent N tweets from a single account (used for the DeItaone dedicated pipeline) |

### Operational concerns (real ones)

These came up in production and are flagged in `okr_queue.md`:

1. **`since:` / `until:` operators may be deprecated**. twitterapi.io has signaled they may disable these operators in their `advanced_search` endpoint. The pipeline depends on them for time-windowed pulls. If they go away, fallback is `user/last_tweets` with N=large + client-side filtering.

2. **No SLA**. When twitterapi.io has an outage, the Twitter pipeline gaps. The system handles this gracefully (the pipeline picks up where it left off when service returns) but cumulative gaps reduce signal.

3. **Public-only data**. twitterapi.io can't access protected tweets; this affects ~3 accounts in the follow list that have on-and-off lockdowns. We accept the gap.

### Why not switch to the official X API?

The math doesn't work for this corpus:

- 185 accounts × ~30 days × ~10 tweets/account/day = ~55K tweet-pulls/month
- Basic ($200/mo) caps at ~10K reads/mo — would need 5× Basic plans or 1 Pro ($5000/mo)
- Plus the auth complexity (OAuth bearer tokens, per-app rate buckets)

twitterapi.io at $30/mo with effectively no read cap is ~170× cheaper than the equivalent X Pro tier. The trade-off is the stability ceiling (a third-party scraper). For an internal intelligence pipeline where short outages are tolerable, that's the right call.

If you're building a customer-facing product that needs SLA, the X API is the right answer despite the cost. For a personal/team curated-follow intelligence agent, twitterapi.io is.

### Alternatives we evaluated and rejected

| Alternative | Why we didn't pick it |
|---|---|
| ScraperAPI | More expensive for the volume; cancelled March 2026 |
| Apify (twitter-scraper actor) | Pay-per-request pricing; cost equivalent to twitterapi.io but more brittle |
| Self-hosted snscrape | Free, but Twitter aggressively blocks unauthenticated scraping in 2026; effectively dead |
| RSS bridges (rsshub etc.) | Inconsistent uptime; per-account setup overhead |

---

## Stats

- **185+ accounts** total (130 macro/geo + ~55 Brazil)
- **20-minute cadence** — every account checked every 20 minutes via cron
- **~5,000 candidate tweets/day** scraped; **~1,500–2,000** survive trash + AI filter
- **~$30/mo** twitterapi.io ($1/account/year)
- **~$18/mo** Haiku scoring
- **~$4/mo** Voyage embedding
- **~$52/mo** total Twitter-pipeline cost — the lowest cost-per-signal of any source class in the pipeline

For context on the wider system this list feeds, see [`m3xa-core`](https://github.com/prcodex/m3xa-core).

---

## How to reproduce

This is a curated list — your version will differ. To build your own:

1. **Decide your domains.** What questions do you want answered? (Macro? Specific country? AI? Sports?) Each domain gets its own classification tag.
2. **Identify your tiers.** Real-time wires (high volume, low per-tweet signal density), Experts (low volume, high density), Officials (medium-medium), OSINT (low-low — keep tight).
3. **Use twitterapi.io for ingestion.** Cheap, simple, predictable. Switch to the official X API only if you need SLA or you're building a customer-facing product.
4. **Run a trash filter + AI ranker.** The two-stage drop (rule-based + LLM) is cheap (~$18/mo at this volume) and dramatically lifts the signal-to-noise of what reaches retrieval.
5. **Mark a small VIP set.** ~10% of accounts where missing a tweet costs more than indexing a marginal one. Skip the ranker for these.
6. **Store with explicit weights.** Don't bake your priority into hardcoded code; keep it in a config or DB that you can tune without redeploying.
7. **Test the boost at retrieval time, not at ingestion.** Ingestion stores everything that passes filtering; the retriever's job is to surface the right subset per query. Don't conflate the two.

The architecture for steps 4–7 is documented in [`m3xa-core`](https://github.com/prcodex/m3xa-core) — fork it as a starting point.

## License

MIT. See [LICENSE](LICENSE).

## Related

- [`m3xa-core`](https://github.com/prcodex/m3xa-core) — the House pattern (didactic reference)
- [`chunking_vector`](https://github.com/prcodex/chunking_vector) — RAG v5 Bedrock spec
- [`M3XABR_NEW`](https://github.com/prcodex/M3XABR_NEW) — expertise-composition slice
