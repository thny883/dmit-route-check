# low latency vps: how to check the route before you buy, which region to pick, and what DMIT's plans actually cost

Every VPS page says "low latency". Almost none of them tell you the one thing that decides it: which path your packets take on the way back.

So the useful question isn't "which provider is fast". It's "how do I verify the path before I pay, and which path do I actually need". Everything below is built around that, using DMIT — a provider that sells China- and Asia-optimized routes out of Los Angeles, Hong Kong and Tokyo — as the concrete example, because its plan structure maps almost one-to-one onto the routing decisions you have to make anyway.

## Latency is a distance problem before it's a spec problem

Round-trip time is roughly four things stacked on top of each other: the physical propagation delay of light in fiber, the per-hop delay of every router in between, queuing delay when a link is congested, and the extra round trips from TCP and TLS handshakes. Only one of those is something you can tune on the server.

Propagation delay alone sets a hard floor. A server 8,000 km away costs you somewhere around 40 ms before a single router has done any work. Same metro, that floor is closer to 1 ms. This is why a small instance next to your users beats a big one on another continent for anything interactive — browsers wait on round trips, not on throughput.

Published latency ranges follow the same shape across every region:

| Path from user to server | Typical median RTT |
| --- | --- |
| Same metro | 1–5 ms |
| Same continent, different city | 15–50 ms |
| Different continent | 120–280 ms |

That spread is why "low latency VPS" is really a region-and-route question. A 2 vCPU box in Hong Kong beats a 4 vCPU box in Frankfurt for a user in Guangzhou, every time, regardless of what the spec sheet says.

There's a second variable most buyers ignore: jitter. A same-metro link with low jitter feels consistent. A cross-continent link with an average of 180 ms and occasional 400 ms spikes feels broken, even though the average looks acceptable. Look at the max and the loss percentage, not just the mean.

## The numbers that decide whether it stays low at 9pm

Peak hour is where cheap routes fall apart. International transit gets congested between roughly 20:00 and 23:00 China time, and that's when a "1Gbps port" turns into 20Mbps of real throughput and 5% packet loss. Four things separate a route that holds up from one that doesn't:

**Return path quality.** Outbound from China is often fine even on ordinary transit; the return direction is where packets get dumped onto congested public internet. A route advertised as "optimized" that only fixes the outbound direction is half a product. Test the direction your users actually consume content in.

**Route consistency.** Compare latency at 10:00 and at 21:00 from the same vantage point. If the evening figure is 2× the daytime figure, you've found the congestion point.

**Loss under load, not idle.** A traceroute at midnight proves nothing. Run MTR for a few hundred packets during peak, and look for a hop where loss jumps above 5% and stays there.

**Overcommitment.** Latency degrades when a host oversells. A provider that caps how many VMs sit on a node, and publishes it, is worth more than one that quietly packs them in.

## Region first, specs second

For users in mainland China and Asia-Pacific, the practical ranking looks like this, based on published tests:

- **Hong Kong** is the lowest-latency option, with one seven-day test showing an average of about 38 ms and 52 ms at evening peak. It's also the most expensive per gigabyte of RAM. Cross-strait fiber distance is short and several providers run direct premium routes.
- **Tokyo** lands in the middle — roughly 55 ms average in the same test, 89 ms at peak. Good for users in Japan, Korea and northern China.
- **Los Angeles** is the budget option and the most route-dependent. On ordinary transit you can expect around 180 ms to Shanghai. On a properly built CN2 GIA route, US West to mainland typically sits in the 130–160 ms band and stays there through peak. That 30–50 ms of difference is the entire product.

One caveat worth internalizing: a server labeled "Hong Kong" isn't automatically low-latency. Some budget "HK" plans actually route through the US and measure 200 ms+. That's not a hypothetical — it's a common enough complaint that "confirm the route with traceroute" is standard advice in every serious buying guide.

## Reading route names: what Pro, Eyeball and Tier 1 actually mean

DMIT organizes its catalog by network profile rather than by region, which is unusual and, once you know the names, useful. The profiles are not "tiers of quality" so much as different answers to "whose packets are you optimizing for".

