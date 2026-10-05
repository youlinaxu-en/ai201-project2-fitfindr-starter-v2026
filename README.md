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
> All three tools are stubs, so that last command will do nothing useful yet.
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

FitFindr turns a clothing request into a thrifted outfit recommendation. It extracts an optional size and price ceiling from the query, searches and ranks the local listings, then uses the selected item and the user's wardrobe to generate outfit advice and a short fit-card caption. If nothing matches, it stops and tells the user which search constraints to change.

---

## Tool Inventory

<!-- Four lines per tool. This is worth 2 points and it's the single most
     common place students lose them.

     "Returns a list" earns NOTHING. The description has to say what is IN
     the list.

     The empty case isn't optional either — it's the thing your loop branches
     on, and if you don't decide it here you'll discover it as a crash in
     Milestone 5. -->

### `search_listings`

- **What it does:** Searches the thrift listings for the best matches to a shopping description, with optional size and price filters.
- **Inputs:** `description` (str), `size` (str | None; case-insensitive whole-label/token match, so `M` matches `S/M` but not `XL` or `US 9`), `max_price` (float | None)
- **Returns:** A list of matching listing dicts sorted by score, each containing fields such as `id`, `title`, `price`, `size`, `platform`, `category`, and `description`.
- **When it has nothing:** An empty list.

### `suggest_outfit`

- **What it does:** Takes a selected thrift find and a wardrobe dictionary and returns an outfit suggestion that fits the item and the user’s clothes.
- **Inputs:** `new_item` (dict), `wardrobe` (dict)
- **Returns:** A non-empty string with one or two outfit suggestions, naming items from the wardrobe when available and otherwise giving general styling advice.
- **When it has nothing:** An empty string.

### `create_fit_card`

- **What it does:** Writes a short, social-style caption that describes the find and how it fits into an outfit.
- **Inputs:** `outfit` (str), `new_item` (dict)
- **Returns:** A two-to-four sentence caption string that mentions the item, its price, and the platform in a natural, post-like tone.
- **When it has nothing:** A descriptive fallback string instead of raising an exception.

---

## Planning Loop

<!-- Your branch rule, stated as a rule — the condition AND both paths — plus
     the file and function that holds it.

     Like this:
       "If search_listings returns an empty list, put a message in the session
        and stop. Otherwise take the first result and go to suggest_outfit."
        — agent.py::run_agent

     The grader checks your code against what you claim here, so the file and
     function have to be real. -->

**Branch rule:** In `agent.py::run_agent`, if `search_listings` returns an empty list, set `session["error"]` to a message suggesting changes to the keywords, size, or price limit, then stop without calling `suggest_outfit`. Otherwise, select the first result and continue to `suggest_outfit` and `create_fit_card`.

**Where it lives:** `agent.py::run_agent`

**How the query is parsed:** Regular expressions extract a size after the word `size` and a price after `under`, `below`, `less than`, `at most`, or `max`; the remaining text is searched as the item description.

**What moves through the session:** `parsed` goes to `search_results`; the first result becomes `selected_item`, which is passed with `wardrobe` to produce `outfit_suggestion`; that suggestion and the same `selected_item` produce `fit_card`.

---

## Sample Run

<!-- Two things go here.

     1. One FULL query and its output, pasted as text.
     2. Your three per-tool terminal tests — the command and what it printed. -->

**One full query**

