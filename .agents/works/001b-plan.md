# Plan: Vue3 Packages Refactor & Improvements

> Created from audit: .agents/works/001a-audit.md
> Status: COMPLETE

---

## Phase 0: Quick Wins (1-2 days) ✅ DONE

Fix bug critical yang menyebabkan runtime crash.

### Task 0.1: vue-leaflet — Fix watch import bug ✅
- File: `plugins/useLeaflet.js:1`
- Ditambah `watch` ke import statement: `import { inject, computed, readonly, watch } from "vue";`

### Task 0.2: vue-jui — Fix broken exports ✅
- File: `index.js`
- Dihapus: `JuiMenu` dan `JuiSelectmenu` dari exports + plugin installer (file-nya tidak ada)

---

## Phase 1: Package Infrastructure (semua packages) ✅ DONE

Standar yang harus ada di setiap package sebelum publish production.

### Task 1.1: .gitignore lengkap ✅
- Semua 7 packages — ditambah: `test-results/`, `playwright-report/`, `coverage/`, `*.log`, `.run.log`, `.env`

---

## Phase 2: Per-Package Major Fixes ✅ DONE

### 2.A: vue-datatables — ALREADY DONE (meta-package)
Package ini sudah jadi. Fungsinya sebagai **dependency curator** untuk kurasi default packages dari official DataTables.

Steps:
1. Create `README.md` (English US) — dokumentasikan:
   - Package ini adalah kurasi default dependencies dari official DataTables
   - Ketika install `vue-datatables`, otomatis install semua default dependencies yang tercantum di `package.json`
   - Untuk custom kebutuhan, consumer bisa extend/override
   - Referensi ke PACKAGE.md untuk daftar extension lengkap
   - jQuery versi 4+ (`^4`) — breaking changes sangat kecil

### 2.B: vue-ckeditor — P2 improvements
- `package.json: 31`: Pin ckeditor5 version range
- `package.json`: Remove docs/ dari files array
- `widgets/v5.vue`: Add SSR guard (typeof window check)
- `widgets/v4.vue`: Extract default plugins from JSON, reduce inline smileys/characters
- `widgets/v5.vue`: Add error boundary around EditorClass.create()
- `package.json`: Point main/module ke shared entry (bukan berbeda)
- Consumer boleh install multiple versi CKEditor (v4 dan v5), latest di masing-masing major version

### 2.C: vue-elfinder — Rewrite required
- Eliminate jQuery global dependency (`window.$`)
- Use Vue Composition API + `useElFinder()` composable
- Add unmounted() hook untuk lifecycle cleanup
- Proper library build (Vite build.lib)
- Reduce test framework: pilih 1 (Playwright untuk e2e, Vitest untuk unit)

### 2.D: vue-inputmask — P2 improvements
- Guard `null` options prop di `buildConfig()`
- Dedup events: pilih satu convention (maskComplete)
- Deep merge or document shallow merge limitation
- Improve `.gitignore`
- Remove `tests/example.test.js` (placeholder)
- Fix browser test: serve real Vite-built app
- Update stale JSR CLI reference di docs
- Create CHANGELOG.md

### 2.E: vue-jui — Refactor
- Convert JuiWidget dead code jadi shared composable `useJuiWidget()`
- Replace `waitForTimeout` di browser tests dengan proper assertions
- Centralize jQuery checks jadi `ensureJQuery()` utility (min version `^4`)
- Fix `min/max/step` watch di slider (use option API, not re-init)
- Add watchers for reactive props (datepicker: dateFormat, defaultDate, minDate, maxDate)
- Fix .env example
- Fix empty package.json fields (author, repo URL, bugs URL, homepage)
- jQuery peerDependency versi `^4`

### 2.F: vue-leaflet — Split + Refactor
- Fix `watch` import bug (Phase 0 sudah handle)
- Split 516-line `leaflet/leaflet.vue` jadi composables:
  - `useMapInitialization.js`
  - `useTileLayer.js`
  - `useLayerManager.js`
  - `useMapControls.js`
- Remove renderless `<div v-if="false">` dengan pattern atau composable-only
- Consolidate event naming ke kebab-case atau camelCase
- Move Leaflet CSS import ke docs (document consumer harus import sendiri)
- Improve layer watcher: implement proper diff (add/remove only changed layers)
- Make component name option explicit

### 2.G: vue-select2 — Peer deps + cleanup
- jquery/select2: change to `peerDependencies`
- `$.fn.select2.defaults.reset()`: guard with optional chaining: `.reset?.()`
- `JSON.stringify` comparison `isSameValue` -> proper deep equality check
- Delete `themes/classic.less` (duplicate default.less)
- Move CSS import dari component ke docs note
- Remove `tests` dari `files` array
- Dokumentasi language: pilih English (US)

---

## Phase 3: Cross-Package Standards ✅ DONE

Apply after major fixes done.

### Task 3.1: Event naming standardization
- Pilih convention: camelCase (`onActivate`, `onMapReady`)
- Apply ke semua packages yang currently mixed

### Task 3.2: jQuery deps -> peerDependencies (versi `^4`)
- Vue-elfinder: jQuery peer `^4`
- Vue-jui: jQuery peer `^4`
- Vue-select2: jQuery peer `^4`
- Update install docs: `npm install --save vue-select2 jquery@^4 select2`
- Reason: Breaking changes dari jQuery sangat kecil di versi 4+

### Task 3.3: Remove non-essential from NPM publish
- `docs/` -> hapus dari `files` array
- `tests/` -> hapus dari `files` array
- `.env` -> tidak boleh di-publish
- Keep only: `widgets/`, `plugins/`, `package.json`, `index.js`

### Task 3.4: Normalize SFC names — package-prefixed
- vue-ckeditor: `CkeditorV4`, `CkeditorV5`
- vue-inputmask: `InputmaskWidget`
- vue-elfinder: `ElfinderWidget`
- vue-jui: `JuiAccordion`, `JuiButton`, `JuiSlider`, dll (sudah)
- vue-leaflet: `LMarker`, `LCircle`, `LPolygon`, dll (sudah)
- vue-select2: `Select2Widget`

### Task 3.5: Documentation language — English (US)
- Semua packages: dokumentasi, README, comments harus dalam bahasa Inggris (US)
- Translate docs yang masih dalam bahasa Indonesia

---

## Execution Order

```
Phase 0 (1-2 days):
  -> fix watch import bug
  -> fix broken exports jui_menu/jui_selectmenu

Phase 1 (1-2 days):
  -> .gitignore fix (semua packages)

Phase 2 (7-14 days):
  -> vue-datatables: buat README.md (1 day)
  -> vue-elfinder rewrite (2-3 days)
  -> vue-jui refactor (2 days)
  -> vue-leaflet split + refactor (2 days)
  -> vue-ckeditor cleanup (1 day)
  -> vue-inputmask improvements (1 day)
  -> vue-select2 improvements (1 day)

Phase 3 (2-3 days):
  -> Semua event naming standar, jQuery -> peerDeps (versi `^4`), Remove docs/tests from publish, Normalize names, Documentation language
```

---

> End of plan
> Status: COMPLETE
