# web scraping c#: build your first scraper with HttpClient, then handle the blocking problem with real pricing math

# is a solid choice for web scraping in 2026, especially if your scraper lives inside a larger .NET app. HttpClient ships with the runtime, HtmlAgilityPack and AngleSharp handle HTML parsing, and Playwright for .NET covers JavaScript-rendered pages. But here's the part most tutorials gloss over: writing the scraper is the easy 20%. Not getting blocked across thousands of requests is the other 80%, and that's where a scraping API like ScraperAPI earns its keep.

# stack, the pitfalls that break .NET scrapers under load. Then the scaling problem, and what it actually costs to solve it: full ScraperAPI plan pricing, the credit consumption rules that most reviews never explain, and which plan fits which situation.

> **Quick picks**
> - Just learning, or scraping open sites? HttpClient + HtmlAgilityPack is all you need. No paid anything.
# parsing code and route the fetch through ScraperAPI. The **Hobby plan is $49/month for 100,000 credits**, and every paid plan includes JS rendering, residential/mobile proxies, and CAPTCHA handling.
> - New accounts get a **7-day free trial with 5,000 credits, no credit card required**, plus 1,000 free monthly credits on the free plan.
> - 👉 [Start the free trial and test ScraperAPI against your target sites](https://www.scraperapi.com/?fp_ref=coupons)

## The C# scraping stack: what to use, and when to level up

# scraping job, and the right one depends on whether the data sits in the initial HTML.

| Tool | Role | Handles JavaScript? | Difficulty |
| --- | --- | --- | --- |
| HttpClient | Fetching pages and JSON APIs (built into .NET) | No | Easy |
| HtmlAgilityPack | Parsing messy HTML with XPath | No | Easy |
| AngleSharp | Parsing with CSS selectors, standards-compliant DOM | No | Medium |
| Playwright for .NET | Driving a real headless browser | Yes | Medium |

Selenium WebDriver is a fifth option for legacy or cross-browser needs, and it has the largest pool of existing examples. Current stable versions on NuGet as of June 2026: HtmlAgilityPack 1.12.4, AngleSharp 1.5.0, Microsoft.Playwright 1.60.0, CsvHelper 33.1.0. On a real project, pin versions with `--version` so a library update doesn't silently change parsing behavior under you.

The decision rule is simple. Start with HttpClient plus HtmlAgilityPack — or AngleSharp if you prefer CSS selectors over XPath. Only reach for Playwright when the data genuinely isn't in the initial HTML and the page is built by JavaScript. Every step up costs speed and adds complexity, and HttpClient plus a parser handles most real sites.

# wins when the scraper is part of a larger typed system — an ASP.NET app, a worker service, an Azure pipeline — rather than a standalone script.

## Building a first C# scraper: the setup and the traps

The basic skeleton is a four-step loop: fetch with HttpClient, parse with HtmlAgilityPack, extract with XPath, write out with CsvHelper. Setting it up takes three commands:

bash
dotnet new console -n BookScraper
cd BookScraper
dotnet add package HtmlAgilityPack --version 1.12.4
dotnet add package CsvHelper --version 33.1.0


HttpClient needs no package at all; it lives in `System.Net.Http` in the runtime. A minimal fetch-and-parse against a practice sandbox like books.toscrape.com runs about 15 lines: `GetStringAsync` for the page, then XPath queries like `//article[@class='product_pod']` to pull titles and prices, with `HtmlEntity.DeEntitize()` to clean up encoded entities like `&amp;`.

Three details separate a working scraper from one that breaks on the first real site:

1. **Reuse one HttpClient instance.** Creating a fresh HttpClient per request is the classic .NET mistake — under load it exhausts sockets. Declare it `static readonly`, or register it through `IHttpClientFactory` for long-running services.
2. **Set a browser-style User-Agent.** The default .NET client string gets blocked by many sites outright. Adding a realistic `User-Agent` header is a one-liner that prevents a whole category of 403s.
3. **Check status codes.** Call `EnsureSuccessStatusCode()` or use `GetAsync` so a 403 or 404 error page never gets parsed as if it were data.

For pagination, don't hardcode page counts. Loop until a page returns no results, track visited URLs in a `HashSet`, and throttle with `Task.Delay` of one to a few seconds — ideally randomized. Firing hundreds of requests per second is the fastest way to get an IP rate-limited or banned, which brings us to the real problem.

## Where DIY C# scraping stops working

# scraping works fine on friendly sites and falls apart on protected commercial targets. The numbers are stark. Independent testing by Scrapeway in their July 2026 benchmark found that even the best commercial anti-bot APIs top out around 99% success against protected targets — while cheaper tools land far lower, with Proxyway measuring Firecrawl near 33% on defended sites. A C# scraper with no proxy rotation does much worse than any of them.

Vendor success-rate claims don't help much either. ScraperAPI's own quick-start guide points out that published web scraping success rates run anywhere from 40% to 100%, and the only number that matters is the one against your target sites — which is exactly why testing before committing matters.

# scrapers keep their parsing code and swap out the fetch layer.

## How ScraperAPI plugs into a C# scraper

# code calls its endpoint with your API key and the target URL, and it returns the page HTML after handling proxy rotation, CAPTCHAs, and (optionally) JS rendering on its side.

# SDK — ScraperAPI ships SDKs for Python, Node, PHP, Ruby, and Java — but since it's a REST endpoint, HttpClient is all a C# project needs:

csharp
static readonly HttpClient http = new HttpClient();

string html = await http.GetStringAsync(
    "https://api.scraperapi.com/?api_key=YOUR_KEY" +
    "&url=https://example.com/product-page&render=true");


# code. There's also a DataPipeline option for teams who want scheduled collection without writing scraping code at all.

The practical integration pattern: keep HtmlAgilityPack for parsing, and decide per request whether to route the fetch through ScraperAPI based on how hostile the target is. Friendly internal or open sites stay on plain HttpClient; protected commercial targets go through the API. To see how your own targets behave before paying anything, 👉 [claim the 5,000-credit free trial and run it against your real target URLs](https://www.scraperapi.com/?fp_ref=coupons).

## ScraperAPI plans and pricing: the full table

All paid plans include the same feature set — JS rendering, premium proxies, structured data endpoints, CAPTCHA handling, unlimited bandwidth, a 99.9% uptime SLA. The tiers differ in credits, concurrency, and geotargeting. Annual billing takes 10% off across the board.

| Plan | Price (monthly / billed annually) | Credits / month | Concurrency | Buy |
| --- | --- | --- | --- | --- |
| Free | $0 | 1,000 | 5 | [ 注册免费账户](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 / $44.10 per month | 100,000 | 20 | [ 开通 Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 / $134.10 per month | 1,000,000 | 50 | [ 开通 Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 / $269.10 per month | 3,000,000 | 100 | [ 开通 Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling (marked most popular) | $475 / $427.50 per month | 5,000,000 | 200 | [ 开通 Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 / $877.50 per month | 10,500,000 | 300 | [ 开通 Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 / $1,777.50 per month | 21,500,000 | 500 | [ 开通 Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom quote | 22,000,000+ | 500+ | [ 联系销售定制](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

Geotargeting is the quiet differentiator most comparison tables skip:

- **Hobby and Startup**: US and EU locations only
- **Business and up**: country-level geotargeting anywhere in the 50+ country pool

New users can apply the promo code **START10** at checkout for 10% off the first month on any subscription, on top of the annual discount. For a Hobby plan billed annually, that works out to roughly $39.69 for the first month before settling at $44.10.

## The credit math most reviews skip

"100,000 credits for $49" sounds straightforward until you read the consumption rules, because a credit is not always a page.

The base rate is 1 credit per standard request. Then the multipliers kick in:

| What you request | Credits per request |
| --- | --- |
| Standard page | 1 |
| Amazon (custom scraper) | 5 |
| Google or Bing, all subdomains | 25 |
| LinkedIn | 30 |
| Cloudflare / Datadome / PerimeterX bypass | +10 |
| `render=true` (JS rendering) | 10 total |
| `premium=true` + `render=true` | 25 total |
| `ultra_premium=true` + `render=true` | 75 total |

Run the numbers on a Hobby plan and the picture changes. 100,000 credits is 100,000 standard pages — or about 20,000 Google results pages, or 4,000 Google pages with rendering enabled. If your project scrapes SERP data daily, Hobby runs dry in weeks, and knowing that before checkout beats discovering it on invoice day.

Three mechanisms keep this manageable, and they're the kind of detail that only shows up in the documentation:

- **Only successful requests are billed** — status codes 200 and 404. Failed fetches due to blocks or timeouts don't burn credits, unless you cancel a request before its 70-second completion limit.
# code can log real costs per target site as it runs.
- **Cost estimation before you scrape.** The dashboard's Domain Cost Estimator tells you what any URL will cost, and there's a programmatic endpoint at `https://api.scraperapi.com/account/urlcost` that accepts all parameters — handy for checking what enabling `render=true` does to a specific target. You can also set a `max_cost` per scrape so no single request blows past a limit you choose.

Exceeding your concurrency limit returns a 429 and charges nothing. Running out of credits mid-month pauses scraping by default; you can pre-purchase more, enable auto-upgrade (available on Hobby through Professional), or turn on pay-as-you-go (Scaling and up) to keep scraping at a fixed per-credit rate under a spending cap you set.

## Which plan actually fits your project

Match the plan to your target sites and monthly volume, and the decision mostly makes itself:

- **Learning, side projects, or evaluating**: the free plan's 1,000 monthly credits plus the 5,000-credit trial cost nothing and require no card. That's enough to test ScraperAPI's success rate against your specific targets.
- **Standard pages, US/EU targets, under ~100k requests a month**: Hobby at $49 does the job. Just note the geotargeting ceiling — if you need Southeast Asian or Latin American localized data, Hobby can't provide it.
- **SERP or Amazon data, or non-US/EU geotargeting**: start at Business ($299). Below that tier, the 25–30 credit domain multipliers and the US/EU-only locations will both bite. A million credits on Startup sounds generous until Google's 25-credit rate turns it into 40,000 searches.
- **Production pipelines and agencies**: Scaling at $475 is the tier ScraperAPI itself marks as most popular, and it's the first one with pay-as-you-go headroom past your credit allowance.

Before you commit to any tier, do one thing: run your five most important target URLs through the cost estimator, multiply by your monthly volume, and compare against the credit allowances in the table above. 👉 [Open the plan comparison and check costs for your own targets](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## FAQ

**Is there an official ScraperAPI SDK for C#?**
# projects use the REST endpoint directly with HttpClient, which works fine since the API is plain HTTP in and HTTP out — the response header `sa-credit-cost` even reports credit usage per request.

**How much does scraping Google with ScraperAPI cost?**
25 credits per request against Google or Bing including all subdomains, before parameters. Adding JS rendering on top brings a Google request to 35 credits. Check your exact targets with the API Playground or the `urlcost` endpoint before building around an assumption.

**Does ScraperAPI charge for failed requests?**
No. Only successful responses (status 200 and 404) consume credits, and so do requests you cancel before the 70-second completion window. Exceeding your concurrency limit returns a 429 with no charge.

**Is there a free way to test it with C#?**
Yes — a 7-day trial with 5,000 credits, no credit card, plus a permanent free plan with 1,000 credits a month and 5 concurrent connections. Both are enough to wire up an HttpClient call and measure real success rates on your targets.

**Do any discounts stack?**
Annual billing takes 10% off every paid plan, and the START10 promo code takes another 10% off the first month for new users at checkout.

## The bottom line before you sign up

# scraper only touches open, friendly sites, spend nothing — HttpClient and HtmlAgilityPack will carry you. The moment you're hitting Amazon, Google, LinkedIn, or anything behind Cloudflare at volume, routing the fetch through ScraperAPI from $49/month is cheaper than the developer weeks a DIY proxy stack consumes, and the trial lets you verify that on your own URLs before paying.

Three things worth double-checking at checkout: estimate the credit cost of your actual targets first (SERP sites burn 25x faster than standard pages), confirm the geotargeting you need is included at your tier — Hobby and Startup are US/EU only — and decide between auto-upgrade and pay-as-you-go so a busy month doesn't silently pause your pipeline. 👉 [Start with the free 5,000-credit trial and pick your plan from real data](https://www.scraperapi.com/?fp_ref=coupons)
