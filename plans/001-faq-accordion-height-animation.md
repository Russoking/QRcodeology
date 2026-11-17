# 001 — Animate the FAQ accordion open/close instead of snapping

- **Status**: TODO
- **Commit**: 9b65582
- **Severity**: MEDIUM
- **Category**: Missed opportunities / Performance (documented exception: measured height)
- **Estimated scope**: 17 files, mechanical/uniform pattern, ~68 individual `<details>` blocks (4 per file)

## Problem

Every FAQ accordion on the site uses a native `<details>`/`<summary>` with zero
transition. Opening or closing one of the 68 FAQ items snaps instantly — the
content teleports in/out and the `+`/`–` indicator swaps character with no
motion. This is a textbook "state change that teleports" (AUDIT.md §8) with a
clean, well-supported fix (AUDIT.md §5 lists "measured height (accordions)"
as one of only two documented exceptions to the transform/opacity-only rule).

Affected, byte-identical in structure, one `.faq`/`.faq2` container with
exactly 4 `<details>` blocks each (verified — every block is exactly one
`<summary>` + one `<p>`, no exceptions):

```
index.html                                        (.faq2 — the only file using this class)
no-expiration/index.html                          (.faq)
logo-watermark/index.html                         (.faq)
wifi/index.html                                   (.faq)
bulk/index.html                                   (.faq)
scanner/index.html                                (.faq)
size-calculator/index.html                        (.faq)
blog/static-vs-dynamic-qr-codes/index.html         (.faq)
blog/qr-codes-for-restaurant-menus/index.html      (.faq)
es/no-expiration/index.html                       (.faq)
es/logo-watermark/index.html                      (.faq)
es/wifi/index.html                                (.faq)
es/bulk/index.html                                (.faq)
es/scanner/index.html                             (.faq)
es/size-calculator/index.html                     (.faq)
es/blog/static-vs-dynamic-qr-codes/index.html      (.faq)
es/blog/qr-codes-for-restaurant-menus/index.html   (.faq)
```

Current CSS (identical on all 17 files except the selector is `.faq2` instead
of `.faq` on `index.html` only, and `summary` font-size varies 15px/15.5px
between files — leave those two differences untouched):

```css
/* e.g. no-expiration/index.html:96-102 — current */
.faq{margin-top:8px;border-top:1px solid var(--surface-line);}
.faq details{border-bottom:1px solid var(--surface-line);padding:16px 0;}
.faq summary{font-family:'Inter',sans-serif;font-weight:600;font-size:15.5px;color:var(--ink);cursor:pointer;list-style:none;display:flex;justify-content:space-between;align-items:center;}
.faq summary::-webkit-details-marker{display:none;}
.faq summary::after{content:"+";font-family:'Space Mono',monospace;color:var(--amber-ink);font-size:18px;margin-left:12px;flex-shrink:0;}
.faq details[open] summary::after{content:"–";}
.faq details p{margin:12px 0 0;color:var(--ink-dim);font-size:15px;max-width:66ch;}
```

Current markup (identical pattern, every one of the 68 `<details>` blocks
site-wide, e.g. `no-expiration/index.html:268-271`):

```html
<details>
  <summary>Does this QR code really never expire?</summary>
  <p>Correct. Because the code is static and generated client-side...</p>
</details>
```

## Target