```
$ python app.py ask '...'
  app.py ask "looking for a vintage graphic tee under $30 size M"er-v2026> 

  Found:    Y2K Baby Tee — Butterfly Print — $18.0 on depop

  Outfit:   **Outfit 1: Y2K Streetwear**
* **Thrifted Item:** Y2K Baby Tee — Butterfly Print
* **Wardrobe Pieces:** Baggy straight-leg jeans (dark wash), chunky white sneakers, and black crossbody bag.
* **Why it works:** The fitted, cropped cut of the baby tee balances the high-waisted, baggy fit of the dark wash jeans for an authentic early 2000s silhouette. Finish with the chunky white sneakers and black crossbody bag.

**Outfit 2: Casual Contrast**
* **Thrifted Item:** Y2K Baby Tee — Butterfly Print
* **Wardrobe Pieces:** Wide-leg khaki trousers, brown leather belt, and chunkywhite sneakers.
* **Why it works:** Pairing the playful pink, purple, and white butterfly graphic tee with neutral wide-leg khaki trousers creates an easy, balanced look. Cinch the trousers with the brown leather belt and complete the outfit with the chunky white sneakers.

  Fit card: Lean into early 2000s nostalgia with this white, pink, and purple Y2K baby tee featuring a sweet butterfly print. Grab this fitted crop top for $18.0 on Depop and style it with baggy dark wash jeans and chunky white sneakersfor an authentic streetwear silhouette.

1 model calls this session, 1 served from cache, 411 prompt + 61 output tokens
```

**The three tools, tested one at a time**

```
$ python -c "from tools import search_listings; print([(item['id'], item['title'], item['price']) for item in search_listings('graphic tee', max_price=30)])"
[('lst_002', 'Y2K Baby Tee — Butterfly Print', 18.0), ('lst_006', 'Graphic Tee — 2003 Tour Bootleg Style', 24.0), ('lst_017', 'Mesh Long-Sleeve Top — Black', 15.0), ('lst_033', 'Vintage Band Tee — Faded Grey', 19.0), ('lst_011', 'Low-Rise Cargo Pants — Khaki', 27.0), ('lst_015', 'Vintage Graphic Hoodie — Faded Black', 26.0)]

```

```
$ python -c "from tools import suggest_outfit; from utils.data_loader import get_example_wardrobe, load_listings; print(suggest_outfit(load_listings()[0], get_example_wardrobe()))"
Here is a wearable outfit using the Levi's 501 jeans and your existing wardrobe pieces:

**Outfit: Casual Streetwear**
* **Thrifted item:** Vintage Levi's 501 Jeans — Medium Wash
* **Wardrobe pieces:** White ribbed tank top (`w_003`), Vintage black denim jacket (`w_006`), Chunky white sneakers (`w_007`), Brown leather belt (`w_009`), and Black crossbody bag (`w_010`).

**Why it works:**
Tuck the white ribbed tank top into the medium wash 501s, secure it with the brown leather belt, and layer the slightly cropped black denim jacket on top. Finish the look with the chunky white sneakers and the black crossbody bag for a classic, effortless streetwear combination.

```

```
$ python -c "from tools import create_fit_card; from utils.data_loader import load_listings; print(create_fit_card('jeans and white sneakers', load_listings()[0]))"
Nothing beats the broken-in feel of these vintage Levi's 501 jeans in a classic medium wash. I love how the natural fading at the knees gives them that ultimate worn-in streetwear edge. Grab these blue denim bottoms for $38.0 on depop and pair them with crisp white sneakers for an effortless off-duty look.

```

---

## How I Used AI

<!-- Two specific moments. What you asked, what came back, what you changed.

     "I used Claude to help me code" is not enough.

     "I gave Claude my search_listings spec. It returned None on no match
     instead of an empty list, so I changed it" is the level we want. -->

**Moment 1**

- *What I asked for:* I asked Copilot to implement `search_listings` with keyword ranking, optional size and price filters, and an empty-list result when nothing matches.
- *What came back:* The first search implementation counted every query word. A focused no-match check still returned listings because common words such as `a` and `not` appeared in listing descriptions.
- *What I changed:* I added stop-word filtering to the query keywords, then reran checks for no matches, size boundaries, and the inclusive price ceiling.

**Moment 2**

- *What I asked for:* I asked Copilot to build `agent.py::run_agent` from the README branch rule and keep each result in the session for the next step.
- *What came back:* The first loop saved the parsed query in the session but passed the local `parsed` variable to `search_listings`, so that handoff did not read the value back from session state.
- *What I changed:* I changed the search call to use `session["parsed"]`, verified that the selected item passed to `suggest_outfit` was the same object stored in the session, and tested both the matching and no-match branches.

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
