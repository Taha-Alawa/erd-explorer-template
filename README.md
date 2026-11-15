# ERD Explorer Template · مستكشف مخطط قاعدة البيانات

<div dir="rtl">

قالب HTML واحد يعرض مخطط قاعدة البيانات (ERD) بشكل تفاعلي وبالعربي: الجداول الجديدة والمعدّلة، وخطوط العلاقات، ووضع التركيز، وشريط جانبي فيه الشرح.

جرّبه مباشرة: افتح [`index.html`](index.html) في المتصفح.

</div>

![معاينة](docs/preview.png)

<div dir="rtl">

## البرومبت

انسخ البرومبت بزر النسخ اللي فوق المربع، وعبّئ قسم **INPUT** بمخططك، وأعطه لأي نموذج لغوي. لو تركت INPUT فاضي يطلع لك مخطط تجريبي.

</div>

````text
Build a single self-contained HTML page: an interactive database schema (ERD) explorer with Figma-like canvas navigation. No frameworks, vanilla JS, all CSS/JS inline. UI language: Arabic (RTL) for all labels and explanations; table/field names stay in English (LTR, monospace).

## LAYOUT
- Fixed top header (92px) on the left area, fixed right sidebar (380px, white, RTL, scrollable), and a canvas viewport filling the rest (overflow hidden — NO scrollbars; navigation is by panning).
- Header: title + one-line subtitle (e.g. "drag to move · scroll to zoom · click a table to pin"), a color legend for table groups (small 10px squares), a legend chip "+ field" = new field in old table, and zoom controls on the left: − / current % / + / fit.
- Initial layout: tables arranged in columns (column index set per table), 320px wide cards, 90px column gap, 50px vertical gap.
- Mobile (<900px): sidebar becomes a bottom sheet (46vh) that slides up when a table is focused.

## NAVIGATION (important)
- The canvas is a "world" div transformed with `translate(x,y) scale(s)` (transform-origin 0 0). Keep {x, y, scale} in state; apply with one function.
- Pan: press and drag on empty canvas space to move the whole view (cursor: grab → grabbing). Use Pointer Events + setPointerCapture so it works with mouse, touch and pen. Space + drag also pans even over cards.
- Zoom: mouse wheel / trackpad pinch (ctrlKey wheel) zooms toward the cursor position — the point under the mouse stays fixed. Smooth exponential step (scale *= exp(-deltaY * 0.0015)), range 0.25–2. Plain two-finger trackpad scroll (no ctrl) pans instead. Touch: two-finger pinch zooms around the midpoint. Call preventDefault on wheel (passive:false).
- Zoom buttons zoom around the viewport center; the % label updates live. "Fit" animates to show all tables with padding. Keyboard: + / − zoom, 0 = fit, arrow keys pan.
- Drag & drop tables: press on a card header and drag to move that table freely on the canvas (move in world coordinates, i.e. divide pointer delta by scale). Relationship lines redraw live while dragging (requestAnimationFrame). Dragged card gets a lifted shadow and slight scale (1.02). Optional: save positions in localStorage (wrapped in try/catch) with a "reset layout" button.
- Distinguish click from drag: only treat as a click if the pointer moved < 4px. A drag must never pin/unpin a table.
- Double-click a table → animate the view to center and zoom on it.
- Optional minimap (bottom-left, ~180px): shows all cards as small rects + the current viewport rect; dragging the rect pans the view.

