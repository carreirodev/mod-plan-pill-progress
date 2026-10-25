# Plan bar title inside and scattered dots (WPC-250) Verification

**Verdict**: PASS - 29 of 30 checks proven; C17 is a manual check by the author, failed on 1.4.0, and must be repeated on 1.4.1
**Profile**: light (no `AGENTS.md`)
**Diff range**: b0f064f..8d99314. Round 3 covers the fix 5feb505..8d99314 on carreiro/wpc-250-vscode-row-fills-the-band; 5feb505 is the merge of the round-2 code into main, and its code matches f8e8849, with only `.checks` and `.design` text differing.
**Round**: 3 - scoped (the fix's diff, plus every verdict that was not PASS; proofs re-run in full at 8d99314)
**Verifier**: independent sub-agent (author != verifier)

What 8d99314 changed:
- `register.tsx`: on the `vscode` surface only, a column is 12 px when `$.plugin.root` starts with a drive letter and 8 px otherwise. The track Box holds only the base `Svg` (no clock, no hover layer), and the fold arrow is the text Button.
- `plugin.json`: 1.4.1.
- `regress.mjs`:
  - C15's case is rewritten to exact widths on both systems.
  - C11's VS Code widths moved to 70/30 columns.
  - C28's case accepts 1.4 or later.
  - A new case, `vscode_bar_goes_alone`, proves C30.
- The checklist (C11, C15, C28 and the new C30) and design Key decision 3 / vscode-row default #1 were updated.
- `tests/engine.mjs` and `tests/stub.mjs` are unchanged.

The fix changes behaviour only in VS Code. Desktop pictures and the desktop band tree are byte-identical before and after the fix. The terminal still hashes to the 1.3.0 build.

## The two extension facts C30 and C15 rest on (verified at 8d99314, in the installed bundle)

I read them myself in `C:\Users\Eduardo\.vscode\extensions\anthropic.claude-code-2.1.310-win32-x64\webview\index.js` (minified; byte offsets given) and `index.css`.

- **The column probe.** Near byte 5437300:
  - `oI0=["0","─"]`, `Yb5=0.5`, `Zb5=20`, `function sI0($,J){return Math.floor(($-Yb5)/Math.max(1,J))}`.
  - `Qb5` returns `columns:Math.max(Zb5,sI0(Q,X.width))`, where `Q` is the element's content width (offsetWidth less borders, padding and scrollbar) and `X` is the probe.
  - `bu` creates the probe with `X.textContent=oI0.join("\n")` and class `cellProbe_HKVJBw`. Its CSS is `font-family:inherit;line-height:1.4` and `position:absolute;white-space:pre;min-width:1ch`, so its width is the wider of "0" and "─".
  - The band's columns are `M=B?.columns` with `B=bu(U)`, where `U` is the band's own `band bandRoom` div, and are passed on as `bodyColumns:Z?.columns??0`.
  - The band sets no font of its own, so the probe inherits `body{font-size:var(--vscode-chat-font-size,13px);font-family:var(--vscode-chat-font-family)}`. That is the chat/UI font, not the editor font.
  - Confirmed as described: `bodyColumns = floor((band width - 0.5) / probe width)`. One detail the brief did not mention: there is a floor of 20 columns once measured, and 0 before.
  - The roughly 12 px value on the author's machine depends on fonts installed there. I could not confirm it (unconfirmed).
- **`ob0` has no `position` or hover.** At byte 5375006, the Box style mapper returns display, flexDirection, flexGrow, flexShrink, flexWrap (defaulting to `wrap` for rows), alignItems, justifyContent, row/column gap, width, height, min sizes, margins, paddings, border, borderRadius, backgroundColor and overflow. There is no `position`, `top`/`left`/`right`/`bottom` or hover. Confirmed.
  - The `Svg` node's schema is `KT5=F7({source,alt,width,height})`, with no `isInteractive`, and `rb0` (byte 5368551) draws it as an `<img>`. So a picture there cannot carry hover either, which backs C30's "draws a picture as a plain image".
  - The picture budget per tree is `pictures:250`, which does not constrain this.

## Steps not run under `light` (carried from 19eabb8)

- Step 1 (checks against the binding sources, screen by screen): not run. The variant-B composition was not compared element by element.
- `Coverage` join and `Test policy` verdicts: not run. The checklist has neither section.
- Step 4 (fault injection): not run in any round. No mutant was tried, including on the surfaces 8d99314 added (C15's exact widths on two systems, and C30).

## Checks (verified at 8d99314)

Line numbers are in `plugins/plan-progress/tests/regress.mjs` at 8d99314 unless noted. The fix moved lines only from line 6017 on (by 35 lines), so citations before that are unchanged since round 2. `register.tsx` citations moved by 1 to 3 lines and are refreshed.

Some 1.3.0-build confirmations are carried from 19eabb8: C26's colours, C27's hash and C29's stored text. Neither the literals nor the desktop/terminal output changed since.

| Check | Claim | Proof run | Evidence | Result |
|---|---|---|---|---|
| C1 | desktop and VS Code row is mark, bar, percent, fold cell, close; no title Text | `row_is_mark_bar_percent_close` ok | `:5773` - `for (const surface of ['desktop', 'vscode'])` (rows through `F.band(surface)`, `:5767`); `:5782-5786` - `same(plain[0], ['mark','bar','percent','close']) && same(folded.find(r => r.key === 'bar-a'), ['mark','bar','percent','fold cell','close']) && same(folded.find(r => r.key === 'bar-b'), ['mark','bar','percent','fold','close']) && !titled`. The VS Code fold arrow is now the text Button, and `kindOf` still reads its 5-wide cell as `fold` | PASS |
| C2 | title at x=12; shared zone = longest title + 24 (up to +15%); at most floor(0.3W) | `title_zone_fits_the_longest_title` ok | `:5797` - `zone >= MEASURED[title] + 24 && zone <= MEASURED[title] * 1.15 + 24`; `:5814` - `inside && fits(alone,'Docs') && fillOf(a) === fillOf(b) && fillOf(b) === longAlone && fits(longAlone,'Refactor payments') && fillOf(capped.source) === cap && titleIn(capped.source)?.[2].endsWith('…')` | PASS (see gap 6) |
| C3 | cut title ends in "…", with a hover area over the zone and a tip holding the whole title; a fitting title has neither | `cut_title_shows_whole_on_hover` ok | `:5831` - `!!tip && over.includes(`.${area[1]}:hover~.${tip[1]}`) && over.indexOf(area[0]) < over.indexOf(`class="tp ${tip[1]}"`) && Number(area[2]) === zone`; `:5833` | PASS (desktop; VS Code has no hover layer since 1.4.1, see gap 1) |
| C4 | 41 columns (188 px): no zone; 42 columns (196 px): zone; alt starts with the title | `narrow_bar_has_no_title_zone` ok | `:5850-5852` | PASS |
| C5 | head at zone + f(W - zone) ±0.1; pill, marks and areas between zone and W | `progress_runs_from_the_title_zone_to_the_end` ok | `:5874` - `Math.abs(fx - (zone + f * (W - zone))) <= 0.1`; `:5875` | PASS |
| C6 | fresh plan fills exactly the zone at the first stop; pill's left edge at the zone's end | `fresh_plan_fills_the_title_zone` ok | `:5894` - `Math.abs(fx - zone) <= 0.2 && x1 >= fx - 0.1 && first === '.05' && Math.abs(pill.left - zone) <= 0.2` | PASS |
| C7 | pill glides with its left edge never before the zone; fill glides from inside the progress | `pill_glides_inside_the_progress` ok | `:5908` | PASS |
| C8 | zone change redraws the other bars at the new zone with no `<animate>` | `title_zone_change_redraws_without_a_glide` ok | `:5930` - `!glides(during) && !glides(after) && Math.abs(fillOf(during) - third(bigZone)) <= 0.1 && ...` | PASS |
| C9 | done: green, title in its zone, pill at the right end with the total time | `finished_bar_keeps_its_title` ok | `:5941` - `green && titleIn(src)?.[2] === 'Task' && Math.abs(pill.right - 820) <= 0.1 && src.includes('>2m 5s<')`; green is the pill face `rx="11" fill="#18883A"` (`:5940`, `register.tsx:977/995`) | PASS (green read off the pill only) |
| C10 | contrast of at least 4.5:1 against the track over the page, the fill, and the dots at their brightest, 4 states x 2 themes | `title_reads_in_every_state_and_theme` ok | `:6127` - `Math.min(contrast(ink, track), contrast(ink, ground), contrast(ink, top))`; `:6129` - `worst >= 4.5` | PASS |
| C11 | at least 160 px of progress gives the named capsule, less gives the round dot; desktop 46/45 columns, VS Code off Windows 70/30 | `pill_name_needs_160_px_of_progress` ok | `:5958` - `run >= 160 ? named && !dot : dot && !named` (desktop); `:5961` - `for (const cols of [70, 30])`, `atCols(F, cols, 'vscode')` at `:5964` with the engine's default root `/plugin` (8 px); `:5969` - `cols === 70 ? run >= 160 && named && !dot : run < 160 && dot && !named`; `:5971` - `/: 160 px/.test(seen[0]) && /: 15\d px/.test(seen[1])`. Observed VS Code 70: 294 px, named; 30: 120 px, round dot | PASS (see gap 6) |
| C12 | strip hides the tool word at 356 px and shows it at 364 px | `strip_tool_word_keeps_its_360_px` ok | `:5984` - `seen[0].join() === '356,356,false' && seen[1].join() === '364,364,true'` | PASS |
| C13 | strips in the bar's column, right under it, as wide as it | `agent_strips_line_up_under_the_bar` ok | `:6001` | PASS |
| C14 | mark hover areas at zone + k/total x (W - zone), each with its tip; clock on the pill | `hover_marks_and_clock_follow_the_progress` ok | `:6012`; `:6014` - `Math.abs(clockAt - pillOf(v.track).x) <= 0.05` | PASS (desktop; VS Code has no hover since 1.4.1, see gap 1) |
| C15 | (renegotiated, 1.4.1) VS Code column = 12 px on Windows (drive-letter install), 8 px elsewhere; bar = floor(N x column) - 140 (180 with the arrow), at most 1400, else 120; divider never past N columns; N=0 counts as 100; both systems, N = 20/30/40/60/100/200/0, arrow N = 25/40/60/100 | `vscode_row_fits_its_columns` ok | `:6024` - `for (const [system, root, px] of [['Windows', 'C:\\...', 12], ['elsewhere', undefined, 8]])`; `:6028-6029` - `room = Math.floor((n \|\| 100) * px) - reserve`, `want = room >= 120 ? Math.min(1400, room) : 120`; `:6038` - `W === want(n, 140) && line <= (n \|\| 100) * px && (W === 120 \|\| W + 140 <= (n \|\| 100) * px)` over `:6033`'s N list; `:6046` - `W === want(n, 180) && (W === 120 \|\| W + 180 <= n * px)` over `:6043`'s list. The widths are now exact, not just upper bounds. Observed e.g. Windows 100: 1060 px, elsewhere 100: 660 px, N=0 the same as 100 | PASS (see gap 2) |
| C16 | desktop bar = max(120, min(1400, max(320, 8N) - 140 - 40 with the arrow)) | `desktop_row_counts_8_px_a_column` ok | `:6083` formula; `:6091` - `W === want(n, false)`; `:6096` - `W === want(120, true)` | PASS |
| C17 | one line per bar in the author's VS Code, pill named from 160 px | manual | Failed in the author's check on 1.4.0 (bar about half the band), which is what 8d99314 fixes. Not verifiable by the Verifier, and it needs repeating on 1.4.1 after `/reload-plugins` | NOT VERIFIED (manual, outstanding) |
| C18 | 2 px dots on the 3 px grid of 7 rows; first window 10-35%, from 2/3 on 55-80%, none 100% | `fill_dots_scatter_on_a_ramp` ok | `:6152` - `x % 3 === 0 && (y - 1) % 3 === 0 && y >= 1 && y <= 19`; `:6165` - `onGrid && length >= 90 && windows[0][1] >= 0.1 && windows[0][1] <= 0.35 && head.every(([, x]) => x >= 0.55 && x <= 0.8) && windows.every(([, x]) => x < 1)` | PASS |
| C19 | under the zone at most 30% dots, at the resting brightness, no flicker, at 0/50/100% | `title_zone_dots_stay_faint` ok | `:6197`; `:6198` - `kfade.every(([, o]) => o === 0)`; `:6200` - `n / (cols * 7) <= 0.3 && faint && still` | PASS |
| C20 | same plan, width and state redrawn, also after a reload: same dots | `fill_dots_stay_put` ok | `:6216` - `a.length > 1000 && a === b && b === c`. Only the reload half carries the claim: `engine.mjs:11`, `:221-227` re-import the module, while the `note` redraw is the cached picture (`register.tsx:773` `trackSig`) | PASS |
| C21 | growing fill: every dot inside the earlier fill was already there | `growing_fill_adds_no_dot_behind` ok | `:6243` - `checked > 1000 && bad.length === 0` | PASS |
| C22 | two ids: dots not all in the same places | `two_bars_scatter_their_own_dots` ok | `:6253-6254` - `a.size > 100 && !same` | PASS |
| C23 | wave one column per step; each dot in one of 24 classes on 50 ms timelines; today's speed running, half at rest | `fill_stream_still_dots_wave_and_flicker` ok, `done_bar_streams_at_half_speed` ok | `:571`; `:579`; `:584`; `:585`; `:526` - `Math.abs(running - 11.65) < 0.01 && flickers(first) && rest.every(([, s, f]) => Math.abs(s - running * 2) < 0.02 && f)` | PASS |
| C24 | twinkle off: no wave slide, no flicker class; twinkle on: reduced motion stops both | `twinkle_off_draws_the_fill_still` ok, `fill_stream_still_dots_wave_and_flicker` ok | `:1398` - `!flicker && !wave && kept && dots && moving && rest < 8000`; `:592` - the `prefers-reduced-motion` rule | PASS |
| C25 | 1400 px picture without the hover layer at most 131,072 chars, 4 states, twinkling, marks share full | `widest_bar_stays_under_the_svg_limit` ok | `:6269` - `track.width === 1400 && track.source.length <= 131072 && marks < 599 && twinkles` | PASS |
| C26 | light theme: dots in the 1.3.0 fill colours; no light rule recolours them | `light_theme_dots_keep_their_colours` ok | `:6276` FROZEN; `:6288` - `fills.every(c => c === FROZEN[state]) && selectors === '.ma .mp .pg .ti'` (frozen values carried from 19eabb8) | PASS |
| C27 | terminal band hashes to the 1.3.0 build; terminal cases unchanged | `terminal_band_is_drawn_as_in_1_3_0` ok, plus the 9 terminal cases ok | `:5751` - `got === '42755b970f0b007c'`, printed again at 8d99314. The 9 case bodies (`:710`, `:722`, `:763`, `:811`, `:1403`, `:2676`, `:2939`, `:3477`, `:4002`) are re-checked as byte-identical to b0f064f | PASS |
| C28 | plugin.json 1.4.0 or later; both READMEs describe the new row; no "grid of dots" claim | `version_and_readmes_describe_the_new_row` ok | `:6320` - `atLeast = major > 1 \|\| (major === 1 && minor >= 4)`; `:6321` - `atLeast && !!rowEn && !!rowPt && !grid` (regexes `:6315-6317`). Observed 1.4.1 | PASS (see gap 1: criterion 28 still names 1.4.0) |
| C29 | bars saved as 1.3.0 saves them come back drawn the new way, unconverted | `saved_bars_from_1_3_0_draw_the_new_way` ok | `:6299`; `:6304` - `same && inside && !outside`; `SAVED_1_3_0` at `:6295-6296` (carried from 19eabb8) | PASS |
| C30 | (1.4.1) in VS Code the bar goes alone in its Box: no hover layer, no clock, no absolutely placed Box anywhere in the band, fold arrow a text Button; the desktop keeps all three | `vscode_bar_goes_alone` ok | `:6066-6070` - `pictures` (Svgs in `track-t`), `interactive` (any interactive Svg in the band), `placed` (any `position === 'absolute'` Box in the band), `clock`, `fold` label; `:6078` - `code.pictures === 1 && !code.interactive && !code.placed && !code.clock && code.fold === '⌃' && desk.pictures === 3 && desk.interactive && desk.placed && desk.clock && desk.fold !== undefined && unpadded(desk.fold) === ''`. Code: `register.tsx:3917` (clock `&& !isVscode`), `:3927` (hover Box `isVscode ? null`), `:3947` (chip `Svg && !isVscode`). The case runs with an agent strip, so the strips are covered too. Only the unfolded state (`⌃`) is drawn | PASS |

### C15, C11, C28 and C30 against their own text and the task and design as now written (verified at 8d99314)

- **C15** matches its renegotiated text. It also matches design Key decision 3, which gives the floors as 22/33 columns, or 25/38 with the arrow; 260/12 = 21.7, 260/8 = 32.5, 300/12 = 25 and 300/8 = 37.5 all agree. It matches vscode-row default #1 (12 / 8 px), and the extension's 20-column minimum makes N = 20 the smallest real value.
  - Against the task: criterion 15's main text still says "N × 6,5 px". Its new parenthetical records the 12/8 px change and the dropped hover layer, and C15 follows the parenthetical. C15 is stronger than the criterion: an exact width instead of a cap.
  - The task's Sources line (line 115) and Unresolved #7 (line 139) still give the floor as "40 colunas (47 com a seta)" at 6.5 px (gap 1).
- **C11** matches its text ("VS Code fora do Windows (70 e 30 colunas)", run under the engine's default `/plugin` root). There is no Windows-width leg, but the pill rule depends only on the track width, which C15 pins exactly on Windows. The criterion 11 cut/step clause is still dropped (gap 6).
- **C28** matches its text. It is looser than criterion 28, which still reads "plugin.json diz 1.4.0", and it would accept any 1.4+ or 2.x version (gap 1).
- **C30** matches its text and the extension facts above. The task has no criterion 30: C30's source is criterion 15's parenthetical and Key decision 3. Neither of those mentions the fold arrow becoming a text Button. That follows from Key decision 3's "no boxes over others", but it is written down only in C30 and the code (gap 1).

### The fix changes only VS Code (verified at 8d99314)

- **Code read.** `colPx` (`register.tsx:3849`) is used only in the `isVscode` arm of `total` (`:3851`). The desktop arm `Math.max(320, cols * 8)` is unchanged. The clock, hover Box and fold chip are each gated on `!isVscode` (`:3917`, `:3927`, `:3947`), so desktop and terminal reduce to the old expressions.
- **Probe.** I compiled 5feb505's `register.tsx` and 8d99314's into the scratchpad. I hashed every picture (alt, width, interactivity, source) and the band tree on each surface, over: 4 states, 40/120/200 columns, twinkle on and off, a bar with an agent, and install roots `/plugin` and `C:/...`.
  - desktop: `0e9f3bbd12ff05f4` on both builds.
  - vscode: `0c91517b8cd65e38` becomes `29dea6453b57b165`, the intended change.
- **Terminal.** C27's hash `42755b970f0b007c` still matches at 8d99314.

## Rewritten and removed cases (carried from f8e8849; re-checked at 8d99314)

The fix touched only C11's, C15's and C28's cases and added C30's, so the round-2 table stands:
- `fill_stream_still_dots_wave_and_flicker`, `fill_stream_reaches_the_fill_during_a_glide_back`, `twinkle_off_draws_the_fill_still` and `done_bar_streams_at_half_speed`: kept and declared.
- `fill_stream_same_size_at_any_width`: removed, with its size guard moved to C25.
- `marks_show_in_the_light_theme`: byte-identical.
- `more_agents_line_fits_a_narrow_strip`: still keys its ellipsis to the observed width (`:5259`, undeclared, gap 6). It still draws a 120 px VS Code strip at 30 columns under the 8 px default root.

The rewritten C15 case (`:6017`) replaces round 2's upper-bound checks with exact equality on both systems. No assertion was weakened: the new `want()` is stricter.

## Swept rows marked existing (carried from 19eabb8, citations refreshed)

- concurrency and ordering: present, at `register.tsx:2074` and `:2103-2108`, where updates are built inside `update()` from the latest list.
- external-dependency failure: present, at `register.tsx:3850` - `const cols = e.props.bodyColumns || 100`. On both VS Code systems, C15 asserts N=0 through `want()`'s `(n || 100)` (`:6028`).
- observability: cannot be confirmed from the repo. The skipped-band log belongs to the host engine (`register.tsx:627` names the limit, and no skip log exists in `register.tsx` or `engine.mjs`).

## Gaps (verified at 8d99314, ranked)

1. **New, open: the task (the record of decision) did not keep up with the 1.4.1 renegotiation.** The task file is gitignored, so this is local text only:
   - Sources (line 115) and Unresolved #7 (line 139) still say the floor is "40 colunas (47 com a seta)" at 6.5 px. Key decision 3 now says 22/33 (25/38 with the arrow).
   - Sources (line 124) still says the extension counts "pelo avanço da fonte do editor". The bundle measures "0"/"─" in the inherited chat/UI font, and Key decision 3 was corrected.
   - Criterion 28 still says "1.4.0".
   - Criteria 3 and 14 (title tip, clock, mark tips) still read as if for every surface. VS Code now draws no hover layer, and that exception lives only in criterion 15's parenthetical and Key decision 3.
   - C30 has no criterion of its own, and its "fold arrow is a text Button" is in neither the task nor the design.
2. **New, precision gap in C15 against vscode-row default #1 (not concrete for this user today).** Default #1 puts the 12 px down to the fallback font on the machine that renders the panel. C15 and the code (`register.tsx:3849`) take "Windows" to mean a drive-letter install path. In a VS Code remote window, such as Remote-WSL, those two could diverge: the bar would be sized at 8 px for roughly 12 px columns and fill about two thirds of the band.
   - Two facts point that way. The extension's `package.json` declares no `extensionKind`, which by VS Code's default (from memory, unconfirmed) makes it a workspace extension that runs on the remote side.
   - This user has no VS Code server in WSL (`~/.vscode-server` does not exist), so nothing exercises the case today.
   - Separately, "8 px elsewhere" is an estimate the design marks as not measured.
   - Neither case can be caught by the suite.
3. **Closed: C15's arrow floor (46 vs 47 columns, gap 1 in round 2).** It no longer applies: the 1.4.1 floors are 25 (Windows) and 38 (elsewhere) with the arrow, and C15 asserts exact widths around them. The stale text it left in the task is item 1.
4. **Closed: VS Code coverage residue (gap 2 in round 2).** C15 now pins the VS Code bar to the full room on both systems, so a short bar fails it. The other picture proofs still render the desktop only, on code the fix leaves shared.
5. **Closed and carried: C23's speed (gap 3 in round 2).** It is pinned at 11.65 s (`:526`).
6. **Open, unchanged:**
   - C2's 15% tolerance loosens criterion 2's "12 px each side" (about 18 px of right margin for "Refactor payments").
   - `more_agents_line_fits_a_narrow_strip`'s observed-width ellipsis is undeclared.
   - These checks are narrower than their criteria: C3 (whole tip only where the bar holds it), C11 (cut/step clause dropped; `maxW` moved from `W` to `run` untested), C18 (the ">= 90 columns" precondition), C9 (green read off the pill).
   - All low.
7. **Informational, unchanged:** C5-C8, C14, C18 and C19 read the zone back from the build's own fill head, and C2 and C11 anchor it.

## Gate (verified at 8d99314)

Build: `node compile.cjs ../hooks/register.tsx <scratchpad>/verify/build.mjs` (252,319 bytes).

Full suite in one invocation: `node regress.mjs file:///.../scratchpad/verify/build.mjs` - 323 cases, 322 ok, 1 failed. All 42 named proof and rewritten-case ids appear individually as `ok`, including the new `vscode_bar_goes_alone`.

The one failure is `speed_is_an_average_over_time` (`:2524`, WPC-210, flaky on 1.3.0 too). Its body is byte-identical to b0f064f and it is not one of this feature's checks. It passed in 3 of 3 solo reruns (`ONLY=speed_is_an_average_over_time`).

Read-only confirmation: before this report was rewritten, `git status --porcelain` showed only the pre-existing `?? .agents/`. This file is the only one modified. The builds (8d99314, 5feb505 and b0f064f) and the one-off probes live in the session scratchpad `verify/` folder.