**Markup** — wrap the existing `<p>` in a new `.faq-content` div (this is a
structural change; see Boundaries — it's the one allowed exception):

```html
<details>
  <summary>Does this QR code really never expire?</summary>
  <div class="faq-content"><p>Correct. Because the code is static and generated client-side...</p></div>
</details>
```

**CSS** — the full replacement block (shown for `.faq`; do the identical
thing for `.faq2` on `index.html`, keeping that file's own existing
`font-size:15px` on `.faq2 summary` untouched):

```css
.faq{margin-top:8px;border-top:1px solid var(--surface-line);}
.faq details{border-bottom:1px solid var(--surface-line);padding:16px 0;}
.faq summary{font-family:'Inter',sans-serif;font-weight:600;font-size:15.5px;color:var(--ink);cursor:pointer;list-style:none;display:flex;justify-content:space-between;align-items:center;}
.faq summary::-webkit-details-marker{display:none;}
.faq summary::after{content:"+";font-family:'Space Mono',monospace;color:var(--amber-ink);font-size:18px;margin-left:12px;flex-shrink:0;display:inline-block;transition:transform 220ms cubic-bezier(.16,1,.3,1);}
.faq details[open] summary::after{transform:rotate(45deg);}
.faq .faq-content{overflow:hidden;height:0;transition:height 220ms cubic-bezier(.16,1,.3,1);}
.faq .faq-content p{margin:12px 0 0;color:var(--ink-dim);font-size:15px;max-width:66ch;}
@media (prefers-reduced-motion:reduce){
  .faq summary::after{transition:none;}
  .faq .faq-content{transition:none;}
}
```

Note exactly what changed from the current block:
1. `content:"+"` / `content:"–"` swap → `content:"+"` always, plus
   `details[open] summary::after{transform:rotate(45deg);}` (a `+` rotated
   45° reads as `×`/close — no character swap needed, so it can transition).
2. `summary::after` gains `display:inline-block` (required for `transform`
   to apply reliably to a pseudo-element) and the `transition`.
3. `.faq details p` → `.faq .faq-content p` (selector follows the new
   wrapper; the margin/color/font-size/max-width values are unchanged).
4. New `.faq .faq-content` rule drives the height animation.
5. New `prefers-reduced-motion` block.

**JS** — one new script, added once per file, immediately before `</body>`:

```html
<script>
(function(){
  var items = document.querySelectorAll('.faq details, .faq2 details');
  items.forEach(function(d){
    var content = d.querySelector('.faq-content');
    var summary = d.querySelector('summary');
    if(!content || !summary) return;
    content.style.height = '0px';
    summary.addEventListener('click', function(e){
      e.preventDefault();
      if(d.hasAttribute('open')){
        content.style.height = content.scrollHeight + 'px';
        requestAnimationFrame(function(){
          requestAnimationFrame(function(){
            content.style.height = '0px';
          });
        });
        content.addEventListener('transitionend', function onEnd(ev){
          if(ev.propertyName !== 'height') return;
          content.removeEventListener('transitionend', onEnd);
          d.removeAttribute('open');
        });
      } else {
        d.setAttribute('open', '');
        content.style.height = '0px';
        requestAnimationFrame(function(){
          requestAnimationFrame(function(){
            content.style.height = content.scrollHeight + 'px';
          });
        });
        content.addEventListener('transitionend', function onEnd(ev){
          if(ev.propertyName !== 'height') return;
          content.removeEventListener('transitionend', onEnd);
          content.style.height = 'auto';
        });
      }
    });
  });
})();
</script>
```

## Repo conventions to follow

- This codebase does not use a shared CSS token file — every page carries
  its own inline `<style>` block, and the same literal cubic-bezier string
  is repeated verbatim across pages wherever custom easing is used (this is
  the established pattern here, not an oversight — do not introduce a
  `--ease-out` custom property or consolidate into a stylesheet).
- The curve to use is `cubic-bezier(.16,1,.3,1)` — **not** the generic
  `cubic-bezier(0.23, 1, 0.32, 1)` from AUDIT.md's own catalog. This repo
  already has its own strong custom ease-out in active use everywhere else
  (menus, dropdowns, card hovers, scroll-reveal) — e.g.
  `index.html:70`: `.menu-panel{...transition:opacity 0.25s cubic-bezier(.16,1,.3,1), transform 0.25s cubic-bezier(.16,1,.3,1)...}`.
  Match the existing repo convention over the generic AUDIT.md default.
- 220ms sits inside AUDIT.md's "Dropdowns, selects: 150–250ms" bucket, the
  closest documented category to a content-reveal accordion; it also matches
  this repo's existing `.menu-panel`/`.lang-panel` duration (250ms) closely
  enough to feel like the same motion language.
- End-of-body script placement: every other motion feature on this site
  (hero fade-up, scroll-reveal, hamburger/lang menu, homepage typewriter)
  lives in its own `<script>` tag directly before `</body>`. Follow that
  exact placement and self-invoking-function style — e.g.
  `index.html:451-467` (the scroll-reveal `IntersectionObserver` script) is
  the exemplar to match stylistically.

## Steps

1. In each of the 17 files listed in Problem, find the `.faq` (or `.faq2` on
   `index.html` only) CSS block and replace it with the Target CSS block
   above, substituting `.faq2` for `.faq` only on `index.html`, and
   preserving that file's own existing `summary` font-size (15px or 15.5px —
   check the file before editing, don't assume).
2. In each of those 17 files, find every `<details>...</details>` inside the
   `.faq`/`.faq2` container (exactly 4 per file) and wrap the single `<p>`
   inside each with `<div class="faq-content">...</div>`, per the Target
   markup above. Nothing else inside `<details>` changes.
3. In each of those 17 files, add the Target JS script block immediately
   before the closing `</body>` tag.
4. Do not touch any other `<script>` block already in the file — this is a
   new, separate, self-invoking script, not a merge into an existing one.

## Boundaries

- Do NOT touch `.menu-panel`, `.lang-panel`, `.card`, `.related-card`, the
  `glitchFlicker`/caret keyframes, or the hero fade-up/scroll-reveal system
  — those are separate, already-tracked findings, out of scope here.
- Do NOT change the existing scroll-reveal `IntersectionObserver` script's
  selector list (it already includes `'.faq details'` on pages other than
  `index.html`, which still matches correctly after this change since the
  `<details>` element itself is untouched — only its inner content gains a
  wrapper div). Leave that script exactly as-is.
- Do NOT add `.faq2 details` to the scroll-reveal selector list on
  `index.html` even though it's missing there — that's a pre-existing,
  separate inconsistency, not part of this plan.
- The **only** structural/markup change allowed is wrapping each FAQ
  answer's `<p>` in `<div class="faq-content">`, exactly as specified in
  Step 2. Do not restructure anything else — no new wrapper on `<summary>`,
  no changes to heading levels, no changes to the questions/answers text.
- Do NOT add a dependency, build step, or external library. Plain CSS +
  vanilla JS only, matching every other script already on these pages.
- If any of the 17 files doesn't match the "4 details, each exactly one
  summary + one p" shape described in Problem when you open it (content
  has drifted since commit `9b65582`), STOP on that file and report it
  instead of improvising a different wrap.

