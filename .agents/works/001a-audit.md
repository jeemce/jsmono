# Audit Results: Vue3 Packages (pkgs/vue-*)

> Audit Date: 2026-08-06
> Criteria: Simplicity, Consistency, Reusability, Interoperability, Overridability
> Principles: JSON/JS configurability, ES Modules, SFC named "widget", NPM + JSR distribution

---

## Score Summary (out of 10)

| Package        | Simplicity | Consistency | Reusability | Interop. | Overridability | **Total** |
|----------------|:----------:|:-----------:|:-----------:|:--------:|:--------------:|:---------:|
| vue-ckeditor   |    7.0     |    7.0      |    8.0      |   8.0    |      9.0       | **7.8**   |
| vue-datatables |    8.0     |    2.0      |    1.0      |   1.0    |      0.0       | **1.6**   |
| vue-elfinder   |    7.0     |    5.0      |    3.0      |   3.0    |      4.0       | **4.4**   |
| vue-inputmask  |    7.0     |    5.0      |    6.0      |   5.0    |      7.0       | **6.0**   |
| vue-jui        |    6.0     |    6.0      |    6.0      |   6.0    |      6.0       | **6.0**   |
| vue-leaflet    |    4.0     |    5.0      |    5.0      |   5.0    |      6.0       | **5.0**   |
| vue-select2    |    7.5     |    7.0      |    5.5      |   6.5    |      8.0       | **6.9**   |

---

# 1. vue-ckeditor (7.8/10) — PALING MATANG

## Overview
Wrapper untuk CKEditor 4 dan CKEditor 5. Dua widget terpisah (v4.vue dan v5.vue), konfigurasi via JSON.

## Strengths
- Dokumentasi terbaik di antara semua packages (docs/specification, structure, testing, api, usage)
- Struktur v4 dan v5 terpisah jelas, arsitektur tepat
- CKEditorLoader singleton pattern mencegah duplicate script tags
- v5 plugin resolution cerdas — terima string name (auto-resolve) atau plugin class
- v-model support dengan watch yang mencegah emit -> watch -> emit cycle
- defineExpose untuk programmatic access (getInstance, getData, setData, focus)
- ES modules throughout, dual publish NPM + JSR

## Weaknesses & Issues (with file:line)

### Critical
- **(P2)** "ckeditor5": "*" di package.json:31 — terlalu longgar, pin version range atau peerDependencies
- **(P2)** docs/ masuk files array package.json:18 — membengkakkan package size
- **(P2)** v5 widget tidak ada SSR guard (typeof window check), beda dengan v4:vue-ckeditor:widgets/v4.vue:19

### Medium
- main/module field package.json:13-14 menyesatkan (v4 di main, v5 di module)
- v4 CDN URL hardcoded vue-ckeditor:widgets/v4.vue:13 — silent fail jika CDN berubah
- v4 editable element access 3 fallback + empty catches vue-ckeditor:widgets/v4.vue:108-115
- v5 EditorClass.create() tanpa error handling vue-ckeditor:widgets/v5.vue:21-22 — unhandled promise rejection
- JSON v4.json 282 lines: inline smiley descriptions 35-items, special characters 70 items, font lists — duplikasi default CKEditor
- .gitignore hanya ignore node_modules/

### Minor
- Tidak ada TypeScript types .d.ts

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 7 | Dua komponen kecil (~160 dan ~130 lines). TAPI 3-fallback editable access + JSON besar tambah komplexitas |
| Consistency | 7 | v4/v5 struktur identik. TAPI events beda (change vs change:data), main/module field convention mismatch |
| Reusability | 8 |Separated v4.vue/v5.vue, clear props/API, support custom config, custom editor class (v5), custom CDN (v4) |
| Interop | 8 | Great v-model, exposed methods, dual NPM/JSR, plugin resolution v5 terima string dan class |
| Overridability | 9 | Excellent. Setiap default di JSON files. config prop shallow-merge. v4 custom scriptUrl. v5 custom editor/licenseKey |

