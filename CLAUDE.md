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
- `avatar.jpg` — 320×320 cropped headshot of Cooper, shown as the circular avatar in the
  About window's info-head (the desktop About icon stays the pixel-art silhouette).
- Both files keep all CSS in one `<style>` and all JS in one `<script>` (IIFE, `'use strict'`,
  ES5-style `var` functions).

## Hard Visual Rules — never violate these

1. **No `box-shadow` anywhere.** Depth comes from flat 1px color borders only
   (highlight `#FFFFFF` top/left, shadow `#888888` bottom/right, outer `#000000`).
2. **No `border-radius`** except the one default-button outer ring (`.dring`, 8px) and the
   circular photo avatar in the About window (`.info-head .avatar`, 50% — user-requested).
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

- **Boot sequence**: `#boot` overlay (z 10000, desktop gray) plays on every load — ~0.9s
  Happy Mac (`#icon-happymac`, 32×32 symbol) → "Welcome to Macintosh." box (double black
  pinstripe frame) with status text + chunky stepped progress bar (`#0000AA` fill, no easing)
  while 7 "extension" icons march in bottom-left (reuses existing symbols). Any mousedown,
  touchstart, or keydown skips. Skipped entirely (overlay removed immediately) under
  `prefers-reduced-motion`; on <768px it still plays, typeset smaller (12px title, 180px bar).
  The wallpaper `life.set(...)` call lives inside the boot module's
  `startWallpaper()` — it only runs when boot ends, so glider guns fire from a fresh desktop.
  Special→Restart reloads the page, which replays the boot (intentional).
- **Window system**: windows registered in `wins` (id → `{el, icon}`) from the array
  `['about','projects','resume','contact','solitaire']`; element ids are `win-<id>` and
  `dicon-<id>`. Default positions via `data-x`/`data-y`. Key functions: `openWin`, `closeWin`,
  `bringToFront`, `setActive`, `zoomRect` (the signature dashed-rectangle open/close
  animation, 120ms, skipped under `prefers-reduced-motion`), `makeDraggable` (titlebar drag,
  clamped to viewport, sets `body.win-dragging` so iframes ignore pointer events mid-drag),
  `makeResizable` (8 invisible `.rs-*` handle divs appended to every window — edges + corners,
  per-window minimums in the `MIN` map), `toggleZoom` (zoom box toggles current size ↔ ~85%
  of viewport, restore rect stashed in `dataset.prevRect`; manual resize clears it).
- **Icons**: single-click/mousedown selects (inverted label), shift-click adds/removes from
  the selection, double-click opens, drag to rearrange (4px threshold so clicks don't move
  them). **Group drag**: dragging an already-selected icon moves every selected icon as one
  rigid unit — the (dx,dy) delta is clamped once for the whole group so relative positions
  survive at the desktop edges; a fresh (unselected) icon drags alone. `iconDragMoved` flag
  keeps the trailing click event from collapsing a multi-selection after a drag, and
  `selectOnly()` is the shared "select one, drop the rest" helper. `layoutIcons()` measures
  the flex column then switches `#icons` to absolute "free" mode; untouched icons re-anchor
  to the right edge on viewport resize (`iconsMoved` flag). Icons of open windows stay
  selected. Trash has a CSS hover tooltip (live item count); double-click opens the Trash
  window. Icon events attach via `hookIconEvents` + `makeIconDraggable` so dynamic icons
  behave like the originals.
