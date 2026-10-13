# WordPress VPS Hosting: Sizing the Box, Picking a Stack, and Choosing a Route That Holds Up at Peak Hours

Most people typing "WordPress VPS hosting" into a search bar are not confused about what a VPS is. They are stuck on one of four questions: how much server do I actually need, do I want to run the server myself, what should it cost, and — depending on where their readers are — which region and network route keeps the site from turning into a slideshow at 9pm local time.

Those four questions pull in different directions, and the answer changes the moment the audience is in Asia rather than Ohio. This piece works through the sizing first, then the operating model, then a provider whose entire price tag sits in the network layer rather than the specs sheet: DMIT.

## Sizing WordPress by workload, not by visitor count

The instinct is to size by traffic. That instinct is usually wrong. A cached WordPress page is close to a static file — nginx hands it over and PHP never wakes up. The expensive requests are the ones that can't be cached: logged-in users, cart and checkout pages, search, admin work, REST calls, and anything a plugin decides to do on a page load.

What that means in practice is that a site with 500 visitors a day running a heavy page builder and a security scanner can need more server than a site with 5,000 visitors and proper page caching. Plugin count is a better proxy for memory use than sessions per day.

The widely cited planning ranges look like this:

| Site type | Working RAM | Notes |
| --- | --- | --- |
| Clean WordPress, light theme, ~10 plugins | 1 GB, tight | Works with swap configured; not much room to grow |
| Small business site, page builder, 10–15 plugins | 2 GB | The sane practical baseline |
| WooCommerce, membership, or 25+ plugins | 4 GB | Cart and checkout requests bypass page cache |
| Several client sites on one box | 4–8 GB | Add margin for per-site PHP pools and backups |

Two numbers matter more than the plan name. First, per-process memory: a lean request costs 40–80 MB, while a page-builder-plus-WooCommerce request can cost 150–250 MB. Second, how many PHP workers run at once — the parameter hosting comparison tables almost never list, and the one that decides whether your site survives a spike. Take your spare RAM, divide by observed per-worker usage, and leave headroom for the database and object cache.

A short list of signals that shared hosting has stopped being enough: admin pages taking more than 3–4 seconds while public pages stay fast, a provider email about CPU quota, or the inability to install a PHP module you need. None of those are fixed by adding RAM. They're fixed by moving.

## What you're actually taking responsibility for on a VPS

A WordPress VPS runs on a server where you control the OS, web server, PHP runtime, database, cache, firewall, and backup schedule. Shared and managed hosting hide that layer. On a VPS it's the job, and the job has a shape:

- **nginx** terminating TLS, serving static assets, and proxying PHP over a Unix socket rather than TCP
- **PHP-FPM** on a current branch (8.3 or 8.4), with `pm.max_children` sized from real memory and `pm.max_requests` recycling workers to contain plugin memory leaks
- **MySQL or MariaDB**, with `innodb_buffer_pool_size` set so the hot working set lives in RAM — WordPress's `wp_options` and `wp_postmeta` tables get hammered on every request
- **A persistent object cache** (Redis, or the protocol-compatible Valkey fork) behind the Redis Object Cache plugin. This is the single highest-leverage change for logged-in and dynamic traffic
- **Page caching** — nginx `fastcgi_cache` or a plugin — with honest bypass rules for logged-in users, commenters, and active carts
- **Real cron** instead of WP-Cron, which only runs when a visitor happens to load a page
- **Hardening**: block `xmlrpc.php`, rate-limit `wp-login.php`, correct file permissions, automatic minor core updates, and a tested restore

That last item is not a checkbox. WordPress security reporting through 2025 counted thousands of new vulnerabilities, the overwhelming majority in plugins, and exploits frequently land within hours of disclosure. An unmaintained VPS is a worse home for WordPress than a mediocre managed host, because the target is well known.