---

# 2. vue-datatables (1.6/10) — BELUM ADA SOURCE CODE

## Overview
Package ini HANYA daftar dependency. Tidak ada widget, plugin config, test, atau docs. Status: **Pre-package state** (5% jalan).

## Weaknesses & Issues

### Critical — Everything Missing
- TIDAK ADA widgets/ — nol SFC component
- TIDAK ADA plugins/ — nol JSON config
- TIDAK ADA tests/, docs/, LICENSE, README.md
- package.json TIDAK ADA: name, version, type: module, exports, main, module, browser
- package.json TIDAK ADA: peerDependencies (vue tidak di-declare), scripts, publishConfig
- package.json jquery/bootstrap/datatables.net-bs5 sebagai direct deps — harusnya peerDependencies
- Filename PACKAGE.md — harusnya README.md (will be ignored by NPM/GitHub docs rendering)
- TIDAK ADA jsr.json — JSR distribution tidak mungkin

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 8 | Nol kode = nol kompleksitas. TAPI artificial karena package body missing |
| Consistency | 2 | package.json structure tidak konsisten sibling packages. Wrong filename PACKAGE.md |
| Reusability | 1 | NUL — tidak ada source files, exports, entry points |
| Interop | 1 | TIDAK ADA peerDependencies, TIDAK ADA exports. Tidak compatible sama |
| Overridability | 0 | TIDAK ADA plugins directory, TIDAK ADA config files. Users tidak bisa customize |

---

# 3. vue-elfinder (4.4/10) — PERLU REWRITE BESAR

## Overview
Wrapper untuk jQuery/elFinder file manager. Satu komponen Vue ElFinder.

## Strengths
- API sederhana: 4 props + 1 event
- ES modules throughout, NPM distribution
- Test infrastructure (Jest + Cypress + Playwright)
- Demo app included di tests/client/

## Weaknesses & Issues (with file:line)

### Critical
- **(P2)** SFC bernama ElFinder, BUKAN Widget — principle violation
- **(P2)** Tidak ada JSR entry — tidak ada exports/jsr.json
- **(P3)** jQuery global dependency: `window.$` check components/widgets/ElFinder.vue:43-49 — consumer manual load jQuery + elFinder CDN, zero bundler integration

### Medium
- **(P3)** Tidak ada unmounted() hook — memory leak jika component destroyed
- **(P3)** Vite config hanya demo app, bukan library build (tidak ada build.lib entry)
- **(P3)** elFinder CSS tidak didistribusikan — consumer harus add CSS dependency sendiri
- **(P3)** plugins/default.json adalah DEMO CONFIG — tidak dipakai komponen (file mati dikirim ke npm)
- **(P3)** options prop dispread ke base config tanpa whitelist — user typo silent overwrite
- **(P3)** Hardcoded theme "smoothness" dan lang "en" — difficult override tanpa full config
- **(P3)** Test coverage minimal — 1 test cek wrapper.exists() saja
- **(P4)** 3 test frameworks (Jest, Cypress, Playwright) — terlalu banyak untuk 1 komponen
- TypeScript support nol — tidak ada types, components pakai Options API bukan Composition API

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 7 | Komponen 52 lines simpel. TAPI project structure kompleks (3 test frameworks, Vite demo config, 3 server setups) |
| Consistency | 5 | Mixed concerns: test configs di root, plugins/ folder unused, mixed English/Indonesian in docs |
| Reusability | 3 | Perlu jQuery global, manual CSS, tidak ada lifecycle cleanup, tidak bisa destroy/reinitialize safe |
| Interop | 3 | Global window.$ — bundlers tidak bisa resolve. No SSR/SSG. Nol types. Options API |
| Overridability | 4 | Limited props, tanpa swap jQuery init, tanpa lifecycle hooks beyond init, options spread risk |

---

# 4. vue-inputmask (6.0/10)

## Overview
Wrapper inputmask library dengan SFC + JSON config. CI/CD workflow sudah ada.

