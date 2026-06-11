# PLAN — openLabel

Claude owns this file (planning + review). Codex reads it and executes. Codex checks off tasks; Claude reviews the diff. Keep code edits single-threaded through Codex.

## Goal
Act on Joseph's 2026-06-10 design review of the `ship/` prototype: sharpen the hero, fix the oversized "Big Read", and start the product-vs-marketing split. Source: `references/transcripts/2026-06-10-joseph-wyatt.md`.

## Tasks — Bucket A: design fixes Joseph named (implement now, scope = `ship/` only)

- [x] **A1. Tame the "Big Read" card.** Joseph: "this big Read text is awkwardly big… not organized well." In `ship/index.html` the `.big-read-card` / `#big-read` (lines ~62-65) and its `styles.css` rule. Reduce font-size/visual weight so it reads as a lead paragraph, not a billboard; give it clear hierarchy against the Honest Proposition + Live Question cards beside it. Keep the three-card row, just rebalance.
- [x] **A2. Sharpen the hero headline + lede.** Joseph: headline has "too many words… what offering? not immediately clear." Current `#page-title` (line 28) = "What is this offering really asking you to believe?" Replace with a short, plain headline that states what OpenLabel *does* for the visitor in <8 words, and tighten the `.lede` (line 29) to one crisp sentence. Move the "what is this asking you to believe?" line down into the read/result framing where it belongs, not the masthead. Propose 2-3 headline options in the PR notes; ship the strongest.
- [x] **A3. Add a one-screen story strip above the tool.** Joseph: "this homepage needs to tell the story more, rather than just being the thing itself." Add a thin narrative band between the hero and the live `.lookup-card` (or directly under the headline): 3 beats — the problem (closed labels / overclaiming), what OpenLabel gives you (an open read of the claim), and the invitation to try it. Keep it short; the try-it widget stays prominent ("that's really good," per Joseph). Do NOT remove or bury the widget.

**Constraints for Bucket A:** Static HTML/CSS only, no new deps, no build step. Don't touch `app.js` data or the seeded reads. Preserve the soul section as-is — Joseph explicitly liked it. After editing, open `ship/index.html` and confirm it still renders + the seeded BrainTap read still populates.

## Bucket B: brand foundation — UNBLOCKED 2026-06-11, analyses DONE
Joseph's prompts arrived (JTBD + Business Foundation + ICP + BrandScript). Executed the full chain with real web research (55+ verbatim forum quotes). Deliverables in `brand/`:
- [x] **B1.** `brand/01-jtbd-analysis.md` — JTBD + Marketing Gold quote bank
- [x] **B2.** `brand/02-business-foundation.md` — offering/value/pricing/diagnosis/priorities
- [x] **B3.** `brand/03-ideal-client-profile.md` — segments, prioritization, lead ICP, where to find them
- [x] **B4.** `brand/04-brandscript.md` — StoryBrand script to drive all marketing copy
- [ ] **B5. Website revision (next).** Rebuild the `ship/` homepage as a StoryBrand narrative driven by `brand/04-brandscript.md`: hero = "Read the claim before it reads you," story strip = External→Internal→Philosophical problem, Guide (is-it-biased?) section, paste tool framed by the 3-step "Open Read" plan, Failure/Success band, identity-shift close + CTA + email magnet. **Awaiting Wyatt sign-off on lead ICP + tagline before rebuild** (see Decisions).

## Bucket C: directional / roadmap (capture, don't build)
- **Browser extension = the distribution channel.** Product functionality (the claim-read loop) eventually moves into an extension: click an icon on any site → OpenLabel's epistemic read of that site's claims. Homepage then becomes pure marketing.
- **Monetization options to research:** Trust Pilot / seal-of-approval models; licensing — annual fee + 6-month re-audit to keep an "OpenLabel approved — claims are real enough" mark. Requires name recognition first.
- **Second product line: open-label placebo tools.** Infrastructure + legibility around open-label placebos (work even when disclosed; underutilized in health/wellness). A distinct OpenLabel offering.

## Decisions / notes
- Bucket A is reversible, static-only, and matches Joseph's exact asks — safe for Codex to implement now.
- Soul panels (current vs. honest) are validated by Joseph; leave untouched.
- The "story vs. product" split is the throughline: this page → marketing; the tool → extension (later).

## Review (Claude) — 2026-06-11
Bucket A implemented by Codex; diff reviewed, matches plan, in scope.
- **A1** ✓ `.big-read` dropped from `clamp(25–38px)` serif billboard to `clamp(18–22px)` weight-600 lead paragraph; `.answer-card` min-height 220→190px. Three-card row preserved.
- **A2** ✓ Headline → "Read health claims clearly." (shipped; alts: "See what wellness claims earn." / "Know what the claim asks."). Lede tightened to one sentence. Old "what is this asking you to believe?" relocated to a `.result-framing` line in the read section.
- **A3** ✓ 3-beat `.story-strip` (closed labels overclaim → OpenLabel opens the claim → try the read) added under hero copy; try-it widget untouched and still prominent.
- Scope clean: only `ship/index.html` + `ship/styles.css` changed; `app.js`, seeded reads, and soul section untouched. Uses existing CSS vars; responsive collapse added.
- Caveat: Codex couldn't do a full visual render in its sandbox (verified via DOM harness only). Opened in browser locally for eyeball check.
- **Next:** Wyatt to eyeball; Bucket B unblocks when Joseph sends the JTBD + BrandScript prompts.