The honest trade: a managed WordPress plan runs $25–45 per month per site, while the equivalent stack fits on a $6–12 VPS — but *you* are the person answering at 3am. If nobody on the team owns OS updates, backups, and monitoring, a VPS is the wrong purchase no matter how good the specs are.

## Where DMIT fits — and where it doesn't

DMIT is not a managed WordPress host, and it will not pretend to be. It's a US-registered KVM virtual server provider selling instances in Los Angeles, Hong Kong, and Tokyo, running AMD EPYC hardware with SSD storage, on top of network routes that cost real money to buy. Support handles Chinese and English, and payment runs through PayPal, Alipay, and credit cards — useful if the person paying isn't holding a US credit card.

For plain WordPress hosting on a budget, DMIT is expensive per gigabyte of RAM. The $10.90 entry plan in Los Angeles buys 2 GB; a generic cloud droplet at that price can give you more disk and similar memory. What you don't get from a generic cloud provider is a route to mainland China that doesn't collapse in the evening. That's the entire product thesis, and it's worth being clear-eyed about it before buying.

### Pro, Eyeball, and Tier 1 — what the routing names mean

Every DMIT region is split into network profiles rather than being one product line:

- **Premium (Pro)** — the expensive one. In Los Angeles that means China Telecom and China Unicom return traffic over CN2 GIA (AS4809), outbound via GIA and direct AS4837, and China Mobile outbound via Hong Kong Mobile AS58453. In Hong Kong and Tokyo the detail differs but the positioning is the same: everything routed for mainland Asia, at a premium.
- **Eyeball (EB)** — one step down, built around CMIN2 and AS9929 for return traffic. Cheaper than Pro, generally with larger monthly traffic allowances at similar specs. It's the value pick.
- **Tier 1 (T1)** — international BGP with no mainland optimisation at all. Cheapest, biggest ports, largest traffic quotas. Completely fine for a WordPress site whose readers are in Europe or North America, and the wrong choice for a site that needs to feel fast inside China.

One design detail worth knowing: on most series, blowing through the monthly traffic quota throttles the port to a stated low speed instead of suspending the server. For a blog that occasionally spikes, that's a much friendlier failure mode than going dark. Terms differ between series, so read the line on the order page.

### Los Angeles (LAX) plans

The LAX.Pro ladder on the AMD EPYC 7003 (AS3) platform, priced monthly:

