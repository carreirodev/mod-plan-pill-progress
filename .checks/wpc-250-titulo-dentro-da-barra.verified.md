# Plan bar title inside and scattered dots (WPC-250) Verification

**Verdict**: PASS - 28 of 29 checks proven; C17 is a manual check by the author and is still outstanding
**Profile**: light (no `AGENTS.md`)
**Diff range**: b0f064f..f8e8849 (HEAD of carreiro/wpc-250-plan-progress-title-inside-the-plan-bar-and-scattered-dots)
**Round**: 2 - scoped (fix diff 19eabb8..f8e8849, plus every verdict that was not PASS; proofs re-run in full at f8e8849)
**Verifier**: independent sub-agent (author != verifier)

What f8e8849 changed:
- `register.tsx`, `tests/engine.mjs` and `tests/stub.mjs` are unchanged since 19eabb8.
- In `regress.mjs`, three cases gained assertions, and none lost one:
  - `row_is_mark_bar_percent_close` now runs its desktop leg unchanged and repeats it on `vscode`.
  - `pill_name_needs_160_px_of_progress` keeps its desktop legs and adds VS Code at 70 and 40 columns.
  - `done_bar_streams_at_half_speed` now asserts `Math.abs(running - 11.65) < 0.01` where it had `running > 0`.
- The checklist changed in three places: the wording of C1, C11 and C15, and four new entries in the list of changed cases.

Gaps 2 and 3 are closed. Gap 5 is mostly closed: one item is still undeclared. Gap 1 is now recorded in C15 and still waits on the author's decision.

## Steps not run under `light` (carried from 19eabb8)

- Step 1 (checks against the binding sources, screen by screen): not run. The checklist says the design is binding for the interface and that `ui` is the profile for that. So the variant-B composition (`.design/plan-bar-title-inside-mockups.jpg`) was not compared element by element.
- `Coverage` join: not run. The checklist has no `Coverage` section either.
- `Test policy` verdicts: not run. The checklist has no `## Test policy` section either.
- Step 4 (fault injection): not run, in either round. No mutant was tried, including on the three surfaces f8e8849 added.

## Checks (verified at f8e8849)

Line numbers are in `plugins/plan-progress/tests/regress.mjs` at f8e8849 unless noted. They are all refreshed, because f8e8849 added lines at 526, 5763-5792 and 5960-5970. Each case returns `[observed, ok]`, and the cited expression is its `ok`.

Some confirmations were run against the 1.3.0 build and are carried from 19eabb8: C26's colours, C27's hash, and C29's stored text. Neither `register.tsx` nor those literals changed since.

