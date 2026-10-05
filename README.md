# FitFindr

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, every
> command, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> python app.py listings --full -n 6      # read the data (Milestone 1)
> python app.py fields                    # what you can filter on
> python app.py ask 'vintage graphic tee under $30'
> ```
>
> The three tools start as stubs and are implemented during Unit 3.
> That's the starting position.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     This is your submission. Fill each section in as you finish the milestone
     it belongs to — don't leave it all to the end.

     Unit 3 asks for the first five sections. Unit 4 adds the five below them.
     Leave the unit 4 sections alone until then; they're here so you know
     what's coming.

     Everything is pasted as TEXT. No screenshots, no images, no video links.
     A typed block of output gets full credit; a picture of the same output
     gets none.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 3 — THE BUILD ═══════════════════════ -->

## What This Does

FitFindr helps a user find thrift listings that match a description, size, and budget. If a match is found, it selects the best result, suggests outfits using pieces from the user's wardrobe, and creates a short fit-card caption. If no listing matches, the agent stops early and tells the user what they could change about their search.

---

## Tool Inventory

### `search_listings(description, size, max_price)`

- **What it does:** Searches the thrift listings for items matching the user's description, optional size, and optional maximum price.
- **Inputs:** `description` (str), `size` (str or None), `max_price` (float or None).
- **Returns:** A list of matching listing dictionaries, best match first, with at most `SEARCH_RESULT_LIMIT` results.
- **When nothing matches:** Returns an empty list `[]`.

### `suggest_outfit(new_item, wardrobe)`

- **What it does:** Suggests one or two outfits using the selected thrift item and the user's wardrobe.
- **Inputs:** `new_item` (dict), `wardrobe` (dict containing an `items` list).
- **Returns:** A non-empty string containing outfit suggestions.
- **When the wardrobe is empty:** Returns general styling advice for the selected item instead of failing.

### `create_fit_card(outfit, new_item)`

- **What it does:** Creates a short social-style caption for the selected thrift item and outfit.
- **Inputs:** `outfit` (str), `new_item` (dict).
- **Returns:** A two-to-four sentence caption mentioning the item, its price, its platform, and the outfit vibe.
- **When the outfit is empty:** Returns a descriptive message instead of raising an error.

---

## Planning Loop

**Branch rule:** If `search_listings()` returns an empty list, put a message in the session telling the user what they can change and stop. Otherwise, take the first result, save it in the session, and continue to `suggest_outfit()`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** The query is parsed with regular expressions. The parser extracts an `under $X` price limit and a `size X` value, then removes those parts from the remaining text to use as the description.

**What moves through the session:** The parsed query is stored in `session["parsed"]`, search results go into `session["search_results"]`, the chosen listing goes into `session["selected_item"]`, the outfit goes into `session["outfit_suggestion"]`, and the final caption goes into `session["fit_card"]`.

---

## Sample Run

**One full query**

```text
$ python app.py ask 'vintage graphic tee under $30'

Found: Y2K Baby Tee — Butterfly Print — $18.0 on depop

Outfit: Here are two practical outfit suggestions using the Y2K Butterfly Baby Tee and pieces from your wardrobe:

Outfit 1: Y2K Streetwear Contrast
- Wardrobe pieces: Baggy straight-leg jeans (dark wash), vintage black denim jacket, chunky white sneakers, black crossbody bag.
- Why it works: The tight, cropped fit of the baby tee balances out the voluminous dark-wash baggy jeans. Layering the slightly cropped black denim jacket on top leans fully into the vintage Y2K aesthetic, while the chunky white sneakers tie the look together.

Outfit 2: Casual Crossover (Y2K Meets Earth Tones)
- Wardrobe pieces: Wide-leg khaki trousers, brown leather belt, black combat boots, black crossbody bag.
- Why it works: Pairing the playful, pink-and-purple butterfly print with minimal khaki trousers creates a fun contrast. Use the brown leather belt to define your waist with the high-rise trousers, and ground the pastel top with edgy black combat boots.

Fit card: Just scored the ultimate Y2K butterfly baby tee for only $18 on Depop and I am obsessed! I love styling it with baggy denim for that effortless streetwear contrast, or dressing it down with khaki trousers and combat boots for an edgy everyday vibe. Run, don't walk, because this piece is total vintage perfection!