- **Mini file system** (`fsItems`: fid → `{type:'folder'|'txt', name, items, content, icon,
  winId, parent}` where `parent` is `'desk' | 'trash' | folder fid` — every item lives in
  exactly one place): File → New Folder / New Text File create items in the active folder
  window, else on the desktop (cascading from top-left, named "untitled folder N" /
  "untitled N.txt"). Double-clicking a folder opens a real Finder window (`openFolder` →
  `buildWindow` + `addFinderChrome`, win id `fld<fid>`) with a classic `.fld-status` strip
  ("N items, 42 MB available") and a flex-wrap `.ficon` grid (`.ficon`, NOT `.dicon`, so the
  marquee/desktop logic ignores in-window icons); the body renders existing children on first
  build, so items moved into a never-opened folder appear when it opens. Double-clicking a
  txt opens a SimpleText editor window (id `txt<fid>`, `contenteditable="plaintext-only"` div
  `.txt-edit`, falls back to `true`; content saved to the item on input, persists across
  close/reopen within the session — no storage).
  `buildWindow` clones the standard chrome markup, registers into `wins`/`MIN`, stores
  defaults in `data-x/y/w/h` (placeDefaults restores dynamic sizes from these), and wires via
  `wireWindow` (the extracted per-window wiring), so openWin/closeWin/zoom/drag/resize all
  work unchanged. Windows are created lazily on first open and kept in the DOM after close.
- **Moving items (drag & drop)**: `moveItem(fid, dest, x, y)` = `detachItem` + `placeItem`;
  dest is `'desk' | 'trash' | folder fid`. `dropTargetAt(x, y, excludeEls)` resolves a drop
  point: the topmost open window under it wins (Trash window → `'trash'`, `fld*` → that fid,
  any other window → `'none'` = blocked); on bare desktop, the Trash icon and closed fs
  folder dicons accept (dragged icons excluded so plain rearranging isn't hijacked); else
  `'desk'`. Desktop drags route through `dropDicons` on mouseup (fs items move; system icons
  snap back, with a "required by the system" alert if the target was the Trash; folder-into-
  itself is blocked by `inSubtree` and snaps back). `.ficon`s drag with a dashed-outline
  `.dragghost` div that follows the cursor (mouse only); dropping on the desktop lands the
  icon under the cursor. `placeItem` re-points `wins[item.winId].icon` so the zoomRect
  animation tracks the icon's current home.
- **Trash**: `trashFids` array + `updateTrash()` (swaps the dicon between `#icon-trash` /
  `#icon-trash-full`, refreshes tooltip + balloon text + window status). Double-click / File →
  Open opens a real Trash Finder window (`openTrash`, win id `trash`, same `addFinderChrome`)
  whose items can be dragged back out. Special → Empty Trash: empty → plain alert; else
  `showConfirm` ("contains N items, which use NK…", `itemSizeK` fakes sizes: txt 4K, folder
  1+children) then `emptyTrashNow` → recursive `destroyItem` (removes descendants' icons,
  windows, `wins`/`MIN` entries, and `fsItems` records for real).
- **Selection marquee**: mousedown on empty desktop deselects, then dragging ≥3px shows
  `#marquee` (translucent `rgba(0,0,0,0.15)` fill, 1px black border, z-index 18 — above
  icons, below windows) and live-toggles `.selected` on every icon whose rect intersects it.
  Suppressed when Life edit mode owns the canvas (target check fails). The ONLY sanctioned
  use of alpha translucency.
- **Menu bar**: Apple/File/Edit/View/Special/Help. Working actions: Get Info / About This
  Website (opens About), File → New Folder / New Text File (create in the active folder
  window, else the desktop — see mini file system above), Open (opens selected icons —
  folders/txt items, selected in-window `.ficon`s too; Trash opens its window), Close Window
  / Quit, Edit → Copy (text selection, else selected icon/file
  names, via `navigator.clipboard`) and Select All (resume frontmost → selects document text;
  otherwise selects all icons), View → by Icon/Name/Size/Date (reorders the icon column via
  `sortIcons`; fake sizes/dates in `ICONMETA`, Trash pinned last, ✓ via `.mchk`), Special →
  Clean Up Desktop (resets window positions/sizes and icon layout — does NOT touch the
  wallpaper), Empty Trash (confirm + real delete — see Trash above), wallpaper switching (below),
  Restart (reload), Shut Down (`#shutdown` black screen, "It is now safe to turn off your
  Macintosh.", click reloads), Help → Show/Hide Balloons (`data-bln` balloon help on icons,
  close/zoom boxes, battery, clock via one `mouseover` delegate + `#balloon` div).
  Still disabled on purpose: Control Panels, Chooser, Undo, Cut, Paste, Clear (era-correct).
  `#menubar` mousedown is `preventDefault`ed so menu clicks don't clear selections (Copy
  depends on this). `#appname` shows active window's `data-app` ("Finder" / "SimpleText" /
  "Solitaire").
