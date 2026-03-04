# Craft CMS – Project Conventions

A checklist of standards to follow on every Craft CMS project. Apply these after the initial setup (based on `craft-starter-kit`) and before handover.

---

## User Groups

- [ ] Update the **Content Editor** user group permissions to include all relevant Entry sections created for the project.

---

## Navigation

- [ ] Review all navigations from the starter kit — remove any that are not needed for this project.
- [ ] **Multi-site:**
  - [ ] Set propagation method to **"Only save nodes to the site they were created in"**.
  - [ ] Verify every navigation is active for each site in the project.

---

## Fields

The starter kit ships with the following shared fields. Reuse these before creating new ones:

| Handle | Name | Type |
|---|---|---|
| `image` | Image | Assets |
| `video` | Video | Assets |
| `svgIcon` | Icon | Assets (SVG) |
| `headerLogo` | Header Logo | Assets |
| `ckeditorDefault` | CKEditor – Default | CKEditor |
| `ckeditorSimple` | CKEditor – Simple | CKEditor |
| `ckeditorTitle` | CKEditor – Title | CKEditor |
| `plainTextSingleLine` | Plain Text – Single Line | Plain Text |
| `plainTextMultiLine` | Plain Text – Multi Line | Plain Text |
| `titleLevel` | Title Level | Dropdown |
| `alignment` | Alignment | Lightswitch |
| `textAlignment` | Text Alignment | Dropdown |
| `backgroundColor` | Background Color | Dropdown |
| `lightswitch` | Lightswitch | Lightswitch |
| `button` | Button – Single | Matrix (max 1, entry type: `button`) |
| `buttons` | Button – Multiple | Matrix (entry type: `button`) |
| `linkText` | Link – Text | Link (with label + target) |
| `linkNoLabel` | Link – No Label | Link (no label) |
| `linkEmail` | Link – Email | Link (email only) |
| `linkPhone` | Link – Phone | Link (tel only) |
| `linkType` | Link – Type | Dropdown |
| `address` | Address | Addresses |
| `form` | Form | Formie |
| `seo` | SEO | SEOmatic |
| `pageBuilder` | Page Builder | Matrix |

**Conventions:**
- [ ] Check whether **Matrix**, **Text**, or **Entry** fields are included as search keywords.
- [ ] **Multi-site:** Ensure Translation Method is set to **"Translate for each site"** on all translatable fields.

---

## Entry Types

### Icons

| Scenario | Icon |
|---|---|
| Page entry (has URI) | `page` |
| Reusable element / no URI | `rectangle-history` |
| Page Builder block | `cube` |
| Sub-block (nested Matrix) | `cubes` |
| Global settings | `globe` |

### General

- [ ] Review each field and mark it **required** where appropriate (e.g. `Content` and `Image` in a Content Media block).
- [ ] Reusable elements (e.g. FAQ item, Team Member) and entries without a URI format use the **`rectangle-history`** icon.

### Page entries (entries with a URI format)

- [ ] Icon: `page`.
- [ ] Tabs must follow this order:
  1. **Settings** — title field + page-specific fields
  2. **Page Builder** — contains the `pageBuilder` field (if the page uses reusable blocks)
  3. **SEO** — contains the `seo` field
- [ ] `showSlugField: true`, `showStatusField: true`.

### Page Builder Blocks

- [ ] Handle ends with `Block` — e.g. `contentMediaBlock`, `cardsBlock`.
- [ ] Icon: **`cube`**. Sub-blocks nested inside a Matrix use **`cubes`**.
- [ ] `showSlugField: false`, `showStatusField: false`, `hasTitleField: false` (use `titleFormat` for CP label).
- [ ] Decide whether the **Title** field should be required (set accordingly).
- [ ] Add an **"Anchor"** tab with a Template field pointing to `lameco/helper/anchor/cp.twig`.

### Reusable / Element entry types (no URI)

- [ ] Icon: `rectangle-history`.
- [ ] `showSlugField: false`, `showStatusField: false` (unless status control is needed).

---

## Sections

### URI format

| Section type | URI format |
|---|---|
| Home page | `__home__` |
| Single page | `{slug}` (or fixed value per language, e.g. `news` / `nieuws`) |
| Channel / detail page | `{slug}` or `overview-slug/{slug}` |
| Structure with parent | `{parent.uri ?? 'fixed-value'}/{slug}` |
| No public URL | `null` |

