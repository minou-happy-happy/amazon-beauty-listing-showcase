# From five Dify workflows to one skill

The first version of this pipeline was five Dify workflows: Product Bible, SEO, Gallery Brief, A+ Brief and Q&A. They worked and were used on a live skincare range. Rebuilding them as a skill was a chance to fix structural problems that are hard to see from inside a node canvas.

## What was kept

- **INCI as the only source of truth.** The Product Bible validated every ingredient the model returned against the formula sheet and rebuilt the supporting-ingredient list in code. This anti-hallucination pattern is the core of the skill.
- **A fixed INCI-to-consumer-name map**, so the model never invents a "complex" name.
- **Briefs written for the people who use them**: purpose, visual direction and ready-to-typeset copy with word counts per image.
- **Separate writer and reviewer steps** for SEO copy.

## What changed

| In the workflows | Problem | In the skill |
|---|---|---|
| Gallery, A+ and Q&A each called the Product Bible workflow over HTTP at temperature 0.7 | Three different Bibles for one product; three times the cost; a credential stored in each workflow file | The Bible is written once to a file, reviewed, and read by every stage |
| Keyword cleaning and scoring ran in code, but the writer prompt received the first 3,000 characters of the raw export | The copy was not written from the cleaned keywords | The writer works only from the cleaned keyword list |
| Keyword relevance = word overlap with the product name plus a generic term list | "retinol serum" scored well for a hyaluronic serum | Keywords naming ingredients outside the formula, or another product type, are excluded with a reason |
| Character limits were checked by a second LLM call | Models cannot count reliably | A script counts characters and bytes |
| Each prompt carried its own banned-word list | Lists disagreed: "anti-aging" banned in one, allowed in another; "dermatologist tested" banned in one, offered as a badge in another | One rules file enforced on every output |
| The reviewer replaced "clinically proven" with "dermatologist-tested" | Swapped a wording issue for an unverified claim | Proof-dependent claims pass only if listed in a claims register |
| Badges chosen by the model as "most relevant" | Unverified claims on images | Badges come only from the claims register |
| A+ module comparing the product with "competitors" | Competitor comparison is not allowed in A+ | "Why this formula" (positives only) and an own-range comparison chart |
| Before-and-after gallery image by default | A results claim without evidence | Texture image by default; results imagery only with proof |
| Fuzzy product-name matching on the first two words | "Vitamin C Serum" could match "Vitamin C Cream" | Exact match, otherwise stop and list what is available |
| "Serum", routine products and brand colours hard-coded in prompts | Worked for one range only | Read from the brand brief and the Bible |
| Title limit only | Item Highlights field (125 characters, searchable) unused | Written and checked alongside the title |
| Q&A length check computed but never shown; prompt limit and code limit differed | Silent failures | One limit, enforced, reported |

## What testing on real data changed

The first version passed on synthetic data and then met a real Cerebro export and a real INCI workbook. Five things broke and were fixed:

- A fixed brand blocklist caught 27 of 118 brand searches. Brand detection now uses click-share concentration in the data itself.
- The head term (273,000 searches) was discarded because a punctuation variant appeared earlier in the file. Variants are now merged and the largest volume kept.
- US-style ingredient names ("Vitamin E (Tocopheryl Acetate)") were not recognised. Lookup now handles both label styles.
- A concentration range of 1-5% had been converted to a date by the spreadsheet. The script now restores it and warns.
- Misspellings and Spanish terms were ranked as hero keywords. They now go to the backend list.

## What a skill does better than a workflow here, and what it does not

A workflow is the right tool when the same job runs unattended on a schedule or for a team that should not touch prompts. A skill is better when a person is in the loop: it can ask for a missing input, show the Bible for approval, explain why a keyword was dropped, and rerun the checker until it passes. For listing creation, where a human is accountable for every claim, that is the better fit.