- **Alert dialog**: `#alert-box` fixed centered (z 7000), `showAlert(text, '#icon-…')` swaps
  the 32×32 icon via `<use id="alert-icon">`; OK button or Enter dismisses, Escape cancels.
  `showConfirm(text, icon, onOk)` adds the `confirm` class, which reveals the Cancel button
  (only OK runs the callback). Used by Empty Trash and the system-icon Trash refusal.
- **SimpleText toolbar** (Resume window): B and I latch (`.btn.on` pressed look) and toggle
  `.st-bold`/`.st-italic` on `.resume-text` (descendant `*` override). Size ▾ opens a
  `.mdrop`-styled popup (9–24 pt, ✓ on current) that sets `--stsize` on `#win-resume`;
  every resume font-size is `calc(var(--stsize,12px) ± Npx)` so the whole document scales.
- **Game of Life wallpaper**: `<canvas id="life">` is the first child of `#desktop` — behind
  icons and windows, `pointer-events:none` except in edit mode, so desktop clicks still pass
  through to the deselect handler. All logic lives in the `life` IIFE module (exposes
  `set(name)` / `resize()`). Special-menu items carry `data-bg` =
  `blank | gun | rpent | soup | edit`; each has a `.mchk` span and `checks()` puts the ✓ on
  the active one (mode `custom` checks "Draw Your Own"). Boot default is `gun` — Gosper
  glider gun top-left plus a 180°-flipped gun bottom-right (second gun only if grid ≥ 90×50
  cells) so the glider streams collide; default falls back to `blank` under
  `prefers-reduced-motion` or <768px. Engine: 8px cells drawn 7×7 (1px gap), Uint8Array
  double buffer, toroidal wrap, 110ms tick (500ms under reduced motion). Cells darken with
  age: `#0000AA` → `#000044` (age ≥2) → `#000000` (age ≥5) — flat colors from the sanctioned
  palette. "Draw Your Own…" pauses the sim and sets `body.life-edit` (canvas gets pointer
  events + crosshair; first cell clicked decides paint vs erase for the drag) and shows
  `#life-palette` (Clear / Random / default Run ↵ button; Enter also runs). `body.life-on`
  gives icon labels solid `#C0C0C0` backplates so they stay legible over cells (selected
  labels stay inverted). Viewport resize preserves overlapping cells; re-picking Random
  Soup re-randomizes.
- **Contact** is a fixed centered dialog (class `no-drag`), pulsing default button.
- **Mobile (<768px)**: the desktop becomes a scrolling page (body `overflow:auto`,
  `#desktop` static). A static icon strip sits up top — tapping an icon `scrollIntoView`s
  its window (`scroll-margin-top` clears the fixed menu bar); the Solitaire icon and window
  are hidden (the game needs a mouse). About / Projects / Resume / Contact stack as static
  full-width cards (`position:static !important`, `height:auto`, max-width 520px) all
  dressed as active (pinstripe titlebars, inverted titles), with close/zoom boxes, grow
  box, and resize handles hidden. Menu bar keeps the Apple logo (decorative,
  `pointer-events:none`), "Finder", battery, and clock; dropdown menus are hidden.
  Life wallpaper stays off. JS guard: `isMobile()` (`matchMedia(max-width:767px)`, checked
  live at event time) makes window-drag and icon-drag handlers stand down so touch
  scrolling isn't hijacked, and reroutes icon clicks to the scroll-to behavior.
  Rotating past 768px restores the full desktop (drags re-enable automatically).

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