| Plan | vCPU / RAM | SSD | Monthly traffic / port | After quota | Price (USD) | Billing | Buy |
| --- | --- | --- | --- | --- | --- | --- | --- |
| LAX.Pro.TINY | 1 / 2 GB | 20 GB | 1 TB @ 1 Gbps | 4 Mbps unlimited | $10.90 | monthly | [Check the LAX.Pro TINY price](https://www.dmit.io/aff.php?aff=12046) |
| LAX.Pro.Pocket | 2 / 2 GB | 40 GB | 1.5 TB @ 4 Gbps | 4 Mbps unlimited | $16.90 | monthly | [See the LAX.Pro Pocket plan](https://www.dmit.io/aff.php?aff=12046) |
| LAX.Pro.STARTER | 2 / 2 GB | 80 GB | 3 TB @ 10 Gbps | 4 Mbps unlimited | $34.90 | monthly | [View the LAX.Pro STARTER plan](https://www.dmit.io/aff.php?aff=12046) |
| LAX.Pro.MINI | 4 / 4 GB | 80 GB | 5 TB @ 10 Gbps | 8 Mbps unlimited | $62.90 | monthly | [Compare the LAX.Pro MINI plan](https://www.dmit.io/aff.php?aff=12046) |
| LAX.Pro.MICRO | 4 / 4 GB | 160 GB | 7 TB @ 10 Gbps | 8 Mbps unlimited | $87.90 | monthly | [See the LAX.Pro MICRO plan](https://www.dmit.io/aff.php?aff=12046) |
| LAX.Pro.MEDIUM | 6 / 8 GB | 160 GB | 14 TB @ 10 Gbps | 10 Mbps unlimited | $199.90 | monthly | [Check the LAX.Pro MEDIUM plan](https://www.dmit.io/aff.php?aff=12046) |

Two moving parts here. DMIT has been rotating its Los Angeles hardware onto the newer AN5 platform (AMD EPYC 9005), where the same plan names carry higher prices — TINY has been reported around $12.98/month on AN5 against $10.90 on AS3 — and can be paid quarterly as well as monthly. The older AS3 and AN4 generations come and go in and out of stock, and DMIT's own product pages have flagged AS3 as still being optimised, meaning disk performance and SLA may not match a mature platform. If a plan matters to you, buy it when you see it in stock rather than planning a purchase a month out.

The **LAX.EB** series walks the same spec ladder — TINY through MEDIUM — at the same monthly headline prices but with bigger traffic allowances, and it appears in annual form too: EB.WEE around $39.90/year and EB.CORONA around $49.90/year with 2 TB of monthly traffic and a 2 Gbps port are the cheapest ways into the Los Angeles Eyeball network when stock exists.

### Hong Kong (HKG) plans

Hong Kong Premium is the highest-value option for a China-facing WordPress site — physical proximity beats route engineering, which is why these plans cost more than the LAX equivalents for less hardware.

| Plan | vCPU / RAM | SSD | Monthly traffic | Price (USD) | Billing | Buy |
| --- | --- | --- | --- | --- | --- | --- |
| HKG.Pro.TINY | 1 / 1 GB | 20 GB | 500 GB @ 1 Gbps | $39.90 | monthly | [Check the HKG.Pro TINY price](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.STARTER | 1 / 2 GB | 40 GB | 1 TB @ 1 Gbps | $79.90 | monthly | [See the HKG.Pro STARTER plan](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MINI | 2 / 2 GB | 60 GB | 1.5 TB @ 1 Gbps | $126.90 | monthly | [View the HKG.Pro MINI plan](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MICRO | 2 / 4 GB | 80 GB | 2 TB @ 1 Gbps | $179.90 | monthly | [Compare the HKG.Pro MICRO plan](https://www.dmit.io/aff.php?aff=12046) |
| HKG.Pro.MEDIUM | 4 / 4 GB | 160 GB | 2.5 TB @ 1 Gbps | $239.90 | monthly | [See the HKG.Pro MEDIUM plan](https://www.dmit.io/aff.php?aff=12046) |

Hong Kong also has a newer AN5 Premium line with larger disks and different pricing again, and the Eyeball series has been withdrawn from the Hong Kong lineup entirely. As with Los Angeles, stock rotates.

### Tokyo (TYO) plans

Tokyo Pro carries CN2 GIA on the return path with AS9929 for Unicom and CMI for Mobile. It's limited-supply by DMIT's own account, so the entry plans appear and disappear.

| Plan | vCPU / RAM | SSD | Monthly traffic | Price (USD) | Billing | Buy |
| --- | --- | --- | --- | --- | --- | --- |
| TYO.Pro.TINY | 1 / 1 GB | 20 GB | 300–500 GB @ 1 Gbps | $21.90 | monthly | [Check the TYO.Pro TINY price](https://www.dmit.io/aff.php?aff=12046) |
| TYO.Pro.STARTER | 1 / 2 GB | 40 GB | 1 TB @ 1 Gbps | $39.90 | monthly | [See the TYO.Pro STARTER plan](https://www.dmit.io/aff.php?aff=12046) |
| TYO.Pro.MINI | 2 / 2 GB | 60 GB | 2 TB @ 1 Gbps | $79.90 | monthly | [View the TYO.Pro MINI plan](https://www.dmit.io/aff.php?aff=12046) |
| TYO.Pro.MICRO | 4 / 4 GB | 80 GB | 4 TB @ 1 Gbps | $159.90 | monthly | [Compare the TYO.Pro MICRO plan](https://www.dmit.io/aff.php?aff=12046) |
| TYO.Pro.MEDIUM | 4 / 8 GB | 160 GB | 5 TB @ 1 Gbps | $259.90 | monthly | [See the TYO.Pro MEDIUM plan](https://www.dmit.io/aff.php?aff=12046) |
| TYO.Pro.LARGE | 8 / 16 GB | 320 GB | 8 TB @ 1 Gbps | $429.90 | monthly | [Check the TYO.Pro LARGE plan](https://www.dmit.io/aff.php?aff=12046) |
| TYO.Pro.GIANT | 8 / 24 GB | 640 GB | 15 TB @ 1 Gbps | $799.90 | monthly | [View the TYO.Pro GIANT plan](https://www.dmit.io/aff.php?aff=12046) |

### Tier 1 across all three regions

Tier 1 is the same ladder everywhere — Los Angeles, Hong Kong, Tokyo — with international BGP routing and no mainland optimisation. It's also where the price-per-resource stops looking absurd, and it's the sensible pick for WordPress sites aimed at readers outside China.

| Plan | vCPU / RAM | SSD | Monthly traffic / port | After quota | Price (USD) | Billing | Buy |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1.WEE | 1 / 1 GB | 20 GB | 1 TB @ 4 Gbps | 50 Mbps unlimited | $36.90 | yearly | [Check the T1.WEE annual plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.TINY | 1 / 1 GB | 20 GB | 2 TB @ 4 Gbps | 100 Mbps unlimited | $6.90 | monthly | [See the T1.TINY plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.STARTER | 1 / 2 GB | 40 GB | 4 TB @ 10 Gbps | 100 Mbps unlimited | $12.90 | monthly | [View the T1.STARTER plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.MINI | 2 / 2 GB | 60 GB | 8 TB @ 10 Gbps | 100 Mbps unlimited | $21.90 | monthly | [Compare the T1.MINI plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.MICRO | 4 / 4 GB | 80 GB | 16 TB @ 10 Gbps | 200 Mbps unlimited | $32.90 | monthly | [See the T1.MICRO plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.MEDIUM | 4 / 8 GB | 160 GB | 32 TB @ 10 Gbps | 200 Mbps unlimited | $49.90 | monthly | [Check the T1.MEDIUM plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.LARGE | 8 / 16 GB | 320 GB | 64 TB @ 10 Gbps | 500 Mbps unlimited | $99.90 | monthly | [View the T1.LARGE plan](https://www.dmit.io/aff.php?aff=12046) |
| T1.GIANT | 8 / 24 GB | 640 GB | 128 TB @ 10 Gbps | 1 Gbps unlimited | $199.90 | monthly | [Compare the T1.GIANT plan](https://www.dmit.io/aff.php?aff=12046) |

Note that traffic counts in both directions on these plans. If your site serves large images or video, the effective quota is roughly half the number on the page. Sizing assumptions for the smallest Tier 1 plans also vary between 1 GB/20 GB and 2 GB/40 GB depending on which generation is in stock — worth confirming on the order page rather than trusting a blog table, including this one.

## Matching a plan to a WordPress site

**A single low-traffic blog with readers outside China.** T1.TINY at $6.90/month. One gigabyte of RAM is tight; run nginx with a page cache and no control panel, or step up to T1.STARTER at $12.90 for the 2 GB.

**A small business site whose readers are mostly in Asia.** LAX.Pro.TINY at $10.90/month is the usual starting point, and its 4 Mbps throttled fallback keeps you online if traffic runs over. If your readers are in Hong Kong or southern China and the budget stretches, HKG.Pro.TINY at $39.90/month buys you real proximity — a legitimate price to pay only if the audience justifies it.

**WooCommerce.** Skip anything with 2 GB. T1.MICRO at $32.90 or LAX.Pro.MINI at $62.90 gives you the 4 GB you need once cart and checkout requests stop being cacheable. The extra money in LAX.Pro buys route quality, not more compute.

**An agency running several client sites.** T1.MEDIUM (4 vCPU, 8 GB, 32 TB) at $49.90/month on Tier 1 hardware, with per-site PHP pools and a control panel of your choosing. Isolate the sites and write down who restores a backup before you sell the third one.

## Billing, discounts, and what to confirm yourself

DMIT runs periodic recurring discount codes that are applied in the checkout page's Validate Code box. Many coupon lists circulating online are tied to specific product lines and expiry windows that have already passed, so treat any code you find as unverified until the order page accepts it and shows a lower total. The reliable levers are boring ones: billing in longer terms generally costs less than paying monthly, and stock availability decides what you can actually buy on any given day.

Because the hardware platforms are being refreshed region by region, plan names that share a name across AS3, AN4, and AN5 generations are not the same product at the same price. Check the platform label on the order page before comparing numbers against anything you read elsewhere — including the tables above, which reflect the lineups as they stood when this was written.

## Before you install WordPress on one of these

These are unmanaged KVM servers. There's no WordPress button to click on the provider side, no plugin support, and no one to tell you which of your twelve plugins caused the white screen. The work is yours or your developer's.

A short pre-flight list:

1. Pick a current Ubuntu LTS or Debian image and update it fully before anything else.
2. Install nginx, PHP-FPM on 8.3 or 8.4, and MariaDB or MySQL. Set `pm.max_children` from measured per-worker memory, not from optimism.
3. Add Redis or Valkey and the Redis Object Cache plugin, then check your query count per page. On plugin-heavy sites the drop is large.
4. Add page caching with bypass rules for logged-in users and carts, then verify with a `curl -I` that you see cache HITs for anonymous visitors.
5. Disable WP-Cron and run `wp cron event run --due-now` from system cron every minute.
6. Block `xmlrpc.php`, rate-limit `wp-login.php`, enable 2FA for admin accounts.
7. Set up off-server backups — `wp db export` plus an archive of `wp-content`, shipped somewhere that isn't the same disk — and actually restore one to a scratch machine before you trust it.

Do that and a 2 GB instance will outrun a $30/month managed plan for most content sites. Skip it and the nicest route in the world won't keep your site online.

## FAQ

**Is 1 GB of RAM enough for WordPress on a VPS?**
For a cached site with a light theme and around ten plugins, yes, with swap configured. It stops being enough the moment you add a page builder, an active cart, or an admin panel running on the same box. 2 GB is the comfortable baseline.

**Does DMIT offer managed WordPress hosting?**
No. DMIT sells unmanaged KVM virtual servers with root access. If you want someone else to handle updates, patching, and plugin conflicts, a managed WordPress host is a different category of purchase.

**Can I run multiple WordPress sites on one DMIT plan?**
Yes — nothing in the plan structure limits site count, only resources do. On Tier 1 hardware, T1.MEDIUM with 8 GB is a reasonable starting point for a handful of low-to-medium traffic sites if you use per-site PHP pools.

**What happens when I exceed the monthly traffic?**
On most series the port is throttled to a slow-but-unlimited speed — 4 to 10 Mbps on Los Angeles Pro and Eyeball, 50 Mbps to 1 Gbps on Tier 1 — instead of the server being suspended. Terms differ between series, so check the row on the order page.

**Do the plans include cPanel or backups?**
Neither. Plan prices cover the virtual server and its network. A cPanel licence runs around $30/month on its own and adds memory overhead; lighter panels cost $10–12/month. Backups are your responsibility, and they need to leave the server.

If the audience for your WordPress site sits in Asia and your current host is the thing making it slow, the route is the product — and that's the only reason the prices above make sense.

👉 [Compare DMIT's Los Angeles, Hong Kong, and Tokyo VPS plans](https://www.dmit.io/aff.php?aff=12046)