## Verification

- **Mechanical**: no build step exists for this site (plain static HTML).
  Open each modified file directly in a browser (or via a local static
  server) and confirm the page still renders with no console errors.
- **Feel check**: on at least `index.html` and `no-expiration/index.html`,
  open and close two different FAQ items and confirm:
  - The answer grows/shrinks smoothly over ~220ms — no instant snap, no
    flash of full-height-then-collapse on first open.
  - The `+` rotates into an `×` on open and back on close — it does not
    flicker or jump.
  - Opening a second item while a first is still mid-close does not freeze
    or desync either item (each `<details>` manages its own height
    independently).
  - In DevTools' Animations panel (or the Rendering tab's "Emulate CSS
    media feature `prefers-reduced-motion`"), set playback to 10% and
    confirm the height genuinely interpolates frame-by-frame rather than
    jumping in one step (this would indicate the double-`requestAnimationFrame`
    guard is missing or broken).
  - Toggle `prefers-reduced-motion` to `reduce` and confirm the accordion
    still opens/closes (content reachable, nothing stuck) but with no
    animated height or icon rotation — it should snap, matching the
    pre-change behavior exactly for those users.
  - Tab to a `<summary>` with the keyboard and press Enter/Space — it must
    still open/close (the `e.preventDefault()` only blocks the *native*
    instant-show behavior; the click handler fires for both mouse and
    keyboard activation of a focused `<summary>`).
- **Done when**: all 17 files have the markup wrap, the updated CSS block,
  and the new script; opening/closing any FAQ item on any of them animates
  smoothly at 220ms with the rotating indicator; reduced-motion users get
  the old instant behavior back; no console errors on any modified page.