| Check | Claim | Proof run | Evidence | Result |
|---|---|---|---|---|
| C1 | desktop and VS Code row is mark, bar, percent, fold cell, close; no title Text | `row_is_mark_bar_percent_close` ok | `:5773` - `for (const surface of ['desktop', 'vscode'])`, rows read through `F.band(surface)` (`:5767`; `engine.mjs:414` passes the surface on); `:5782-5786` - `same(plain[0], ['mark','bar','percent','close']) && same(folded.find(r => r.key === 'bar-a'), ['mark','bar','percent','fold cell','close']) && same(folded.find(r => r.key === 'bar-b'), ['mark','bar','percent','fold','close']) && !titled` (`titled` at `:5781`). Observed the same row on both surfaces | PASS |
| C2 | title at x=12; shared zone = longest title + 24 (up to +15%); at most floor(0.3W) | `title_zone_fits_the_longest_title` ok | `:5797` - `zone >= MEASURED[title] + 24 && zone <= MEASURED[title] * 1.15 + 24`; `:5814` - `inside && fits(alone,'Docs') && fillOf(a) === fillOf(b) && fillOf(b) === longAlone && fits(longAlone,'Refactor payments') && fillOf(capped.source) === cap && titleIn(capped.source)?.[2].endsWith('…')` | PASS (see gap 4) |
| C3 | cut title ends in "…", with a hover area over the zone and a tip holding the whole title; a fitting title has neither | `cut_title_shows_whole_on_hover` ok | `:5831` - `!!tip && over.includes(`.${area[1]}:hover~.${tip[1]}`) && over.indexOf(area[0]) < over.indexOf(`class="tp ${tip[1]}"`) && Number(area[2]) === zone`; `:5833` - `!/<rect class="h[a-z]/.test(docs) && !docs.includes('>Docs<')` | PASS |
| C4 | 41 columns (188 px): no zone, fill and marks over the whole track; 42 columns (196 px): zone; alt starts with the title | `narrow_bar_has_no_title_zone` ok | `:5850` - `W === 188 && !hasTitle && Math.abs(fillOf(track.source) - W / 3) <= 0.1 && Math.abs(marks[0] - W / 3) <= 0.1 && Math.abs(marks[1] - (2 * W) / 3) <= 0.1`; `:5851-5852` - `W === 196 && titleIn(...)?.[2] === 'Task' && ...; track.alt.startsWith('Task: ')` | PASS |
| C5 | head at zone + f(W - zone) ±0.1; pill never before the zone's end; marks and areas between zone and W | `progress_runs_from_the_title_zone_to_the_end` ok | `:5874` - `Math.abs(fx - (zone + f * (W - zone))) <= 0.1`; `:5875` - `pill.left >= zone - 0.05 && marks.every(x => x >= zone && x <= W) && hits.every(([a, b]) => a >= zone - 0.05 && b <= W + 0.05)`, at f = .25/.5/.75 | PASS |
| C6 | fresh plan fills exactly the zone, on the gradient's first stop; pill's left edge at the zone's end | `fresh_plan_fills_the_title_zone` ok | `:5894` - `Math.abs(fx - zone) <= 0.2 && x1 >= fx - 0.1 && first === '.05' && Math.abs(pill.left - zone) <= 0.2` | PASS |
| C7 | pill glides old to new with its left edge never before the zone; fill glides from inside the progress | `pill_glides_inside_the_progress` ok | `:5908` - `Number(to) > Number(from) && Number(from) - kw / 2 >= zone - 0.05 && Number(to) - kw / 2 >= zone - 0.05 && Number(fillFrom) >= zone - 0.05 && Number(fillTo) > Number(fillFrom)` | PASS |
| C8 | a bar coming or going that changes the zone redraws the others at the new zone with no `<animate>` | `title_zone_change_redraws_without_a_glide` ok | `:5930` - `!glides(during) && !glides(after) && Math.abs(fillOf(during) - third(bigZone)) <= 0.1 && Math.abs(fillOf(after) - fillOf(before)) <= 0.1 && bigZone > smallZone + 20` | PASS |
| C9 | done: green, title in its zone, pill at the right end with the total time | `finished_bar_keeps_its_title` ok | `:5941` - `green && titleIn(src)?.[2] === 'Task' && Math.abs(pill.right - 820) <= 0.1 && src.includes('>2m 5s<')`; green is `rx="11" fill="#18883A"` at `:5940`, which is the pill's face (`register.tsx:976/994`), not the fill | PASS (green read off the pill only) |
| C10 | contrast of at least 4.5:1 against the track over the page, the fill, and the dots at their brightest, in 4 states x 2 themes | `title_reads_in_every_state_and_theme` ok | `:6092` - `Math.min(contrast(ink, track), contrast(ink, ground), contrast(ink, top))`; `:6094` - `worst >= 4.5` | PASS |
| C11 | at least 160 px of progress gives the named capsule; less gives the round dot; desktop at 46/45 columns, VS Code at 70/40 | `pill_name_needs_160_px_of_progress` ok | `:5958` - `run >= 160 ? named && !dot : dot && !named` (desktop); `:5969` - `cols === 70 ? run >= 160 && named && !dot : run < 160 && dot && !named` (VS Code, through `atCols(F, cols, 'vscode')` at `:5964`); `:5971` - `/: 160 px/.test(seen[0]) && /: 15\d px/.test(seen[1])`. Observed VS Code 70: 221 px, named; VS Code 40: 120 px (no zone), round dot | PASS (see gap 6) |
| C12 | strip hides the tool word at 356 px and shows it at 364 px | `strip_tool_word_keeps_its_360_px` ok | `:5984` - `seen[0].join() === '356,356,false' && seen[1].join() === '364,364,true'` | PASS |
| C13 | strips in the bar's column, right under it, as wide as it | `agent_strips_line_up_under_the_bar` ok | `:6001` - `kids.indexOf(column) === 1 && kids[0]?.type === 'Text' && trackBox === 0 && strips.length === 2 && strips.every(s => s.props.width === width)` | PASS |
| C14 | mark hover areas at zone + k/total x (W - zone), each with its tip; clock on the pill | `hover_marks_and_clock_follow_the_progress` ok | `:6012` - `areas.length === 3 && areas.every(([k, mid]) => Math.abs(mid - (zone + (k / 4) * (W - zone))) <= 0.1 && v.overlay.includes(`class="tp p${k}"`) && v.overlay.includes(`.h${k}:hover~.p${k}`))`; `:6014` - `Math.abs(clockAt - pillOf(v.track).x) <= 0.05` | PASS |
| C15 | VS Code: bar + 140 (180 with the arrow) <= N x 6.5 where the bar keeps 120 px; 120 at N=30; divider <= N x 6.5; N=0 counts as 100. The checklist now notes that the arrow floor is 47 by the rule and 46 in the sources, left open for the author | `vscode_row_fits_its_columns` ok | `:6032` - `line <= cols * 6.5 && (cols * 6.5 - 140 >= 120 ? W + 140 <= cols * 6.5 : W === 120)` for N = 30, 40, 60, 100, 200, 300, 0; `:6033-6034` - `W === 120` at 30, and N=0 equals N=100; `:6042` - `W >= 120 && W + 180 <= n * 6.5` for 47, 60, 100 | PASS (see gap 1) |
| C16 | desktop bar = max(120, min(1400, max(320, 8N) - 140 - 40 with the arrow)) | `desktop_row_counts_8_px_a_column` ok | `:6048` formula; `:6056` - `W === want(n, false)` for 30/100/300; `:6061` - `W === want(120, true)` | PASS |
| C17 | one line per bar in the author's VS Code, pill named from 160 px | manual | not verifiable by the Verifier: the author has to check it in the extension after `/reload-plugins` | NOT VERIFIED (manual, outstanding) |
| C18 | 2 px dots on the 3 px grid of 7 rows; first window 10-35%, from 2/3 on 55-80%, none 100% | `fill_dots_scatter_on_a_ramp` ok | `:6117` - `x % 3 === 0 && (y - 1) % 3 === 0 && y >= 1 && y <= 19` (dots are parsed only as `h2v2h-2z`, `:194`); `:6130` - `onGrid && length >= 90 && windows[0][1] >= 0.1 && windows[0][1] <= 0.35 && head.length > 0 && head.every(([, x]) => x >= 0.55 && x <= 0.8) && windows.every(([, x]) => x < 1)`, over 3 ids x 3 configurations | PASS |
| C19 | under the zone at most 30% dots, at the resting brightness, no flicker, at 0/50/100% | `title_zone_dots_stay_faint` ok | `:6162` - `fade.every(([, o]) => o === fade[0][1]) && Math.abs(fade.at(-1)[0] - zone) <= 0.1 && fade[0][1] > 0`; `:6163` - `kfade.every(([, o]) => o === 0)`; `:6165` - `n / (cols * 7) <= 0.3 && faint && still` | PASS |
| C20 | same plan, width and state redrawn, including after a reload: same dots | `fill_dots_stay_put` ok | `:6181` - `a.length > 1000 && a === b && b === c`. Only the reload half (`c`) carries the claim: `reload()` re-imports the build as a fresh module (`engine.mjs:11`, `:221-227`). The `b` draw follows a `note` call, which `trackSig` ignores (`register.tsx:772`), so it is the cached picture | PASS |
| C21 | growing fill: every dot inside the earlier fill was already there, at the same place | `growing_fill_adds_no_dot_behind` ok | `:6208` - `checked > 1000 && bad.length === 0` | PASS |
| C22 | two ids, same width and progress: dots not all in the same places | `two_bars_scatter_their_own_dots` ok | `:6218` - `same = a.size === b.size && shared === a.size`; `:6219` - `a.size > 100 && !same` | PASS |
| C23 | wave slides one column per step; every dot in exactly one of 24 classes, each on a 50 ms timeline; wave at today's speed running, half at rest | `fill_stream_still_dots_wave_and_flicker` ok, `done_bar_streams_at_half_speed` ok | `:571` - `Number(shift) === Number(cols) * 3 && s.wpW === Number(shift)`; `:579` - `[...s.dots.values()].every(ks => ks.length === 1)`; `:584` - `s.timelines.length === 24 && ...`; `:585` - `onBeat(t.length) && onBeat(t.delay) && ...`; `:526` - `Math.abs(running - 11.65) < 0.01 && flickers(first) && rest.every(([, s, f]) => Math.abs(s - running * 2) < 0.02 && f)`. The 11.65 s literal is the loop the 1.3.0 build printed in round 1, and is 233 columns at 20 a second | PASS |
| C24 | twinkle off: no wave slide, no flicker class; twinkle on: reduced motion stops both | `twinkle_off_draws_the_fill_still` ok, `fill_stream_still_dots_wave_and_flicker` ok | `:1398` - `!flicker && !wave && kept && dots && moving && rest < 8000` (`flicker` at `:1390` includes `/class="q\d/`; `wave` at `:1391`); `:592` - `/@media \(prefers-reduced-motion:reduce\)\{\.wm,\[class\^="q"\]\{animation:none\}/` | PASS |
| C25 | 1400 px picture without the hover layer at most 131,072 chars in 4 states, twinkling, marks share full | `widest_bar_stays_under_the_svg_limit` ok | `:6234` - `track.width === 1400 && track.source.length <= 131072 && marks < 599 && twinkles` | PASS |
| C26 | light theme: dots in the 1.3.0 fill colours; no light rule recolours them | `light_theme_dots_keep_their_colours` ok | `:6241` FROZEN; `:6253` - `fills.every(c => c === FROZEN[state]) && selectors === '.ma .mp .pg .ti'`. Carried from 19eabb8: the frozen values reproduce on the 1.3.0 build, whose light rules were `.mp .ma` only | PASS |
| C27 | terminal band hashes to the 1.3.0 build; terminal cases unchanged | `terminal_band_is_drawn_as_in_1_3_0` ok, plus the 9 terminal cases ok | `:5751` - `got === '42755b970f0b007c'` (carried from 19eabb8: the 1.3.0 build prints the same hash, and the hashed tree carries each Raster's `cells`). Verified at f8e8849: the 9 listed case bodies (`:710`, `:722`, `:763`, `:811`, `:1403`, `:2676`, `:2939`, `:3477`, `:4002`) are still byte-identical to b0f064f | PASS |
| C28 | plugin.json 1.4.0; both READMEs describe the new row; no "grid of dots" claim left | `version_and_readmes_describe_the_new_row` ok | `:6283` - `version === '1.4.0' && !!rowEn && !!rowPt && !grid` (regexes at `:6280-6282`) | PASS |
| C29 | bars saved as 1.3.0 saves them come back drawn the new way, unconverted | `saved_bars_from_1_3_0_draw_the_new_way` ok | `:6264` - every saved field `JSON.stringify(F.bar(p.id)?.[k]) === JSON.stringify(v)`; `:6269` - `same && inside && !outside`. Carried from 19eabb8: `SAVED_1_3_0` (`:6260-6261`) is byte-identical to what the 1.3.0 build stores for the same calls, and HEAD stores the same text | PASS |

### New checklist wording against the task's criteria (verified at f8e8849)

- C1 now reads "No desktop e no VS Code". That matches criterion 1 ("no desktop ou no VS Code"), and nothing is dropped.
- C11 adds the surfaces and widths it is proven on, so it no longer contradicts criterion 11. It still leaves out criterion 11's "cut with … if needed, and the step after it if it fits" (gap 6). The VS Code 40-column leg draws a 120 px bar with no title zone, so it reaches the dot rule through a bar without a zone. The 160 px boundary itself is still tested only on the desktop, where it is 160 against 154.
- C15's added note does not contradict criterion 15, which names no column count. It does diverge from the task's Sources line and Unresolved #7, which both say "46 com a seta". The divergence is now stated in the check and left open for the author, so it is recorded rather than silent (gap 1).

## Rewritten and removed cases (verified at f8e8849)

These were compared against b0f064f. Since b0f064f, the diff changes only these pre-existing case bodies plus the shared `streamOf` helper, whose only callers at b0f064f were the three stream cases listed here. f8e8849 itself touched only `done_bar_streams_at_half_speed` among them, making it stronger.

| Case | Property the list says it keeps | Kept and declared? |
|---|---|---|
| `fill_stream_still_grid_wave_and_flicker` renamed to `..._dots_...` (`:552`) | still dots, one-column slide over a pattern covered once, one class per dot, 50 ms beat, `--f`/`--w` offsets, reduced motion | Kept. Both extra changes are now declared: `fades` (`:588`) starting faint above zero, and `>= 20` classes counted (`:577`). Still not asserted: that each laid class id has one of the 24 timelines (low) |
| `fill_stream_same_size_at_any_width` removed | its size guard moves to C25 | Kept |
| `fill_stream_reaches_the_fill_during_a_glide_back` (`:596`) | a shrinking fill is never left without dots | Kept. The 6 px tolerance (`:611`) is now declared |
| `twinkle_off_draws_the_fill_still` (`:1377`) | reads the dots; 8,000 ceiling counted without the dots' path | Kept (`:1397-1398`) |
| `done_bar_streams_at_half_speed` (`:506`) | reads one class use per class with dots; the plan gains a done step; now also pins the running loop at 11.65 s | Kept and stronger (`:526`); the new pin is declared |
| `marks_show_in_the_light_theme` | unchanged | Byte-identical to base |
| `more_agents_line_fits_a_narrow_strip` (`:5239`) | narrowest strip is VS Code at 30 columns (120 px) and the desktop at 40 | Kept, but still not fully declared. The ellipsis requirement is keyed to the observed width (`more.width !== 120 \|\| text.endsWith('…')`, `:5259`), not the input. If the strip stopped being 120 px, the cut path would silently go unasserted. The list's entry does not mention this |

## Swept rows marked existing (carried from 19eabb8)

`register.tsx` is unchanged, so its line numbers stand. The only refreshed citation is C15's N=0 leg.

- concurrency and ordering: present. Updates are computed inside `update()` from the latest stored list, at `register.tsx:2073` and `:2102-2107`.
- external-dependency failure: present. `register.tsx:3847` - `const cols = e.props.bodyColumns || 100`, asserted on `vscode` by C15's N=0 leg (`regress.mjs:6034`).
- observability: cannot be confirmed from the repo. The debug log for a skipped band belongs to the host engine, and neither `register.tsx` nor `tests/engine.mjs` has a limit or skip log. This is not a defect.

## Gaps (verified at f8e8849, ranked)

1. **Open, now recorded: C15's arrow floor is 47 columns, the decision record says 46.** The task (Sources, Unresolved #7) and design Key decision 3 say the row holds from 46 columns with the arrow. In round 1, a probe at 46 columns showed `floor(46 x 6.5)` = 299 px of room against a 120 + 180 = 300 px row; nothing in f8e8849 changes that, since `register.tsx` is untouched. C15 now states the divergence and leaves it open for the author. Still needs a decision: accept 47 and correct the design and task text, or trim 1 px from the reserve or the minimum so that 46 fits.
2. **Closed: VS Code was covered by width bounds only.** C1 now asserts the row composition on `vscode`. C11 now asserts the named pill there at 70 columns and the round dot at 40. The 70-column leg also catches a VS Code bar stuck at 120 px, which C15 alone would pass. Low residue:
   - VS Code width is still bounded loosely from below. At 70 columns the leg needs about 229 of the 315 px of room, so a bar up to about 86 px short would pass.
   - The other picture proofs (C2-C10, C12-C14, C18-C22) still render the desktop only. Both surfaces share that code, since only `total` branches on `e.surface` (`register.tsx:3848`).
3. **Closed: C23's "today's speed" was checked only as a ratio.** `:526` now pins the running loop at 11.65 s, the 1.3.0 value. The loop is `STREAM_COLS` (233) columns at any width, and `:571` fixes one column per step, so the pin fixes the speed itself.
4. **Open, unchanged: C2 loosens criterion 2's "12 px each side".** The 15% tolerance allows a right margin of about 18 px for "Refactor payments" (zone 132 against 101.7 px measured in Chromium). The `MEASURED` literals cannot be reproduced here.
5. **Mostly closed: rewrites beyond what the change list declared.** The `fades` change, the `>= 20` classes and the 6 px tolerance are now declared. Still undeclared: `more_agents_line_fits_a_narrow_strip` keying its ellipsis to the observed width (`:5259`). Low.
6. **Open, unchanged: checks narrower than their criteria.** All low:
   - C3 shows the whole title in the tip only where the bar holds it. This is an author decision, recorded.
   - C11 still drops criterion 11's cut/step clause. Pre-existing cases cover it, but `maxW` moved from `W` to `run` with no case for a narrow progress with a long name.
   - C18 keeps its ">= 90 columns" precondition against criterion 18's "Sempre".
   - C9 reads "green" off the pill rect alone.
7. **Informational, unchanged: expected values read back from the build.** C5-C8, C14, C18 and C19 read the zone back from the build's own fill head. C2 and C11 anchor it.

## Gate (verified at f8e8849)

Build: `node compile.cjs ../hooks/register.tsx <scratchpad>/verify/build.mjs` (251,906 bytes, the same size as round 1).

Full suite in one invocation: `node regress.mjs file:///.../scratchpad/verify/build.mjs` - 322 cases, 322 ok, 0 failed, `all passed`. All 42 named ids appear individually as `ok`: the 41 proof and rewritten-case ids from round 1, plus `speed_is_an_average_over_time`.

The timing-sensitive `speed_is_an_average_over_time` (`:2524`, WPC-210, flaky on 1.3.0 too) passed in this run. Its body is still byte-identical to b0f064f.

Read-only confirmation: before this report was rewritten, `git status --porcelain` showed only the pre-existing `?? .agents/`. This file is the only one modified. The builds and probes live in the session scratchpad `verify/` folder.