- [ ] A **single overview page** uses a fixed slug matching the site language (e.g. `news` / `nieuws`).
- [ ] **Detail pages** related to an overview page use: `overview-slug/{slug}` (or `{parent.uri ?? 'overview-slug'}/{slug}` for structures).
- [ ] For Structure sections, verify the **max level depth** matches the content needs.

### Multi-site

- [ ] Propagation method is set to **"Let each entry choose which sites it should be saved to"**.

---

## Entries (CP sidebar grouping)

- [ ] Entries **without** a URI format are grouped under a heading called **"Elements"**.
- [ ] Entries **with** a URI format are grouped under a heading called **"Pages"**.
- [ ] Category groups are grouped under a heading called **"Categories"**.

---

## Assets

- [ ] Add **image transforms** for all image fields used in templates.
- [ ] Add the **preferred dimensions** of an image to the field instructions (e.g. `1920 × 1080 px`).

---

## Twig Templates & Partials

### Rule: extract reusable markup into `_partials/`

Any template fragment that appears in more than one place, or that is likely to be reused across sections, **must** be extracted into `templates/_partials/`. Do not copy-paste the same markup into multiple templates.

### Directory structure

```
templates/
├── _partials/
│   ├── sprig/
│   │   ├── loader.twig        # Loading overlay for Sprig components
│   │   └── pagination.twig    # Pagination controls (prev/next + page numbers)
│   ├── formie/
│   │   └── form.twig          # Formie form wrapper
│   ├── structured-data/       # JSON-LD structured data snippets
│   └── {name}.twig            # Other reusable partials (hero, card, share, …)
├── _pages/
│   └── {section}/
│       ├── index.twig         # Overview / listing page
│       ├── entry.twig         # Detail page
│       └── _components/       # Sprig components scoped to this section
│           └── {name}.twig
```

### Common partials to always extract

| Partial | Path | When to use |
|---|---|---|
| Sprig loader | `_partials/sprig/loader.twig` | Any Sprig component that fetches data |
| Pagination | `_partials/sprig/pagination.twig` | Any paginated listing |
| Entry card | `_partials/{section}/card.twig` | Whenever a card appears in ≥2 places |
| Hero | `_partials/hero.twig` | Reusable hero/banner section |
| Share article | `_partials/share.twig` | Social share buttons on detail pages |
| Structured data | `_partials/structured-data/{type}.twig` | JSON-LD per content type |
| Formie form | `_partials/formie/form.twig` | Form render wrapper |

### Conventions

- [ ] Include the Sprig **loader partial** (`_partials/sprig/loader.twig`) inside every Sprig component instead of inlining a custom spinner.
- [ ] Include the **pagination partial** (`_partials/sprig/pagination.twig`) inside every paginated Sprig component instead of duplicating the pagination markup.
- [ ] **Cards** for a section (e.g. news card, event card) live in `_partials/{section}/card.twig` and are included from both the Sprig component and any other template that needs to render the same card.
- [ ] Pass all required data to partials via `include` variables — partials must not assume a global `entry` variable unless explicitly documented.
- [ ] Document the expected variables at the top of each partial with a comment block:
  ```twig
  {#
   # Pagination Partial
   # Variables:
   #   page        – current page (int)
   #   totalPages  – total number of pages (int)
   #   category    – active filter value (optional)
   #}
  ```

---



- [ ] Filtering and pagination is implemented using the **[Sprig](https://putyourlightson.com/plugins/sprig)** plugin (`putyourlightson/craft-sprig`).
- [ ] The Sprig component lives in `templates/_pages/{section}/_components/{name}.twig`.
- [ ] Filter state and current page are reflected in the URL via `s-push-url` so links are shareable and browser Back works correctly.
- [ ] Initial query params (`?category=`, `?page=`) are read from the request and passed to the `sprig()` function call so shared URLs restore the correct state on page load.

---

## Translations

- [ ] **No hard-coded strings** in templates — all visible text goes through `| t('site')` or the appropriate translation category.
- [ ] Use **descriptive dot-notation keys**, e.g.:
  - `blog.readMore`
  - `search.placeholder`
  - `nav.skipToContent`
  - `form.submit`
- [ ] Translation files live in `translations/{locale}/{category}.php`.