## Strengths
- SFC `<script setup>` Composition API (modern Vue 3 standard)
- JSON config plugins/default.json — shallow merge
- defineExpose public API (getUnmaskedValue, setValue, clear, isValid, removeMask, reinitialize)
- CI/CD pipeline (ci.yml, publish.yml): multi-node-version matrix, NPM+JSR auto publish

## Weaknesses & Issues (with file:line)

### Critical
- **(P1)** "types": null package.json:8 — zero TypeScript types (docs claim "TypeScript Ready" tapi tidak typed)

### Medium
- **(P1)** props.options crash jika null — widgets/inputmask.vue:54 tidak guard null/undefined
- **(P2)** Duplicate event emission: maskComplete DAN oncomplete line 132-136 — redundant event surface
- **(P3)** .json import widgets/inputmask.vue:4 non-portable across bundlers (esbuild/vanilla TS no JSON import)
- **(P3)** Shallow merge saja — nested options (alias/definitions) di-replace bukan di-merge
- **(P3)** Accessibility gap — tanpa id, aria-label, name (tidak bisa label association)
- **(P4)** Watch deep [mask,options] reinitialize berlebihan — any property ganti = destroy+recreate
- **(P4)** Test mocks duplikat 4 test files
- **(P4)** .gitignore terlalu minimal — test-results/ committed
- **(P3)** CHANGELOG.md di-referensi di docs/PUBLISHING.md tapi file tidak ada

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 7 | BuildConfig pattern straightforward. TAPI lack of TS types adds cognitive overhead |
| Consistency | 5 | Test mocks duplikat 4 files, event naming campur, e2e vitest (bukan browser), example.test.js placeholder |
| Reusability | 6 | Boleh via v-model + exposed API. TAPI missing accessibility, tanpa name/id prop |
| Interop | 5 | Full Inputmask API tidak ter-expose (onBeforeMask, onBeforePaste, preProcess, onUnMask tidak documented) |
| Overridability | 7 | JSON defaults via options prop, mask/placeholder override, defineExpose. TAPI tanpa theme/style slot |

---

# 5. vue-jui (6.0/10) — SETENGAH SELESAI

## Overview
Wrapper jQuery UI widgets. 12 widget SFC (accordion, autocomplete, button, datepicker, dialog, progressbar, slider, spinner, tabs, tooltip).

## Strengths
- Pattern konsisten: init/destroy/watch di semua widget
- v-model: datepicker, dialog, slider, progressbar, spinner
- Event mapping: jQuery UI events -> Vue emits (kebab-case)
- defineExpose: getInstance, destroy, widget-specific methods via refs
- Vue plugin installer: createVueJui (app.use(vueJui))
- Library build config: ESM + CJS dengan externals
- JSON plugin configs (accordion, datepicker, dialog, progressbar, slider, spinner, tabs)

## Weaknesses & Issues (with file:line)

### Critical
- **(P1)** jui_menu.vue + jui_selectmenu.vue exported index.js:18-20 tapi TIDAK ADA di disk — broken exports! README 12 widgets tapi hanya 10 exist
- **(P2)** Tidak ada JSR config (tidak ada jsr.json / publish field)
- **(P3)** README klaim "TypeScript definitions included" tapi tidak ada .d.ts — documentation bug

