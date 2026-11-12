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
Build a single self-contained HTML page: an interactive database schema (ERD) explorer. No frameworks, vanilla JS, all CSS/JS inline. UI language: Arabic (RTL) for all labels and explanations; table/field names stay in English (LTR, monospace).

## LAYOUT
- Fixed top header (92px) on the left area, fixed right sidebar (380px, white, RTL, scrollable), and a scrollable canvas filling the rest.
- Header: title + one-line subtitle (e.g. "hover or click a table · N new tables · M modified"), a color legend for table groups (small 10px squares), a legend chip "+ field" = new field in old table, and zoom buttons (− / fit / +) on the left.
- Canvas: light warm background (#f6f5f2), tables arranged in columns (column index set per table), 320px wide cards, 90px column gap, 50px vertical gap. Canvas is scaled with CSS transform; zoom range 0.35–1.6; "fit" scales to container width (min 0.55) and refits on window resize.
- Mobile (<900px): sidebar becomes a bottom sheet (46vh) that slides up when a table is focused.

## VISUAL STYLE
- Fonts: "IBM Plex Sans Arabic" for text, "IBM Plex Mono" for names/types (Google Fonts).
- Calm, neutral palette: ink #1d1c1a, muted #6b6862, borders #dedbd4 / #efede8. Group colors in oklch with similar lightness/chroma (blue, green, amber, purple, red) + grey #c9c5bd for existing tables.
- Card: white, 8px radius, 1px border shadow, 4px colored top bar (group color), header 46px with English name (mono 600) + tag (NEW green / MODIFIED amber / EXISTING grey) on one side and Arabic name (muted, ellipsis) on the other.
- Each field = 28px row: key column (PK/FK/UQ, tiny mono), field name (mono 13px), short Arabic note right-aligned (muted, ellipsis). Full "name · type · note" in the row's title tooltip.
- New fields inside an existing table: light green background, 3px green left bar, green text, "+" in the key column if it has no key.
- Card footer (30px, #faf9f7): "createdAt · updatedAt" (+ "· status" for new tables).
- Existing/unchanged tables look subdued (greyer header name).

## RELATIONSHIP LINES
- One SVG layer behind the cards, drawn after layout. Each relation goes from the FK row of the child table to the first row (PK) of the parent.
- Smooth cubic bezier curves: if both tables are in the same column, loop out to the left (~34px); otherwise connect the facing edges with horizontal control points (offset = max(60, 45% of distance)).
- Marker: small filled circle at the FK end, short vertical bar at the PK end.
- Relations from new tables/new fields are darker (#6f6a61); old ones lighter (#b3aea4). Optional dashed style (e.g. audit relations).

## INTERACTION
- Hover a card → focus mode: all other cards fade to 0.28 opacity, related tables stay full, current card gets a 2px dark ring + shadow; all lines fade to 0.08 except the ones connected to it (dark, 2px). Sidebar shows its details.
- Click a card → pins it (📌 after its name); hover no longer changes anything. Click again, click empty canvas, or the ✕ button in the sidebar → unpin and reset.
- Empty sidebar state: eyebrow (project name), heading "hover or click any table", a short hint paragraph, a 2×2 stats grid (new tables, modified tables, new fields, new enums — computed or provided), a "the whole idea in two lines" section with inline code chips, and an amber "important rule" box.
- Table sidebar: close button, eyebrow with group color dot + status ("new table · group" / "existing · fields added" / "existing · unchanged"), English name (mono 22px), Arabic name, a one-line TL;DR (bold-ish 16px), a grey "example" box with a real-world example, then sections:
  - "what was added?" (green code chips) for modified tables
  - "key fields" list: code chip + Arabic explanation
  - "states · EnumName": pill chips "Arabic label  Key=1,2,3…"
  - "related to": clickable chips (table name + relation type 1:1 / 1:N / M:N); clicking one pins that table and smoothly scrolls it into view
  - "important rule" amber box (right border 3px)

## DATA MODEL (keep all data in two JS objects so the template is reusable)
```js
const G = { groupKey: { c: 'oklch(...)', ar: 'Arabic group name' }, ..., old: {c:'#c9c5bd', ar:'existing table'} };
const f = (name, type, key='', note='', isNew=false) => ({...});
const T = {
  TableName: {
    col: 0, group: 'groupKey', isNew: true, mod: false, ar: 'Arabic name',
    fields: [ f('Id','bigint','PK'), f('X','int','FK','note', /*isNew*/ false) ],
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
- Empty-state summary text and stats: ___

If no schema is provided, generate a realistic sample schema of ~10–15 tables across 5 columns so the template is fully demonstrated.
````