0 model calls this session, 2 served from cache
```

**The three tools, tested one at a time**

```text
$ python -c "from tools import search_listings; print(search_listings('graphic tee', max_price=30))"

[{'id': 'lst_002', 'title': 'Y2K Baby Tee — Butterfly Print', 'description': 'Super cute early 2000s baby tee with butterfly graphic. Fitted crop length. Tag says medium but fits like a small.', 'category': 'tops', 'style_tags': ['y2k', 'vintage', 'graphic tee', 'cottagecore'], 'size': 'S/M', 'condition': 'excellent', 'price': 18.0, 'colors': ['white', 'pink', 'purple'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_006', 'title': 'Graphic Tee — 2003 Tour Bootleg Style', 'description': 'Vintage-style bootleg tee with faded graphic. Slightly boxy fit. 100% cotton, soft and worn-in.', 'category': 'tops', 'style_tags': ['graphic tee', 'vintage', 'grunge', 'streetwear', 'band tee'], 'size': 'L', 'condition': 'good', 'price': 24.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_017', 'title': 'Mesh Long-Sleeve Top — Black', 'description': 'Sheer black mesh long-sleeve. Great for layering under a graphic tee or over a bralette. Stretchy material, fits true to size.', 'category': 'tops', 'style_tags': ['y2k', 'grunge', 'goth', 'layering'], 'size': 'S/M', 'condition': 'excellent', 'price': 15.0, 'colors': ['black'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_033', 'title': 'Vintage Band Tee — Faded Grey', 'description': 'Faded grey band-style tee with distressed graphic. Crew neck. Fits boxy. Well-loved but no holes or major damage.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'band tee', 'graphic tee', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 19.0, 'colors': ['grey', 'charcoal'], 'brand': None, 'platform': 'depop'}, {'id': 'lst_011', 'title': 'Low-Rise Cargo Pants — Khaki', 'description': 'Y2K era low-rise cargo pants. Lots of pockets. Khaki color, slightly distressed at the hems. Great for layering with a long tee.', 'category': 'bottoms', 'style_tags': ['y2k', 'cargo', '2000s', 'streetwear'], 'size': 'W29', 'condition': 'fair', 'price': 27.0, 'colors': ['khaki', 'tan'], 'brand': None, 'platform': 'poshmark'}, {'id': 'lst_015', 'title': 'Vintage Graphic Hoodie — Faded Black', 'description': 'Faded black pullover hoodie with barely-visible vintage graphic on the chest. Cozy interior. Some pilling but adds to the worn-in look.', 'category': 'tops', 'style_tags': ['vintage', 'grunge', 'graphic', 'streetwear'], 'size': 'L', 'condition': 'fair', 'price': 26.0, 'colors': ['black', 'charcoal'], 'brand': None, 'platform': 'depop'}]
```

```text
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"

Here are two ways to style the vintage Levi's 501s using pieces already in your wardrobe:

Outfit 1: Effortless Casual Streetwear
- Top: White ribbed tank top
- Outerwear: Vintage black denim jacket
- Shoes: Chunky white sneakers
- Accessories: Black crossbody bag
- Why it works: The medium-wash 501s provide a great contrast against the black denim jacket, while the fitted white tank balances out the structured denim. Finish with chunky sneakers for an easy, everyday look.

Outfit 2: Cozy & Balanced Proportions
- Top: Oversized grey crewneck sweatshirt
- Accessories: Brown leather belt, Black crossbody bag
- Shoes: Black combat boots
- Why it works: 501s have a classic, straight-leg fit that pairs perfectly with an oversized top. Tucking the front of the grey crewneck into the jeans creates definition, and the combat boots add a grounded, grunge edge.
```

```text
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"

Scored the ultimate vintage Levi’s 501s and I’m honestly obsessed with this medium wash. Just dropped them on my Depop for $38 and they're ready for your new favorite everyday fit. Throw these on with some crisp white sneakers for that effortlessly cool streetwear vibe. Grab them before I change my mind and keep them!
```

---

## How I Used AI

**Moment 1**

- **What I asked for:** I asked ChatGPT to help me complete `search_listings()` using the starter TODOs and the listing fields.
- **What came back:** It suggested keyword scoring, size filtering, price filtering, and helper functions for tokenizing sizes.
- **What I changed:** I changed the stopword list after class discussion so words like `under` and `over` were not treated as meaningless search terms, and I fixed the size helper so `.upper()` was called correctly.

**Moment 2**

- **What I asked for:** I asked ChatGPT to help wire the three tools into the planning loop in `agent.py`.
- **What came back:** It suggested a regex-based query parser, session-based state flow, an empty-search branch, and a loop guarded by `trace.check_iterations()`.
- **What I changed:** I fixed an indentation error while integrating the code and verified that the selected item is read back from session state before calling `suggest_outfit()`.

<!-- ═══════════════════════ UNIT 4 — THE TEST ═══════════════════════

     Don't fill these in during unit 3.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Run Log — Before

<!-- Five criteria, five tries each, in this exact format.

     Five, because your criteria are written out of five. Mark each try PASS
     or FAIL, count the passes, and read that count against your target — a
     row targeting 4 of 5 with three PASS cells is MISSED (3/5).

     `python run_eval.py --label before` runs everything and writes the table
     into results/. Paste it here and fill in the verdicts. -->

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Real output from one try**, pasted as text, naming the file and function
that produced it:

```

```

---

## Verdicts and Diagnoses

<!-- MET or MISSED per criterion against LAST UNIT's target, plus a sentence on
     how you decided.

     Then, for every miss: which of the four places it happened — a tool, the
     loop's branch, the session, or the model's output — AND the mechanism.

     Not a diagnosis:  "The fit card was bad."
     A diagnosis:      "The fit card criterion missed on 2 of 5 items. Both had
                        an empty brand field. My prompt puts the brand in the
                        first sentence, so the card opened with a blank and read
                        like a fragment. The tool worked; the prompt assumed a
                        field that isn't always there."

     Look for a pattern. Three misses on the same tool is one problem, not
     three. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**



---

## Loop Trace

<!-- One full run, printed step by step, with the MCP call visible in it.

     `python app.py ask '...' --trace` once you've added the trace.step()
     calls in Milestone 2.

     Worth pasting BOTH the happy path and the empty-search path. The empty
     one should be visibly shorter, because it stops. If your two traces are
     the same length, your branch isn't working — and this is the fastest way
     anyone will ever find that out. -->

**Happy path**

```

```

**Empty search**

```

```

**On the MCP move:** <!-- what changed in your code, and whether anything
behaved differently afterwards. If the rewire didn't work, say exactly where it
broke — the error text and the last thing that worked. That earns the point in
full. -->



---

## The Improvement

<!-- What you changed, why your diagnosis pointed at it, and the after-run in
     the same table format. One change, measured properly.

     `python run_eval.py --label after` -->

**What I changed:**

**Which failure it was meant to fix:**

### Run Log — After

| Criterion | Target | Try 1 | Try 2 | Try 3 | Try 4 | Try 5 | Verdict |
|---|---|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |  |  |
| 2.  |  |  |  |  |  |  |  |
| 3.  |  |  |  |  |  |  |  |
| 4.  |  |  |  |  |  |  |  |
| 5.  |  |  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it made things worse, say that. Honestly reported, that earns full
     credit and is more interesting than one that worked. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped where
     you did. "I ran out of time" is fine if it's true. Pretending nothing is
     left is not. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 3

       [ ] criteria.md has five numbered criteria, each with a target
       [ ] Each criterion has a reason underneath it
       [ ] All five unit 3 sections above have real content
       [ ] Tool Inventory: all three tools, inputs WITH TYPES, a specific
           return value, and the empty case
       [ ] Planning Loop names the branch rule and agent.py::run_agent
       [ ] Sample Run: one full query plus the three per-tool tests, as text
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN, you submit the same one
           next unit

     SUBMISSION CHECKLIST — unit 4

       [ ] mcp_server.py exists with one tool registered
           (or a written record of exactly where the rewire broke)
       [ ] Run Log — Before, five criteria, five tries each
       [ ] Real output pasted underneath, naming file and function
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, naming a place AND a mechanism
       [ ] Loop Trace, with the MCP call visible in it
       [ ] All three failure modes triggered and handled
       [ ] One improvement, with Run Log — After in the same format
       [ ] What's Still Broken
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository. Your commit history is what
     shows your criteria existed before your results did.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