### Medium
- **(P3)** JuiWidget base (jui_widget.vue 234 lines) adalah DEAD CODE — tidak dipakai widget manapun
- **(P4)** window.jQuery check di setiap widget (jui_widget.vue:91-92) — gagal SSR, gagal ESM import
- **(P4)** Stale closures: inline callbacks capture props at init time, old callbacks remain attached
- **(P4)** Wrong CSS .jui-button di jui_autocomplete.vue:239 — copy-paste error (autocomplete = input)
- **(P4)** Duplicate content prop di jui_tooltip.vue:18-31 — Vue ignore pertama, kedua overrides
- **(P4)** Browser tests: waitForTimeout 500ms (flaky). Harus waitForSelector/toBeVisible
- **(P4)** Slider min/max/step watch destroy+rebuild — inconsistent, state loss
- **(P4)** Tidak ada watchers untuk reactive props (dateFormat, defaultDate, minDate, maxDate)

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 6 | Pattern sama, simple tapi excessive repetition. Base JuiWidget dead code |
| Consistency | 6 | Pattern consistent, wrong CSS autocomplete, duplicate prop tooltip, unused imports |
| Reusability | 6 | JSON configs reusable, individual imports OK. TAPI loose coupling, hardcoded jQuery |
| Interop | 6 | ESM, peer deps OK. TAPI glob exports tidak supported bundlers, window.jQuery no SSR |
| Overridability | 6 | Props/JSON allow overrides. TAPI init lifecycle locked, no hooks/middleware |

---

# 6. vue-leaflet (5.0/10)

## Overview
Wrapper Leaflet mapping library. 12 components: main map + 11 renderless (Marker, Circle, Polygon, Polyline, Rectangle, TileLayer, WMS, GeoJSON, LayerGroup, FeatureGroup, Popup, CircleMarker).

## Strengths
- useLeaflet() composable — reusable via provide/inject tree
- Expose: getMap(), getTileLayer(), setTileUrl()
- layers prop: array configs auto-create Leaflet objects
- Test infrastructure: unit, integration, browser

## Weaknesses & Issues (with file:line)

### Critical
- **(P1)** `watch` TIDAK DI-IMPORT di plugins/useLeaflet.js:1 — dipakai di line 25 (inject/computed/readonly only). TypeError di runtime whenReady()

### Medium
- **(P3)** widgets/leaflet.vue 516 lines — terlalu banyak tanggung jawab (init, tiles, CRUD, controls, events, styles, watching)
- **(P3)** Switch 14-branch widgets/leaflet.vue:240-280 — viola Open/Closed Principle
- **(P4)** CSS import global widgets/leaflet.vue:19 — CSS bleed semua consumer
- **(P4)** 360-line mock tests/unit/setup.js — menandakan terlalu banyak direct L.* calls di main component
- **(P4)** Layer watcher (widgets/leaflet.vue:370-383) remove ALL layers — no diffing, flash + event spam
- **(P4)** Popup binding logic DUPLICATE di semua child component

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 4 | Renderless pattern bersih. TAPI 516-line component, 14-branch switch, 360-line mock |
| Consistency | 5 | Pola renderless konsisten. TAPI naming (leaflet.vue vs "Widget" di docs), JSON.stringify array equality LPopup.vue:117 |
| Reusability | 5 | useLeaflet() OK. TAPI v-if="false" hack, OSM hardcoded, tidak ada shared composable popup/tooltip |
| Interop | 5 | Provide/inject good. TAPI CSS leak, nol types, event naming mixed (map-ready vs map-mounted) |
| Overridability | 6 | Config prop ada. Lifecycle callbacks ada. TAPI defaults tidak bisa selectively disable (zoomControl selalu true) |

---

# 7. vue-select2 (6.9/10)

## Overview
Wrapper Select2 (jQuery-based) dengan SFC. Helper utilities plugins/select2.js, 5 tema CSS (default, classic, bootstrap3-5).