| Profile | What the return path looks like | Who it's for |
| --- | --- | --- |
| **Premium (Pro)** | Tier 1 + DMIT's own backbone + China Telecom CN2 GIA | Users on all three mainland carriers, peak-hour stability required |
| **Eyeball (EB)** | Tier 1 + reasonable effort toward Chinese ISPs via CMIN2 or similar | Budget-first, but still want better-than-generic routing into China |
| **Tier 1 (T1)** | Plain international BGP, no China-specific optimization | International traffic, backups, CI/CD, relay nodes |
| **Premium Secure (sPro)** | CN2 GIA plus DDoS protection | Sites that get attacked and still need the good route |

A note on the letters you'll see in plan names: AS3, AN4 and AN5 are hardware generations, not network tiers. AS3 is AMD EPYC 7003, AN4 is EPYC 9004, AN5 is EPYC 9005. DMIT has been retiring older platforms — LAX AN4 sold out and disappeared from the catalog, HKG Pro moved to AN5 with DDR5, and LAX AS3 was reintroduced as a mid-tier platform that the company itself describes as still being optimized, with disk performance and SLA potentially below a mature platform's. Read the platform suffix as "how new the box is", and the Pro/EB/T1 tag as "how good the route is".

**Where the routes actually go.** On LAX Pro, the published routing is Telecom outbound via GIA (AS4809), Unicom outbound direct (AS4837), Mobile outbound via Hong Kong CMI (AS58453), and all three carriers returning over GIA. On LAX Eyeball, the return path is CMIN2 (AS58807) with Telcos routed to Unicom's AS9929 to ease peak-hour cross-network congestion. CN2 GIA is China Telecom's premium tier — separated from the public 163 backbone, higher scheduling priority, and roughly 3–5 hops instead of 10–15. CMIN2 is China Mobile's international premium path; AS9929 is Unicom's equivalent.

If you want a quick sanity check on any provider's claim, run a traceroute and look at the address ranges: `59.43.x.x` indicates CN2, while `202.97.x.x` is 163, the congested public backbone. Mixed results mean partial optimization, whatever the sales page says.

## Where DMIT fits, and what it costs

DMIT has been running since 2017–2018, registered in the US, with data centers in Los Angeles, Hong Kong and Tokyo. It supports Alipay, PayPal and credit cards, has Chinese-speaking support, and publishes per-region test IPs and Looking Glass nodes so you can measure before committing. The company also claims a self-built backbone with dedicated CN2 GIA capacity in the tens of gigabits and separately provisioned 9929 and CMIN2 capacity, in a market where reselling someone else's transit is the norm. Treat those capacity numbers as vendor figures rather than audited facts — but the routing configuration is verifiable with a traceroute, which is the part that matters.

Prices below are the published rates I could verify across current listings. Stock on this provider rotates constantly, and individual tiers ladder up in RAM, SSD, transfer and port speed — so treat this as the map, and confirm the exact configuration and availability on the live cart page before buying.

### Los Angeles — LAX.AS3.Pro (CN2 GIA return, EPYC 7003)

| Plan | Specs | Transfer / port | Price |
| --- | --- | --- | --- |
| LAX.AS3.Pro.TINY | 1 vCore / 2GB / 20GB SSD | 1TB @ 1Gbps | $10.90/mo |
| LAX.AS3.Pro.Pocket | 2 vCore / 2GB / 40GB SSD | ~1TB @ 1Gbps | $16.90/mo |
| LAX.AS3.Pro.STARTER | 2 vCore / 2GB / 80GB SSD | scales with tier | $34.90/mo |
| LAX.AS3.Pro.MINI | 4 vCore / 4GB / 80GB SSD | scales with tier | $62.90/mo |
| LAX.AS3.Pro.MICRO | 4 vCore / 4GB / 160GB SSD | scales with tier | $87.90/mo |
| LAX.AS3.Pro.MEDIUM | 6 vCore / 8GB / 160GB SSD | scales with tier | $199.90/mo |

