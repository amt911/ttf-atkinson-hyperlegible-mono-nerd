# ttf-atkinson-hyperlegible-mono-nerd — Claude Guide

A tiny Arch Linux packaging repo. A single `PKGBUILD` clones the upstream
[Atkinson Hyperlegible Next Mono](https://github.com/googlefonts/atkinson-hyperlegible-next-mono)
font, patches every TTF through the **Nerd Fonts** `font-patcher` (FontForge)
to inject the Nerd Fonts glyph set (icons, Powerline, etc.), and installs the
results into `/usr/share/fonts/TTF`. Package name:
`ttf-atkinson-hyperlegible-next-nerd-mono-git`.

## Start here

The whole project is one `PKGBUILD` plus (optionally) a generated `.SRCINFO`.
There is no application code — just read `PKGBUILD` end to end before changing
anything. It is a `-git` VCS package: `pkgver` is derived from the upstream
git history, so it changes on every rebuild.

## ⚡ superpowers — use whenever applicable

Always prefer **superpowers** skills over ad-hoc approaches. If there's even a small chance a skill applies to the task, invoke it via the `Skill` tool before acting (including before clarifying questions).

- **Process skills first** — `brainstorming` before creative/feature work, `systematic-debugging` before fixing bugs, `test-driven-development` before writing implementation.
- **Then implementation skills** — domain-specific skills guide execution.
- **Verify before claiming done** — `verification-before-completion` / `requesting-code-review` before merging.

User instructions always take precedence over skills; skills override default behavior.

### Mode switch

- **"lite mode"** — fully disables superpowers: no skill is invoked, not even the applicability check, until **"normal mode"** is said.
- **"normal mode"** (default) — standard superpowers behavior, plus: when delegating coding work, dispatch at most 1 **implementation** agent at a time (a read-only review agent runs alongside it — see **Agent orchestration**), and never use a model above Sonnet (no Opus).
- **"modo desatendido"** (unattended mode) — the user is away and delegates autonomy: work without waiting for confirmations and make reasonable decisions yourself instead of asking. In this mode you MAY **`git push` the feature branches you create** and **open PRs via `gh`** on your own, so the work is ready for review when the user returns. The hard limits still hold and are NOT lifted: **never merge anything** (no `git merge`, no fast-forward integration, no `gh pr merge`), **never push to `main`** or any protected/default branch directly, and **never** `git push --force` / `--force-with-lease`. Deliver everything as pushed branches + PRs for the user to merge. Reverts to defaults on **"normal mode"**.

Confirm the switch briefly when it happens.

## Stack

- **Arch Linux packaging** — `PKGBUILD` (bash), built with `makepkg`. `arch=('any')`, `license=('OFL-1.1')`.
- **Nerd Fonts `font-patcher`** — the `font-patcher` package provides `/usr/share/font-patcher/font-patcher`; declared as a `makedepends`.
- **FontForge** — scripting engine that runs `font-patcher` (`fontforge -script …`).
- **Upstream source** — `git+https://github.com/googlefonts/atkinson-hyperlegible-next-mono.git`, checksummed `SKIP` (VCS source).

## Commands

```bash
# build + install the package locally (runs pkgver/build/package, resolves deps)
makepkg -si

# build only, force a clean rebuild, keep the built package
makepkg -f

# refresh source checksums in PKGBUILD after changing sources
updpkgsums

# (re)generate the AUR metadata after editing PKGBUILD
makepkg --printsrcinfo > .SRCINFO

# lint the PKGBUILD and the built package for common mistakes
namcap PKGBUILD
namcap *.pkg.tar.zst

# inspect what actually landed in the package
tar tvf *.pkg.tar.zst | grep -E 'fonts|licenses'

# after install, verify the OS sees the family and its Nerd variants
fc-list | grep -i atkinson

# patch a single TTF by hand (what build() does per file, for debugging)
fontforge -script /usr/share/font-patcher/font-patcher -q \
  --outputdir output/ --complete --careful --makegroups 5 --metrics TYPO in.ttf
```

The `build()` step fans out across all `fonts/ttf/*.ttf` with `xargs -P $(nproc)`,
patching each with `--complete --careful --makegroups 5 --metrics TYPO`.

## Tests and quality

There are no unit tests — quality here means the package builds reproducibly and
the patched fonts are correct. Verify these before considering a change done:

- **Package builds cleanly** — `makepkg -f` completes with no errors; `makepkg -si`
  installs without conflicts.
- **`namcap` is clean** — run on both `PKGBUILD` and the built `*.pkg.tar.zst`;
  address warnings (wrong deps, missing license, bad permissions) or justify them.
- **PKGBUILD ↔ `.SRCINFO` in sync** — regenerate `.SRCINFO` with
  `makepkg --printsrcinfo > .SRCINFO` after ANY change to `PKGBUILD` metadata
  (pkgver/pkgrel/deps/source). A stale `.SRCINFO` is the most common AUR mistake.
- **Checksums current** — after changing `source=()`, run `updpkgsums`. VCS
  (`git+…`) sources legitimately use `SKIP`; leave those as-is.
- **Patched fonts contain the Nerd glyphs** — the whole point of the patch. After
  install, confirm the extra ranges landed, e.g. `ttx -t cmap output/<Font>.ttf`
  and check for Nerd Fonts codepoints (Private Use Area, e.g. `U+E000–F8FF`,
  `U+F0000+`), or open in a font viewer and look for icons/Powerline arrows.
- **TTFs are valid** — `fontlint output/*.ttf` (from FontForge) reports no fatal
  errors; `ttx output/*.ttf` round-trips without exceptions.
- **Monospace advance widths preserved** — this is a MONO font. The patch must not
  break fixed-pitch: confirm all glyphs share one advance width (the reason
  `--complete --careful` and `--makegroups`/`--metrics TYPO` are used rather than
  a naive patch). Spot-check with `ttx -t hmtx` and confirm uniform `width`.
- **Installs to the right place** — files land under `/usr/share/fonts/TTF` and the
  license under `/usr/share/licenses/${pkgname}/`; `fc-list` shows the family.

If you change patcher flags, rebuild and re-verify glyph coverage AND advance
widths — those two properties are what this package exists to get right.

- **Mutation gate (60%) — does not apply here, and that is why it is written down.** The template
  requires a 60% mutation threshold over business logic; this repo packages font files and has no
  executable logic of its own to mutate. The checks above are its equivalent: the built fonts have
  to be the files they claim to be. If a script with tests ever lands here, the rule applies from
  that day.

## Real-system verification — what no green build can prove

The section above lists the right gates. This one names the **kinds** of check they are, so one can
be asked for by name, and states the rules for writing one worth trusting.

The framing that matters for a font package: **`fontlint` clean, `ttx` round-tripping and `fc-list`
finding the family do not mean a single icon renders.** They are structural checks on a file. The
product is glyphs on a screen. The failure mode this package exists to prevent — tofu where a
Powerline arrow or a git-branch glyph should be — is *visual*, and no structural check reports it.

What "real system" means here, concretely:

- **A real display, at a real size.** Install, `fc-cache -f`, then open a terminal and an editor
  configured with the patched family and look. Rasterisation depends on the FreeType version,
  fontconfig rules, subpixel settings and DPI — a font that reads well on a 4K panel can be muddy at
  1080p, and this is a *legibility* typeface, so that is the whole point.
- **A clean chroot for the build**, not your dev box: `extra-x86_64-build` or `makechrootpkg`. A
  build that works only because `font-patcher` or FontForge happens to be installed globally is a
  missing `makedepends` that fails for everyone else — silently, for you.
- **A throwaway system for the install.** `makepkg -si` installs fonts on your daily machine and
  rewrites your fontconfig cache; a bad build there is a real problem, not a failed test.
- **A terminal, specifically.** This is the MONO variant: the property that breaks is column
  alignment, and it breaks *visually* long before `ttx -t hmtx` looks wrong to you. Print a box-drawing
  table and a column of icons and confirm nothing shifts.

### The names, so you can ask for them by name

| Name | What it means here |
| --- | --- |
| **E2E / on-system acceptance test** | Build in a clean chroot, install on a throwaway system, and assert on observable results — `pacman -Ql` lists the TTFs under `/usr/share/fonts/TTF` and the license under `/usr/share/licenses/`, `fc-list` reports the family, and **the icons actually render in a terminal**. Never on the build log. |
| **Contract test** | Checks that assumptions about **things you do not control** still hold, which for a `-git` package is most of them. The `source` floats to upstream's default branch, so the input fonts change under you between builds. `font-patcher` is a `makedepends`, so its flag semantics move with the Nerd Fonts release — `--complete`, `--careful`, `--makegroups 5` and `--metrics TYPO` have each changed meaning or naming output across versions. FontForge's own version changes what comes out. Record which upstream commit and which patcher version produced a build, or a regression is unattributable. |
| **Mutation testing** (here: by hand) | Drop a patcher flag or a `makedepends`, rebuild in the chroot, confirm the check you rely on goes red, restore. **A check that has never failed has not been tested** — a glyph-coverage check that has never seen an unpatched font proves nothing. |
| **State-invariant test** | Asserts a relationship **between two things** neither one alone can prove: `PKGBUILD` against the generated metadata file; the family name baked into the TTF against what `fc-list` reports against what a user writes in their terminal config; the patched output against the upstream input it came from. Each side can be individually valid while the pair is wrong. |
| **Test pollution / isolation leak** | State that outlives a run and changes the next one. The sharp one here is the **fontconfig cache**: an older installed version can keep rendering after a bad rebuild, so the icons look fine and the package is broken — always `fc-cache -f` and re-check, and prefer verifying on a machine that never had a previous build installed. Building outside a chroot is the same class of problem. |

### Rules that came out of real bugs, not theory

- **Prove every check can fail before you trust it green.** Patch a font with `--complete` removed,
  confirm your coverage check reports the missing ranges, restore. An unexercised check is decoration.
- **Never assert on a count you cannot predict.** This is the specific trap for this package: the
  number of Private Use Area codepoints in a patched font tracks the **Nerd Fonts release**, not your
  build — so `ttx -t cmap … | grep -c 'code=0xe'` will happily report a healthy-looking number for a
  half-patched font, and will "fail" a perfectly good one after an upstream glyph-set change. Assert
  **named codepoints you actually use** instead: `U+E0B0` (Powerline arrow), `U+F09B` (git branch),
  and whichever glyphs your prompt and editor really draw. Same for file size and glyph totals.
- **A checksum, generated metadata file or `pkgrel` must die with the source it describes.** VCS
  sources legitimately use `SKIP`, so the *only* thing tying a build to its input is what you record
  about it — stale metadata beside a rebuilt font produces a package that installs happily and is
  wrong, with nothing failing.
- **Never test destructively on your own machine.** Chroot to build, container or VM to install. If
  you do install locally, know `pacman -Rns` and `fc-cache -f` before you start.
- **Run the build the way that actually works here.** Patching fans out with `xargs -P $(nproc)` and
  FontForge is memory-hungry, so on a many-core box cap the fan-out (or run it under a memory ceiling)
  rather than discovering the limit by taking the machine down. And **claim exactly what you
  verified**: "`makepkg` succeeded" is not "the icons render".

## Debugging — keep the loop from running away

What a bug costs is not the fix. It is how many times you go around
`build → deploy → reach the state → observe` before you know what to fix, times what one lap costs.
Every rule below carries the number it came from; the ones this repo has not measured are marked
`<!-- pendiente de medir -->` until someone does.

- **Measure before you ablate.** Ablation costs one lap per hypothesis and answers yes/no;
  instrumentation costs one lap total and answers *what is actually happening*. **Measured: 28
  ablations over 1 h 42 min moved nothing; one 13-min batch of probes changed the question and the
  bug fell on the next round.** The rule that came out of it: **if a pipeline completes every phase
  with non-empty output, the output exists** — stop asking "why doesn't it appear" and ask "where
  does it appear". Here that pipeline is `fetch/clone → build → package → install`.
- **Budget the lap, then attack the dominant term.** Time the four phases once and write the real
  seconds in; one dominates and the rest are noise. **If a bug needs more than three reproductions,
  write the shortcut before the fourth** — here that means
  `makepkg -e` / `--noextract` to skip re-fetching, a VM snapshot at the starting state, or the
  script's `--dry-run`.
  Commit it as `<scripts/repro-<bug>.sh>` and name it in `docs/FINDINGS.md` (create it from the starter kit if this repo has none yet).

  | Lap phase | Command here | Measured |
  | --- | --- | --- |
  | build | `makepkg -f` | `<n s>` |
  | install | `<pacman -U · chroot · VM>` | `<n s>` |
  | reach the state | `<boot the clean VM>` | `<n s>` |
  | observe | `<exit code · pacman -Ql · files on disk>` | `<n s>` |

- **A review finding is not a reproduction.** Whoever reviewed read the code; they did not run it.
  Reproduce it yourself before sending anyone to fix it, and **if the implementer says they cannot
  reproduce it, believe the implementer** — one of them has the thing running. **Measured: 1 h 25 min
  chasing a bug that did not exist.**
- **A test that refuses to go red is data, not a failure.** The fourth failed attempt to pin down
  that non-existent bug is what uncovered the real one, pointing the opposite way. "I cannot make
  this fail" is a result and it gets reported; a green test papered over it throws the signal away.
- **Before demanding a red, ask whether the mechanism can produce one.** If another layer masks the
  effect there will be no red however hard you push, and the time goes into the test instead of the
  bug. **Measured: over 1 h on two structurally impossible reds.**
- **Assertions that are inert by construction** — none of these shows up as a failure, a warning or
  a coverage drop. **Every assertion is watched failing once**, and expected values are written by
  hand:

  | Inert by | What it looks like here |
  | --- | --- |
  | a check without `set -e` | the script carries on past the failure and exits 0 |
  | a failure inside a pipe | without `set -o pipefail` the exit code is the last command's, not the one that failed |
  | a `grep` whose status is ignored | `grep pattern file` with no `-q` and no checked exit status verifies nothing |
  | `set -e` inside an `if` or a short-circuit | it does not apply there: a failure in that branch is invisible |
  | comparing against your own output | the expected value is generated by the very script being checked |

- **Verify the resource limit reaches the process doing the work.** A job wrapped in a memory scope
  can hand the work to a daemon or worker pool living outside it, and the tool still reports the
  limit as applied — over a process that is idle. Check the **worker's** cgroup
  (`cat /proc/<worker-pid>/cgroup`), not the scope's.
- **Environment claims get measured or they don't get made.** "That heap sounds low" produced a
  recommendation that was simply wrong; measuring it — three runs per setting, not one — gave a
  **0.4% difference, below the run-to-run variance**. No performance tuning lands without a
  before/after over more than one run.
- **Locate which layer owns a rule before deciding which side gives.** A rule that lives in one
  layer and isn't shared by the others fails where the assumption breaks, not where it is written,
  which is why the fix keeps landing in the innocent layer.
- **Replacing a component can remove capabilities in silence.** When you swap one API for another,
  enumerate what the old one did that the new one does not, and say it in the PR — nothing will fail
  to compile. An optional parameter that defaults to off is a capability that only exists if the
  caller remembers it.

## Agent orchestration — parallel where it's free, batched where it's yours

Delegating to agents moves the bottleneck to **scheduling**: what waits on what, what each agent
re-derives, and which decisions quietly stop being yours. Same convention — every rule carries its
measured number.

- **Review is not on the critical path.** Reviewing task N and starting N+1 are independent when
  they touch different files. Serialized, review is **10-15% of the wall clock** and blocks
  everything behind it; in parallel it is free. **On receiving an implementation report, dispatch
  its review and the next implementation in the same turn.** This is the one exception to
  *"at most 1 agent at a time"*: the cap counts **implementation** agents — a review agent reads and
  reports, it writes nothing, so it cannot race the implementer. **The exclusive resource here is:**
  the test VM or chroot, and the `pacman` lock
  — at most one agent touching it.
- **Keep one shared facts file.** Every fresh agent re-derives the same things: the real selector,
  which fake exists, what that helper accepts. Keep `docs/FACTS.md`, have each agent append to it
  when it finishes, and hand it to the next one in its dispatch. Only **facts verified against the
  repo or the running system**, with how they were verified. It is not the gotchas log: that holds
  what is *not* deducible from the code and outlives the branch; this holds what is perfectly
  deducible and merely expensive to look up, and it may die with the branch.
- **Plans carry contracts, not literal code.** The agent **trusts** the code in the plan; code you
  never compiled is an error wearing authority. **Measured: 4 wrong blocks, 15-40 min of detour
  each.** Write exact names, exact signatures and "mirror the shape of `<X>`" — claims the agent can
  check against the repo — and reserve literal code for what you have run.
- **Batch the discretionary decisions.** Work that appears along the way — a capability being
  dropped, a missing script, an adjacent bug — added **5-6 h of 15**. Each was justified; deciding
  them on the fly is what takes them away from you. Accumulate and ask **once per batch, with the
  estimated cost**. In **"modo desatendido"** the batch goes in the PR body instead, with its costs.
- **What never gets cut.** Review was **1.5 h of 15** and found a `create()` silently discarding
  fields, a 404 caused by SQL deduplication, a silent merge that corrupted data, a
  delete-and-recreate with no transaction, and several inert assertions. **Cutting review does not
  give time back; it defers it to production.** Cut reproduction (write the shortcut) and
  serialization (dispatch review in parallel) instead.

### Day one — the numbers that fill the blanks

1. **The lap** — time `build → deploy → reach the state → observe` once and write the seconds into
   the table above. The dominant phase gets the shortcut script; the rest stay unoptimized.
2. **The exclusive resource** — confirm the one named above is really the only one.
3. **The inert assertions** — break one assertion on purpose and run the suite; anything still green
   is inert. Then prune the table above to what this stack can actually produce.

## Working rules

- **Use superpowers skills whenever they apply** — invoke via `Skill` before acting; process skills before implementation skills.
- **New dependencies: ask first, then add** — adding a `makedepends`/`depends` entry is allowed when
  the package genuinely needs it, but ask before adding it (which package, why) and wait for the
  go-ahead. Those lists are intentional, and `font-patcher` and its FontForge path are load-bearing
  — same rule before swapping them.
- **Regenerate `.SRCINFO` whenever `PKGBUILD` metadata changes** — never commit a `PKGBUILD` change without the matching `.SRCINFO`.
- **Reuse before you write** — this recipe has a sibling (`ttf-atkinson-hyperlegible-nerd`): when the patch flags, the `pkgver()` derivation or the install layout change here, check the other and keep both in the same shape instead of growing a second style. Dependency names come from the official repos, the `font-patcher` invocation is the single place the flags live, and no value that `pkgver()`, `updpkgsums` or `makepkg --printsrcinfo` already derives gets re-typed by hand.
- **Keep it reproducible** — don't hardcode a pkgver; the `pkgver()` function derives it from upstream git. Bump `pkgrel` when the packaging (not upstream) changes.
- **Don't vendor built fonts or `pkg/`/`src/` into git** — these are build artifacts. Only `PKGBUILD` (and `.SRCINFO`) are tracked.
- **UI work → N/A** — this repo has no UI/frontend surface.
- **Instrument before you ablate, budget the lap, and dispatch review in parallel** — a pipeline that completes with non-empty output produced output; more than three reproductions means you owe a shortcut script; a review finding is not a reproduction; and the review of task N runs alongside the implementation of N+1. See **Debugging** and **Agent orchestration** above.

## Git & GitHub

- **Commits and branches OK** — create commits and new branches whenever it makes sense, without asking first.
- **Never push** *(default)* — no `git push` under any circumstance, and absolutely never `git push --force` / `--force-with-lease`. Leave pushing to the user. **Exception:** when **"modo desatendido"** is active, you may push the feature branches you create (never `main`/protected branches, never force) so PRs are ready for review.
- **Never merge — no permission** — you do NOT have permission to merge anything into any branch, nor to merge any pull request. No `git merge`, no fast-forward integration, no `gh pr merge`. Leave every merge (branches and PRs alike) to the user. This holds in every mode, **including "modo desatendido"**.
- **GitHub via `gh`** — if the `gh` CLI is available, you may open pull requests, issues, and similar (comments, labels, etc.). These don't require pushing on your part beyond what `gh` itself does for an already-pushed branch.
- **Every PR must include a manual test plan** — when opening a PR, add a **How to test manually** section with the exact steps to exercise the change by hand. Here that means: run `makepkg -si` (or `makepkg -f`) so it builds and installs cleanly, run `namcap` on the PKGBUILD and package, then verify the font renders with Nerd icons — e.g. `fc-list | grep -i atkinson` and open a patched TTF in a viewer / terminal to confirm the injected glyphs (icons, Powerline) show. Note any deps to install first (`font-patcher`, `fontforge`).
