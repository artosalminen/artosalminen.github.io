---
title: "How I Built a Vehicle Cost Comparison Tool That Doesn't Ignore Depreciation"
date: 2026-09-19
series: ["Coding", "Tools"]
weight: 1
tags: ["Coding", "JavaScript", "Finance"]
author: "Arto Salminen"
---

I was replacing a 2014 petrol MPV with a used EV. Along the way I collected insurance quotes, checked the vehicle tax bracket, asked for a battery health certificate, got a trade-in offer, and compared a cash price against a dealer's financing plan. Somewhere in the middle of all that I wanted one simple thing: a single number telling me what switching would actually cost per year, compared to just keeping the car I already had.

I couldn't find a calculator that did that honestly, so I built one.

## The problem with the calculators that exist

Search for a car cost calculator and you'll mostly find two kinds. The first compares fuel costs between two vehicles and stops there — which makes EVs look like an obvious win every time, because it leaves out the line that usually decides the outcome: depreciation. The second kind is a lead-generation funnel dressed up as a calculator, built to get you talking to a dealer or an insurer, not to give you an honest number.

Neither compares against the option that's actually on the table: keeping the car you have. That car is depreciating too, just slower, and it has running costs of its own. If a candidate vehicle doesn't beat that baseline, it isn't worth switching for, no matter how good the fuel savings look in isolation.

## What it does

[Vehicle cost comparison](https://slmnn.fi/vehicle-tco/) ([source](https://github.com/artosalminen/vehicle-tco)) takes the vehicle you own as the baseline and compares any number of candidates against it. Every candidate gets reduced to one hero number: **the annual cost of switching, versus doing nothing.** Negative means the candidate is cheaper than keeping what you have; positive means it costs you money to switch, even if it feels nicer to drive.

Under that headline number, the full breakdown is there for anyone who wants to argue with it:

- **Petrol, diesel, electric and plug-in hybrid** on either side of the comparison, with separate cold-weather and normal consumption figures. This turned out to matter more than I expected — a combustion engine on short winter trips can burn 50% more fuel than its rated figure, and an EV loses a lot of range to cabin heating on the same kind of trip. Averaging the two into one annual figure hides a real difference between vehicles.
- **Cash or financed purchase**, with actual month-by-month loan amortisation rather than a flat interest guess — including balloon loans, which is what most EV financing offers looked like when I was shopping. The tool warns you if the balloon payment is bigger than the car's estimated value at that point, because that's the scenario where you'd owe more than the car is worth.
- **Trade-in value per candidate**, not a single shared figure. Different dealers quoted me different numbers for the same car, and a shared trade-in value would have silently corrupted any comparison between them.
- **A residual-value stress test** that shifts every candidate's assumed end-of-term value at once, so you can see what a slump in used prices would do to the numbers without re-typing them one by one.
- **An optional baseline**, for anyone who owns nothing and just wants to compare candidates against each other. The tool is upfront that this is a weaker framing than a real delta — an absolute total isn't the same claim as "cheaper than what you have."

Every input is a plain, editable field — nothing is hidden, and nothing is estimated behind the scenes. There's no valuation data baked in; you type in a purchase price and an expected end value, the same numbers you'd take to a spreadsheet.

It's also a single static HTML file with no backend, no account, and no tracking. Your scenario lives in your browser's local storage, and if you want to share it, a **Copy link** button compresses the whole thing into the URL fragment — the part after `#`, which browsers never send to a server — so a five-vehicle comparison turns into a link you can paste into a message.

## What it deliberately leaves out

The spec I wrote before building this has a whole section of non-goals, and I think the reasoning is worth stating plainly rather than leaving people to assume an oversight:

- **No valuation lookups.** The tool doesn't know what your car, or the one you're considering, is actually worth. Getting that wrong silently would be worse than asking you to type in a number you got from a real quote.
- **No tax or insurance API.** Both are quoted per person and per vehicle in ways a lookup can't capture accurately. You enter the number your insurer or tax authority actually gave you.
- **The baseline assumes your current car survives the whole comparison period.** If it doesn't — if it fails in year three — you'd buy something sooner than planned, and the model doesn't try to predict that. It's a known bias in favour of keeping what you have, and I'd rather say so than pretend the model accounts for it.
- **Metric units only, for now.** Distance in km, consumption per 100 km or 100 kWh. An imperial toggle is on the list but didn't make the first cut.
- **No CSV export, print stylesheet, or country presets yet**, and no way to import a candidate straight from a classifieds listing — that last one turns out to need either a proxy, a browser extension, or you pasting in the page content, since a static site can't fetch another site's page directly. All of that is written up in the spec as later phases, not forgotten.
- **One external request.** The page loads Google Fonts. Everything else — the calculation, your data, the shared link — never leaves your device.

None of this is because the missing pieces are hard; it's because a plausible-but-wrong number is worse than an honest gap. The tool would rather ask you for a real figure than guess one.

## Trying it

It's live at [slmnn.fi/vehicle-tco](https://slmnn.fi/vehicle-tco/), the source and the full design spec are on [GitHub](https://github.com/artosalminen/vehicle-tco), and it's MIT licensed if you want to fork it for your own country's tax and insurance quirks. If you end up adding a preset for fuel or electricity prices where you live, that's exactly the kind of contribution the project is set up for.