Every plan in this family includes 1 IPv4, 1 IPv6 /64, standard DDoS protection and bidirectional transfer metering. Exceeding the monthly transfer doesn't cut you off — it drops you to an unmetered 4 Mbps.

👉 [Check LAX.AS3.Pro availability and current pricing](https://www.dmit.io/aff.php?aff=12046)

### Hong Kong — HKG Pro (CN2 GIA, lowest RTT to mainland)

Prices below come straight off DMIT's own Hong Kong Premium listing page:

| Plan | Specs | Transfer / port | Price |
| --- | --- | --- | --- |
| HKG.AS3.Pro.TINY | 1 vCore / 1GB / 20GB SSD | 500GB @ 1Gbps | $39.90/mo |
| HKG.AS3.Pro.STARTER | 1 vCore / 2GB / 40GB SSD | 1000GB @ 1Gbps | $79.90/mo |
| HKG.AS3.Pro.MINI | 2 vCore / 4GB / 60GB SSD | 1500GB @ 1Gbps | $126.90/mo |
| HKG.AS3.Pro.MICRO | 4 vCore / 4GB / 80GB SSD | 2000GB @ 1Gbps | $179.90/mo |
| HKG.AS3.Pro.MEDIUM | 4 vCore / 8GB / 160GB SSD | 2500GB @ 1Gbps | $239.90/mo |

Roughly four times the Los Angeles price for a comparable configuration, which is what a 100 ms latency advantage costs in this market. Hong Kong Pro's regular line has since been moved to the AN5 platform (EPYC 9005 with DDR5-5600), starting at $39.90/mo.

👉 [Compare the Hong Kong CN2 GIA plans](https://www.dmit.io/aff.php?aff=12046)

### Entry prices across the rest of the catalog

| Line | Region / platform | Entry configuration | Entry price |
| --- | --- | --- | --- |
| LAX.Pro (AN5, EPYC 9005) | Los Angeles, CN2 GIA | 1 vCore / 2GB / 20GB / 1TB @ 1Gbps | $12.98/mo |
| LAX.EB (AS3) | Los Angeles, CMIN2 + 9929 | 1 vCore / 2GB / 20GB / 1.5TB @ 2Gbps | $10.90/mo |
| LAX EB ladder (AS3) | Pocket → MEDIUM | 2→6 vCore, 2→8GB, 40→160GB | $16.90 → $199.90/mo |
| LAX.T1 | Los Angeles, international | 1 vCore / 1GB / 20GB | $6.90/mo, WEE $36.90/yr |
| HKG.T1 | Hong Kong, international | 1 vCore / 1GB / 20GB / 2TB | $6.90/mo, WEE $36.90/yr |
| TYO.Pro (AS3) | Tokyo, CN2 GIA | from 1 vCore / 1GB / 20GB | from $21.90/mo |
| TYO.T1 | Tokyo, international | 1 vCore / 1GB / 20GB / 1TB @ 4Gbps | WEE $36.90/yr |
| sPro (Premium Secure) | Los Angeles, CN2 GIA + DDoS | varies by configuration | see live cart |

Two things about the T1 line that are easy to miss: it's the only line where you can get into DMIT for under $40 a year, and its overage behavior is genuinely unmetered — once the monthly transfer is used up, the port throttles to a fixed speed (100 Mbps on the LAX WEE, 50 Mbps on the HKG and TYO WEE) rather than billing you or suspending the box. For a relay or backup node that's a reasonable deal. For a site serving mainland China users, it's the wrong product entirely.

👉 [See every plan family and current prices](https://www.dmit.io/aff.php?aff=12046)

## Discounts: where the recurring codes live

DMIT runs promo codes through the checkout's "Validate Code" field, and the codes that matter are the recurring ones — they apply on renewal, not just the first term. Codes circulating in late-2025 and 2026 community roundups include `LAX-EB-LAUNCH-NON-MONTHLY-RECURRING-20OFF` for LAX Eyeball on non-monthly billing, `202510_HKG_TYO_PRO_20OFF_RECURRING` for Hong Kong and Tokyo Pro on quarterly or longer terms, and `Lite-Annually-Recur-30OFF` for annual billing on the standard network line.

Campaign codes with dated prefixes (last year's Christmas sets, for example) are usually dead. Every code is tied to a specific region, line and billing cycle, and the wrong combination silently fails rather than erroring. Validate it before you pay, and confirm the line item actually changed.

Separately, annual billing discounts are sometimes applied automatically without a code at all. Historically that has meant 25% off the annual price on LAX.Pro.TINY and 10% off annual on the rest of the Pro ladder. Also worth checking: renewal pricing. In 2026 the minimum annual price on LAX AN5 Pro reportedly moved from $88 to $119.99, which is the kind of change that only shows up on your second invoice.

## Four things to check before you pay

**Stock and discontinuations.** This catalog churns. Tokyo Eyeball was retired on 15 June 2026, and Hong Kong EBv2 was retired on 10 August 2026, with holders moved off and offered credits plus discount codes. Plans that sold out for months include LAX AN4, TYO T1 and the Hong Kong T1 WEE. If a specific plan matters to you, verify it's orderable rather than assuming the page means it's in stock.

**IP replacement policy.** DMIT's documented policy is one free IP swap every 15 days, then $5 per change. Compared with the $8–10 many hosts charge, that's cheap — but it's a policy that can change, so read the current terms if IP hygiene is central to your use case.

**Traffic accounting.** Transfer is counted bidirectionally on these plans, meaning uploads count too. If you're running a relay or a file-heavy workload, model that before choosing a tier.

**Hardware generation.** AS3 boxes are cheaper and the platform is still being optimized. AN5 costs more and is newer. For a latency-sensitive workload the network matters far more than the CPU generation; for a compute-heavy one, the gap inverts. Don't pay for AN5 if your bottleneck is the route.

## A five-minute test you can run before buying

1. Pull the provider's test IP for your target region from their page.
2. `mtr -rwC 100 -c 100 <IP>` from where your users actually are — not from a datacenter looking glass, which bypasses the congestion you're worried about.
3. Repeat it at 21:00 local time. Compare average, max and loss against the daytime run.
4. `traceroute` and look for `59.43.x.x` (CN2) versus `202.97.x.x` (163). Mixed routes mean partial optimization.
5. `tcping <IP> 443` to see whether handshake overhead adds meaningful delay on top of raw ping.

If step 3 shows latency doubling or loss climbing past 5%, no plan table matters. That route won't hold at peak, and you'll be back here in a month.

## Who should not buy this

If your users are in Europe or North America and you have no China-facing traffic, the Premium and Eyeball premiums are wasted money — take the Tier 1 line at $36.90 a year, or a cheaper provider outside this category entirely. If your workload is trading-related and you need sub-2ms RTT to a broker's matching engine, you need co-location in Equinix NY4, LD4 or TY3, not an optimized transit VPS from Asia. And if the absolute lowest monthly number is your only criterion, this isn't that category of product; the low-latency segment costs more because scarce premium transit is what you're paying for.

## FAQ

**Is a Hong Kong VPS always the lowest-latency pick?**
Only if your users are in mainland China or nearby Asia-Pacific markets. For users in the US, a Los Angeles server is 30–60 ms away and Hong Kong is 150 ms+. Pick the region from where your users are, then optimize the route.

**How low can US West to mainland China actually go?**
On a properly built CN2 GIA return path, roughly 130–160 ms and stable through peak. On ordinary international transit, expect 180 ms and up with much worse evening behavior.

**Does lower ping always mean a faster site?**
No. Latency dominates interactive workloads and page loads with many round trips; throughput dominates large file transfers. A 130 ms route with 4 Mbps of overage throttling will feel slow for video, fast for a web dashboard.

**Can I test DMIT before buying?**
There are no trials, but the published test IPs and Looking Glass nodes per region let you measure real RTT and routing before you pay — which is the same evidence that should decide which of these plan families you pick.

👉 [Start with the plan family that matches your users' location](https://www.dmit.io/aff.php?aff=12046)
