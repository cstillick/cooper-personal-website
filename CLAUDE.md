# Cooper Stillick — Personal Website (Retro Mac OS)

Personal site for Cooper Stillick styled as a classic Macintosh desktop (System 7 /
Mac OS 8 Platinum era, ~1993). The desktop IS the page: draggable windows, pixel-art
icons, working menu bar, and a playable Solitaire app.

## Files

- `index.html` — the entire site: desktop, menu bar, icons, all windows. Embedded CSS + JS,
  no build step, no framework. Only external dependency: one Google Fonts import
  (Press Start 2P).
- `solitaire.html` — fully playable Klondike Solitaire, same visual system. Standalone page
  AND embedded app (see Architecture). Same single-file, no-dependency constraints.
- Both files keep all CSS in one `<style>` and all JS in one `<script>` (IIFE, `'use strict'`,
  ES5-style `var` functions).

## Hard Visual Rules — never violate these

1. **No `box-shadow` anywhere.** Depth comes from flat 1px color borders only
   (highlight `#FFFFFF` top/left, shadow `#888888` bottom/right, outer `#000000`).
2. **No `border-radius`** except the one default-button outer ring (`.dring`, 8px).
3. **No gradients** except 1px/2px hard-stop `repeating-linear-gradient` used as pattern
   fills (titlebar pinstripes, Solitaire card-back checkerboard). Nothing soft, ever.
4. **Pixel art only**: every icon is an inline SVG `<symbol>` on a strict integer grid
   (32×32 desktop icons, 24×24 card figures, 16×16 small icons), `shape-rendering:crispEdges`,
   `<rect>`s with whole-number coordinates only. No strokes on icons (they blur), no
   fractional coords. Susan Kare style: bold silhouettes, checkerboard dither fills via
   SVG `<pattern>`s `#d50` / `#d25` defined in the hidden defs SVG at the top of each file.
5. **No emoji in UI.** The user explicitly asked for hand-drawn pixel icons instead
   (Apple logo, folder, envelope, X logo, compact Mac, mouse are all SVG symbols).
   The ✓ in menus and ↵ on default buttons are fine (text symbols, era-correct).

## Palette / Type

- Desktop `#C0C0C0`, titlebar `#E0E0E0` (active = pinstripe), inactive titlebar `#C0C0C0`,
  shadow `#888888`, button face `#C0C0C0`, felt green `#2D6A2D`, card red `#CC0000`,
  card back `#000044`/`#0000AA`.
- Chrome font stack: `'Chicago','Charcoal','Press Start 2P',Geneva,sans-serif` at 12px;
  body text Geneva/Helvetica 11–12px; icon labels 11px; tooltip 10px.
  `-webkit-font-smoothing:none` for crispness.

## index.html architecture

- **Window system**: windows registered in `wins` (id → `{el, icon}`) from the array
  `['about','projects','resume','contact','solitaire']`; element ids are `win-<id>` and
  `dicon-<id>`. Default positions via `data-x`/`data-y`. Key functions: `openWin`, `closeWin`,
  `bringToFront`, `setActive`, `zoomRect` (the signature dashed-rectangle open/close
  animation, 120ms, skipped under `prefers-reduced-motion`), `makeDraggable` (titlebar drag,
  clamped to viewport, sets `body.win-dragging` so iframes ignore pointer events mid-drag),
  `makeResizable` (8 invisible `.rs-*` handle divs appended to every window — edges + corners,
  per-window minimums in the `MIN` map), `toggleZoom` (zoom box toggles current size ↔ ~85%
  of viewport, restore rect stashed in `dataset.prevRect`; manual resize clears it).
- **Icons**: single-click/mousedown selects (inverted label), double-click opens, drag to
  rearrange (4px threshold so clicks don't move them). `layoutIcons()` measures the flex
  column then switches `#icons` to absolute "free" mode; untouched icons re-anchor to the
  right edge on viewport resize (`iconsMoved` flag). Icons of open windows stay selected.
  Trash has a CSS hover tooltip "Trash (0 items)".
- **Menu bar**: Apple/File/Edit/View/Special/Help. Working actions: Get Info / About This
  Website (opens About), Close Window, Clean Up Desktop (resets window positions/sizes and
  icon layout), Restart (reload).
  `#appname` shows active window's `data-app` ("Finder" / "SimpleText" / "Solitaire").
- **Contact** is a fixed centered dialog (class `no-drag`), pulsing default button.
- **Mobile (<768px)**: everything hidden except the About window; menu bar shows
  "Mac OS requires a larger display" + pixel Mac icon.

## Solitaire integration (the in-page app)

- `solitaire.html` detects embedding via `window.self !== window.top` (`EMBED` flag) and adds
  `html.embed`, which hides its own desktop/titlebar/apple and goes static — its menu strip
  (Game/Options/Help) + toolbar + felt render inside the host window.
- Host side: `#win-solitaire` contains `<iframe id="solitaire-frame" data-src="solitaire.html">`,
  lazy-loaded on first `openWin('solitaire')`. Iframe fills the win-body (100%/100%); the
  window's default CSS size is 581×595, which yields exactly 577×570 of iframe inside the
  chrome, and `MIN.solitaire` pins that as the minimum so the game never clips. In embed
  mode the body goes flex-column and the felt stretches (`width/height:auto`), so enlarging
  the window just shows more green felt.
- postMessage protocol: child sends `'solitaire-quit'` (close box / Game→Quit) and
  `'solitaire-focus'` (any mousedown) → parent calls `closeWin`/`bringToFront`.
  Standalone visits still work; Quit then navigates to index.html.
- Game specifics: Draw 1/3, standard + Vegas scoring, Ctrl/Cmd+Z undo (snapshot stack,
  cap 200), auto-complete dialog, parabolic bouncing-card win animation (bounce factor 0.7,
  click to dismiss), reduced-motion fallbacks, "requires a mouse" dialog on touch-only.
- Foundations are suit-locked left→right: ♠ ♥ ♦ ♣ (matches the faint slot symbols).

## Content (filled vs placeholder)

Filled: name (Cooper Stillick), bios, Created date (Tue Jun 9, 2026), location
(Oklahoma City, OK), email `cestillick@wm.edu`, Twitter/X `Cooper80330725`, all six projects
(Corridor, Backtesting Engine, Crypto Arbitrage, Stock Screener, Fraud Detection, DNS Resolver
— descriptions sourced from the real GitHub READMEs at github.com/cstillick/<repo>).

- **Corridor has no GitHub link on purpose** (repo lvroberto1/corridor is private/404); its
  detail view hides the "View on GitHub" link via the missing `data-url`. Description was
  written from the user's local `~/corridor-app` source (agent-native API metering/pricing).
- **Still placeholder: `[RESUME CONTENT]`** in the Resume window (search `[` to find it).

## Gotchas

- Opening via `file://` may block the Solitaire iframe in some browsers — serve over HTTP
  (`python3 -m http.server`) when testing the embed.
- After editing either file, syntax-check: extract the `<script>` body and run `node --check`.
- Self-review greps before finishing any styling change: no `box-shadow`, `border-radius`
  only on `.dring`, no gradients beyond the sanctioned patterns, no fractional SVG coords
  (`grep -oE '(x|y|width|height)="[0-9]+\.[0-9]+"'` must return nothing).