## Strengths
- 4-Layer Config Merge: defaults.json -> global defaults -> plugin prop -> theme prop -> option data (plugins/select2.js:115-131)
- Pure/helper functions: cloneValue, mergeObject, normalizeValue, resolveThemeName, setGlobalDefault
- Event namespaceing .vue-select2 (widgets/select2.vue:149-179) — no cross-contamination
- Safe value sync: isSyncingValue flag (widgets/select2.vue:199-214)
- Subpath exports OK: widgets/select2.vue, plugins/default, themes/*.less
- name: "widget" correct

## Weaknesses & Issues (with file:line)

### Medium
- **(P2)** jquery + select2 sebagai dependencies bukan peerDependencies (package.json:42-48) — duplikasi per consumer
- **(P3)** $.fn.select2.defaults.reset() plugins/select2.js:110 — method tidak ada di Select2, akan throw
- **(P3)** JSON.stringify equality order-sensitive plugins/select2.js:62 — order beda = sync gagal
- **(P4)** themes/classic.less duplikat themes/default.less (both @import select2/dist/css/select2.css)
- **(P4)** CSS default theme auto-import di component widgets/select2.vue:12 — selalu load
- **(P4)** Dokumentasi language mixing (Indonesian + English)
- **(P4)** tests/ masuk files array — test code ter-publish ke consumer
- **(P4)** Empty .env file

## Scores Detail

| Kriteri | Score | Alasan |
|---------|-------|--------|
| Simplicity | 7.5 | Config merge clean, component straightforward. TAPI jQuery adds implicit complexity |
| Consistency | 7.0 | Internal consistent. TAPI language mixing, redundant classic.less |
| Reusability | 5.5 | Helpers reusable. TAPI jQuery dependency tied |
| Interop | 6.5 | Vue v-model/expose OK. Exports structured. TAPI jQuery constraint |
| Overridability | 8.0 | Excellent 4-layer config system. Defaults override via multiple layers |

---

## Cross-Package Common Issues

| Issue                    | vue-ck | vue-dt | vue-ef | vue-im | vue-jui | vue-lf | vue-sel |
|--------------------------|--------|--------|--------|--------|---------|--------|---------|
| No TypeScript types      | YES    | YES    | YES    | YES    | YES     | YES    | YES     |
| No JSR config            | OK     | YES    | YES    | OK     | YES     | YES    | OK      |
| jQuery global dep        | v4 only| N/A    | YES    | NO     | YES     | NO     | YES      |
| name: widget             | N/A    | N/A    | NO     | N/A    | N/A     | N/A    | YES     |
| Incomplete .gitignore    | YES    | N/A    | YES    | YES    | OK      | YES    | YES     |

---

## Priority Action Items

### P0 — Critical: fix immediately

| #  | Package        | Action |
|----|----------------|--------|
| 01 | vue-leaflet    | Fix `watch` import bug — `ReferenceError` di runtime |
| 02 | vue-jui        | Buat atau hapus eksport jui_menu.vue dan jui_selectmenu.vue |
| 03 | vue-datatables | Mulai dari template vue-inputmask atau remove dari repo |

### P1 — Semua packages wajib punya:

| #  | Action |
|----|--------|
| 04 | TypeScript types — `.d.ts` untuk props, emits, exposed methods |
| 05 | JSR config — `jsr.json` atau `publish` field di package.json |
| 06 | `.gitignore` lengkap — `test-results/`, `playwright-report/`, `*.log` |

### P2 — Major per-package fixes:

| #  | Package        | Action |
|----|----------------|--------|
| 07 | vue-ckeditor   | Pin ckeditor5 dependency; remove docs/ dari files; add SSR guard v5 |
| 08 | vue-elfinder   | Rewrite: eliminasi jQuery global, tambah unmounted(), composition API |
| 09 | vue-inputmask  | Guard null options prop; dedup events; add accessibility attrs |
| 10 | vue-jui        | Refactor JuiWidget dead code; fix stale closures; fix CSS/prop bugs |
| 11 | vue-leaflet    | Split 516-line component; fix CSS leak; layer diffing; fix naming |
| 12 | vue-select2    | jquery/select2 -> peerDeps; fix reset() call; remove tests from files |

### P3 — Standards improvements:

| #  | Action |
|----|--------|
| 13 | Standarize event naming (camelCase semua package) |
| 14 | Pindahkan jQuery deps ke peerDependencies |
| 15 | Hapus non-essential dari NPM publish (docs/, tests/) |
| 16 | Normalisasi semua SFC: `name: "Widget"` |
| 17 | Tambah accessibility attrs (id, aria-label, name) |
| 18 | Pilih dokumentasi language: English untuk public packages |

---

> End of audit
