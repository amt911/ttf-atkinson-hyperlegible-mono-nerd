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
- **"normal mode"** (default) — standard superpowers behavior, plus: when delegating coding work, dispatch at most 1 agent at a time, and never use a model above Sonnet (no Opus).
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

## Working rules

- **Use superpowers skills whenever they apply** — invoke via `Skill` before acting; process skills before implementation skills.
- **Don't add dependencies casually** — `makedepends`/`depends` are intentional. `font-patcher` and its FontForge path are load-bearing; don't swap them without asking.
- **Regenerate `.SRCINFO` whenever `PKGBUILD` metadata changes** — never commit a `PKGBUILD` change without the matching `.SRCINFO`.
- **Keep it reproducible** — don't hardcode a pkgver; the `pkgver()` function derives it from upstream git. Bump `pkgrel` when the packaging (not upstream) changes.
- **Don't vendor built fonts or `pkg/`/`src/` into git** — these are build artifacts. Only `PKGBUILD` (and `.SRCINFO`) are tracked.
- **UI work → N/A** — this repo has no UI/frontend surface.

## Git & GitHub

- **Commits and branches OK** — create commits and new branches whenever it makes sense, without asking first.
- **Never push** *(default)* — no `git push` under any circumstance, and absolutely never `git push --force` / `--force-with-lease`. Leave pushing to the user. **Exception:** when **"modo desatendido"** is active, you may push the feature branches you create (never `main`/protected branches, never force) so PRs are ready for review.
- **Never merge — no permission** — you do NOT have permission to merge anything into any branch, nor to merge any pull request. No `git merge`, no fast-forward integration, no `gh pr merge`. Leave every merge (branches and PRs alike) to the user. This holds in every mode, **including "modo desatendido"**.
- **GitHub via `gh`** — if the `gh` CLI is available, you may open pull requests, issues, and similar (comments, labels, etc.). These don't require pushing on your part beyond what `gh` itself does for an already-pushed branch.
- **Every PR must include a manual test plan** — when opening a PR, add a **How to test manually** section with the exact steps to exercise the change by hand. Here that means: run `makepkg -si` (or `makepkg -f`) so it builds and installs cleanly, run `namcap` on the PKGBUILD and package, then verify the font renders with Nerd icons — e.g. `fc-list | grep -i atkinson` and open a patched TTF in a viewer / terminal to confirm the injected glyphs (icons, Powerline) show. Note any deps to install first (`font-patcher`, `fontforge`).