## VISUAL STYLE
- Fonts: "IBM Plex Sans Arabic" for text, "IBM Plex Mono" for names/types (Google Fonts).
- Calm, neutral palette: canvas #f6f5f2 with a subtle dot grid that moves/scales with pan & zoom (background-position/size tied to state), ink #1d1c1a, muted #6b6862, borders #dedbd4 / #efede8. Group colors in oklch with similar lightness/chroma (blue, green, amber, purple, red) + grey #c9c5bd for existing tables.
- Card: white, 8px radius, 1px border shadow, 4px colored top bar (group color), header 46px (cursor: move) with English name (mono 600) + tag (NEW green / MODIFIED amber / EXISTING grey) on one side and Arabic name (muted, ellipsis) on the other.
- Each field = 28px row: key column (PK/FK/UQ, tiny mono), field name (mono 13px), short Arabic note right-aligned (muted, ellipsis). Full "name · type · note" in the row's title tooltip.
- New fields inside an existing table: light green background, 3px green left bar, green text, "+" in the key column if it has no key.
- Card footer (30px, #faf9f7): "createdAt · updatedAt" (+ "· status" for new tables).
- Existing/unchanged tables look subdued (greyer header name). Disable text selection while dragging.

## RELATIONSHIP LINES
- One SVG layer inside the world div behind the cards; recompute paths from current card positions (function redrawLines()), so they follow dragged tables.
- Each relation goes from the FK row of the child table to the first row (PK) of the parent. Smooth cubic bezier curves: pick the facing edges based on current x positions (child left of parent → child right edge to parent left edge, and vice versa); if horizontally overlapping, loop out to the left (~34px). Control-point offset = max(60, 45% of horizontal distance).
- Marker: small filled circle at the FK end, short vertical bar at the PK end.
- Relations from new tables/new fields are darker (#6f6a61); old ones lighter (#b3aea4). Optional dashed style (e.g. audit relations). Stroke width stays visually constant (vector-effect: non-scaling-stroke).

## INTERACTION
- Hover a card → focus mode: other cards fade to 0.28 opacity, related tables stay full, current card gets a 2px dark ring + shadow; all lines fade to 0.08 except the ones connected to it (dark, 2px). Sidebar shows its details. (No hover-focus changes while panning or dragging.)
- Click a card → pins it (📌 after its name). Click again, click empty canvas (without dragging), press Esc, or the ✕ in the sidebar → unpin and reset.
- Empty sidebar state: eyebrow (project name), heading "hover or click any table", a short hint paragraph including navigation tips, a 2×2 stats grid (new tables, modified tables, new fields, new enums — computed from data), a "the whole idea in two lines" section with inline code chips, and an amber "important rule" box.
- Table sidebar: close button, eyebrow with group color dot + status ("new table · group" / "existing · fields added" / "existing · unchanged"), English name (mono 22px), Arabic name, a one-line TL;DR, a grey "example" box with a real-world example, then sections:
  - "what was added?" (green code chips) for modified tables
  - "key fields" list: code chip + Arabic explanation
  - "states · EnumName": pill chips "Arabic label  Key=1,2,3…"
  - "related to": clickable chips (table name + relation type 1:1 / 1:N / M:N); clicking one pins that table and smoothly animates the view to center on it
  - "important rule" amber box (right border 3px)

## DATA MODEL (keep all data in JS objects so the template is reusable)
```js
const G = { groupKey: { c: 'oklch(...)', ar: 'Arabic group name' }, ..., old: {c:'#c9c5bd', ar:'existing table'} };
const f = (name, type, key='', note='', isNew=false) => ({...});
const T = {
  TableName: {
    col: 0, group: 'groupKey', isNew: true, mod: false, ar: 'Arabic name',
    fields: [ f('Id','bigint','PK'), f('X','int','FK','note') ],
    tldr: '...', ex: '...',
    keys: [['Field','explanation']], newKeys: [['Field','explanation']],
    enumName: 'EnumName', enums: [['Key','Arabic label']],
    rule: '...'
  }
};
const R = [ ['ChildTable','ChildField','ParentTable','1:N', /*dashed*/ false] ];
```
Escape all text before inserting into HTML.

## INPUT (replace with your own schema)
- Project name / page title: ___
- Groups (name + color): ___
- Tables: for each → name, Arabic name, column, group, new/modified/existing, fields (name, type, key, note, new?), TL;DR, example, key fields, enum states, rule
- Relations: child.field → parent (type)
- Empty-state summary text: ___

If no schema is provided, generate a realistic sample schema of ~10–15 tables across 5 columns so the template is fully demonstrated.
````
