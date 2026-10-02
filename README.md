# Amazon Beauty Listing Skill - showcase

An AI agent skill that builds a complete, compliance-checked Amazon listing for a skincare product from three inputs: the formula (INCI list), the brand brief and a Helium 10 keyword export.

I built the first version as five Dify workflows for a live skincare range, then rebuilt it as a single [Agent Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) with Python checks. This repository shows what it does, how it is designed and what it produces. The skill itself and its scripts are kept private; contact me if you would like a walkthrough.

## What it produces

| Stage | Output | Example |
|---|---|---|
| 1. Product Bible | Product summary, key and supporting ingredients, target customer, verified claims; Seller Central attributes | [product_bible.md](examples/output/product_bible.md), [attributes](examples/output/seller_central_attributes.md) |
| 2. Keywords and SEO copy | Cleaned and ranked keywords; title, item highlights, five bullets, backend search terms | [listing.md](examples/output/listing.md) |
| 3. Gallery brief | Main image plus seven secondary images, with visual direction and ready-to-typeset copy | [gallery_brief.md](examples/output/gallery_brief.md) |
| 4. A+ brief | Seven modules with creative direction, copy and alt text | [aplus_brief.md](examples/output/aplus_brief.md) |
| 5. Shopper Q&A | Answers to the questions shoppers ask the Amazon assistant | [qa.md](examples/output/qa.md) |

The example inputs are in [examples/input](examples/input). All brand, product and keyword data in this repository is fictional.

## How it works

```
INCI sheet + brand brief + packaging copy
        |
   1. Product Bible  ------------------------------+
        |                                          |
   2. Keywords + SEO copy   3. Gallery brief   4. A+ brief   5. Shopper Q&A
        |                          |               |              |
        +----------- compliance check on every output ------------+
```

Four design decisions do most of the work:

1. **The formula is fact, the copy is craft.** Which ingredients are in the bottle, character counts and claim rules are handled by code. The model writes; it does not decide what the product contains or whether a title fits.
2. **One Product Bible.** It is written once, reviewed by a person and reused by every stage, so the title, images and A+ never contradict each other.
3. **No claim without a source.** "Cruelty free", "dermatologist tested" and every "free from" statement appear only when listed in the brand's claims register.
4. **Rules are data.** Limits and claim rules live in configuration, so an Amazon policy change (such as the 75-character title limit of July 2026) is a one-line edit.

## What the keyword step does

A reverse-ASIN export describes what competitors rank for, so most of it is not usable. On a real 179-row export for a vitamin C serum:

| Finding | Handling |
|---|---|
| 118 rows were searches for other brands | Removed. Brands are detected from the data (click-share concentration), not only from a fixed list |
| Misspellings, abbreviations and Spanish variants | Routed to the backend search terms, not the visible copy |
| Keywords naming an ingredient the formula does not contain | Removed, using the INCI list as the reference |
| Other product types and use areas ("face oil", "body serum") | Removed |
| Word-order and punctuation variants | Merged, volumes combined |
| Result | 12 usable keywords, led by two head terms carrying over 95% of the volume |

## What the compliance check verifies

Title, item highlights, bullet and backend limits (backend in bytes); title special characters and word repetition; emoji; drug-style and unsubstantiated claims by market (US and EU); competitor brand names; ingredients that are not in the product's INCI list; backend words wasted on terms already indexed.

## From workflows to a skill

[docs/from-dify-to-skill.md](docs/from-dify-to-skill.md) explains what was kept from the Dify version, what changed and why, and what testing on real data broke.

## About

Built by YM Pan. Brand and new-product-development background in beauty and consumer goods across the US, French and Chinese markets, with a focus on Amazon.
