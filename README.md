# scrape single page application: three methods that actually work, what JavaScript rendering really costs, and when to skip the headless browser

To scrape a single page application, you have three workable options: replay the JSON API the SPA pulls data from, run your own headless browser, or send the URL to a scraping API that renders JavaScript for you. The first is cheapest when it works, the second gives you full control but means maintaining Chromium infrastructure, and the third trades credits for engineering time. This guide covers how each route works, where they break, and — if you go the API route — the real per-request cost math behind ScraperAPI's plans, because the number on the pricing page and the number of pages you actually get are very different things.

> **Quick take:** for most developers scraping a few thousand to a hundred thousand pages a month from React/Vue/Angular sites, ScraperAPI's Hobby plan ($49/month, 100,000 credits) with the `render=true` parameter is the fastest path. A 7-day trial with 5,000 credits lets you confirm your target renders correctly before paying anything.
>
> 👉 [Start the 7-day trial and test JavaScript rendering on your target SPA](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Why a plain HTTP request returns an empty shell

A single page application builds its interface in the browser. The initial HTML you fetch is nearly empty — a root `<div>` and a pile of script tags. The actual data arrives after JavaScript runs, usually via calls to an internal JSON API. That's why `requests.get()` or `curl` on a React/Vue/Angular page gives you a blank document with no product data in it.

Every method below is a different answer to the same question: how do you get the data that only exists after the JavaScript runs? You can either intercept the data before it's rendered, execute the JavaScript yourself, or have someone else's browser execute it for you.

## Route 1: replay the API behind the SPA

This is the cheapest and most reliable route when the target allows it. Open the site in your browser, open the Network tab, filter for XHR/fetch requests, and load the page. You'll usually see requests to endpoints like `/api/products?page=1&per_page=50` returning clean JSON with everything the frontend displays.

Replaying that endpoint with a plain request costs you one normal HTTP call — no browser, no rendering overhead, no waiting on selectors. For stable, unauthenticated endpoints this beats everything else on this page hands down.

The catch: internal APIs aren't meant for you. They can change without notice, require session tokens or signed parameters, and some validate headers or reject anything that doesn't look like it came from the frontend. If the endpoint is locked down, move on to Route 2 or 3.

## Route 2: run your own headless browser

Puppeteer or Playwright driving headless Chromium executes the page's JavaScript for real, so you get exactly what a visitor sees. Launch a browser context with a realistic user agent and viewport, navigate, wait for a key selector to appear, then extract from the DOM. The setup is well documented and costs nothing in licensing.

What most tutorials underplay is the ongoing operational load. Browsers leak memory over long runs. You need your own proxy rotation when targets rate-limit by IP. CAPTCHAs are your problem. And each rendered page is slow — you're loading a full browser for every URL. For a one-off project, fine. For a pipeline you maintain weekly, it becomes a part-time job, which is precisely the trade a rendering API is priced against.

## Route 3: a scraping API renders JavaScript for you

Add one parameter and the heavy lifting moves off your machine. With ScraperAPI, you send your target URL with `render=true`, and the request routes through a Chromium instance that executes the page's JavaScript before returning the rendered HTML. Proxy rotation, retries, CAPTCHA handling, and header management happen on their side — no proxy lists, no Playwright configuration, no solver subscriptions.

Two details from their documentation matter a lot for SPA scraping, and most reviews skip them entirely:

1. **Missing elements have a fix.** To keep requests fast, the API doesn't render everything on the page by default — it skips images and tracking scripts, and sometimes that skips data you actually need. The `wait_for_selector=x` parameter tells it to wait until a specific selector appears before returning the response. If your first render comes back without the data, this is usually why.
2. **Rendering has its own throttle.** The render burst limit is 10 requests per second by default. If each rendered request takes 25 seconds and you fire 10 per second, you can have up to 250 rendering requests running concurrently. Enterprise users can arrange higher limits with support.

The rendered response also supports an instruction set — clicks, form input, scrolling, waiting for specific events — so you can handle SPAs that need interaction before the data appears, all through parameters on the same request.

## What JavaScript rendering actually costs: credits are not requests

This is the part that trips people up. ScraperAPI prices in credits, and one credit equals one standard page request. But the cost per request depends on the target and your parameters. Their official pricing FAQ is explicit: a standard page costs 1 credit, Amazon pages cost 5, Google and Bing cost 25, LinkedIn costs 30, and sites behind Cloudflare, Datadome, or PerimeterX add 10 credits per request when their protection is bypassed.

JavaScript rendering adds on top: **every JS-rendered request costs 10 credits**. With premium proxies it's 25; with ultra premium proxies, 75. Their docs even recommend using rendering only when you absolutely need it, because it adds latency and can lower success rates.

Run the math on the Hobby plan's 100,000 credits. On static pages, that's 100,000 real fetches. Turn on JS rendering for a typical SPA workload and it's about 10,000 rendered pages. Same plan, one-tenth the output — and that gap is exactly what most competitor articles don't spell out. Before committing to any plan size, open the dashboard's Domain Cost Estimator, check the exact per-URL cost for your targets, and set `max_cost` per scrape so a single request can't quietly eat more credits than you expect.

👉 [Check plan sizes against your real request volume on the pricing page](https://www.scraperapi.com/pricing?fp_ref=coupons)

## All current plans at a glance

Every paid plan includes JS rendering, premium residential and mobile IPs, rotating proxy pools, CAPTCHA and anti-bot handling, automatic retries, JSON auto parsing, and unlimited bandwidth. Annual billing takes 10% off every tier. Here's the full lineup as listed on the official pricing page:

| Plan | Monthly price | Annual (per month) | API credits/mo | Concurrency | Get it |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 (5 concurrent) | 5 | [Start free](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Contact sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

A few billing details that only show up in the FAQ, all of which affect real usage:

- **Trial before commitment:** a 7-day trial with 5,000 credits, no credit card required. Sign-up alone gets you 1,000 free credits at 5 concurrent connections, and support can grant extra testing credits.
- **When credits run out:** scraping pauses until renewal by default. Hobby through Professional offer opt-in auto-upgrade; Scaling and above offer pay-as-you-go at a fixed per-credit rate with a monthly cap you set.
- **Concurrency overruns are free:** exceeding your thread limit returns a 429 and the request is turned away rather than queued — no extra charge for trying.

## How ScraperAPI compares for SPA work

Against doing it yourself, the comparison is honest: you're paying credits to avoid proxy management, browser maintenance, and CAPTCHA infrastructure. One reviewer's framing is hard to improve on — raw residential proxies plus your own Playwright cluster cost less per request at high volume, but it's trading money for engineering time. If your monthly volume sits in the 1,000–100,000 page range and your targets are JavaScript-heavy, that trade usually favors the API.

Against enterprise-grade tools, the picture is more mixed. Bright Data's Web Unlocker measures around 95–98% success on protected targets in independent benchmarks, versus roughly 62–72% overall for ScraperAPI (two separate benchmarks: one at 72.57%, another around 62.8–63.7%) — but Bright Data costs meaningfully more and carries more setup friction. On Cloudflare and DataDome targets specifically, reviewers report ScraperAPI sometimes returning outright 500 errors where stronger tools get through. ScrapeOps, a proxy aggregator that routes through multiple providers per request, makes sense if you already handle rendering yourself and want raw proxy access.

None of these are links you need — the point is that the right tool depends on how hard your targets fight back. For the easy-to-moderate 80% of SPA scraping, ScraperAPI's combination of a simple endpoint and built-in rendering is a reasonable price for not running your own browser fleet.

## Where people get burned (and how to avoid it)

Two failure modes recur in third-party reviews, and both have workarounds you can set up on day one.

First, CAPTCHA or error pages sometimes come back coded as "successful" responses — and get billed as credits. Reviewers recommend spot-checking responses for CAPTCHA markers rather than trusting the HTTP status. The `max_cost` parameter doesn't fix this directly, but reviewing your dashboard's per-domain costs makes anomalies visible quickly. If you do get incorrectly billed, support refunds it once flagged.

Second, speed. Independent benchmarks logged 20–40 seconds per request on targets like Amazon and Glassdoor. For a price-monitoring job where timing matters, that latency plus the Hobby plan's 20-thread cap is a real constraint. Batch jobs scheduled through their DataPipeline don't care; a dashboard that needs fresh prices every minute does.

One more thing worth knowing before you commit: there's no meaningful free tier. The free 1,000 credits are for testing, and the 7-day trial is the real evaluation window. Run your actual targets through it with `render=true` and the correct `wait_for_selector` values, count the credits each page really consumes, then size your plan from measured data.

## Picking a plan and place to start

Match the plan to your measured volume, not the credits number:

- **Testing or a small one-off:** start with the free 1,000 credits, then the 7-day trial's 5,000. If your target needs rendering, that's only 500 rendered pages — enough to validate the approach.
- **A side project or small pipeline:** Hobby at $49/month ($44.10 on annual). Keep JS rendering off where the API replay route works, spend rendering credits only where you must, and 100,000 credits goes a long way.
- **Multi-client or production workloads:** Startup or Business, where per-credit cost drops sharply ($0.149 down to $0.10 per thousand at Business) and concurrency stops being the bottleneck.
- **Heavily protected targets at scale:** be honest with yourself about the benchmark numbers. If your list is mostly Cloudflare/DataDome sites, a stronger (and pricier) unlocker or a self-managed proxy setup will serve you better than a bigger plan here.

Whatever tier you land on, do the three checks first — replay-or-render test, Domain Cost Estimator readout, and a small-batch success-rate check during the trial — because those numbers, not the pricing page's headline credits, determine what you actually need.

👉 [Open the plans page and start your trial with 5,000 free credits](https://www.scraperapi.com/pricing?fp_ref=coupons)
