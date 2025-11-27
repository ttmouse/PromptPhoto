# Preset Management & Sidebar Enhancements

Planning notes for building a scalable preset browser/editor inside PromptPhoto. This document captures the product goals, required UX, and technical approach so we can iterate without revisiting the same questions.

## 1. Goals
- Support dozens or hundreds of structured presets that come from manual curation or AI analysis.
- Let users browse, search, and filter presets quickly, even when each preset controls a different subset of dimensions.
- Allow optional thumbnails, tags, and metadata without forcing every preset to provide them.
- Make it easy for non-developers to add/edit presets directly from the UI (including importing a JSON blob).

## 2. Feature Set
1. **Preset Sidebar**
   - Sticky left sidebar with collapse/expand.
   - Search box (name/description/tag), tag filters, sorting (updated, favorites, A-Z), list/grid view toggle.
   - Infinite scroll or pagination for large datasets; optional virtualized list.
2. **Preset Cards**
   - Optional thumbnail; fallback to gradient/initials if absent.
   - Display name, short description, mood/scene badges, “N dimensions” badge, quick actions (Apply, Preview, Favorite, Edit, Delete, Export).
   - Hover/overflow menu for additional actions.
3. **Tagging System**
   - Multi-select tags that describe scene/purpose/style (e.g., 自拍, 证件照, 影棚肖像, 私房, 广告).
   - Tag groups (Scene, Mood, Lighting, etc.) and tag cloud/filter UI.
   - Search syntax supports `tag:自拍`.
4. **Preset Detail/Preview**
   - When a card is selected, show summary panel at top of the editor: thumbnail, description, tags, dimension badges, “Apply (defined only)” vs “Apply (override all)”, “Compare/Save As”.
5. **Preset CRUD**
   - “Save as preset” from the current configuration: fills form with current `state` + `enabled` + `enabledSub`.
   - “New preset” modal: name, tags, description, optional thumbnail upload/URL, choose which dimensions stay enabled (default to current toggles).
   - Import/export JSON (single or batch); bulk delete / favorite.
6. **Optional Thumbnails**
   - Field stored per preset but not required. Can be uploaded (convert to Base64) or set from URL. If empty, UI uses generated gradient.

## 3. Data Model
```ts
type Preset = {
  id: string;
  name: string;
  descriptions: { zh: string; en: string };
  state: Record<string, string>;
  enabled: Record<GroupKey, boolean>;
  enabledSub: Record<SubKey, boolean>;
  tags: string[];
  thumbnail?: string;      // optional data URL or remote URL
  favorite?: boolean;
  updatedAt: number;
};
```
- We keep the global “dimension universe” unchanged; every preset picks its subset via `enabled`/`enabledSub`.
- When applying a preset, only the `true` entries are copied; provide UI toggle to override all dimensions if needed.
- Future-proof by allowing additional metadata (`author`, `source`, etc.).

## 4. UX/Interaction Flow
1. Sidebar loads preset list from store; search & filters update the displayed subset instantly.
2. Selecting a preset:
   - Highlights card, shows detail summary, buttons to apply (defined/all), compare with current config, copy JSON.
3. Creating or editing:
   - Modal shows fields for name/descriptions, tags multiselect, thumbnail picker, checkboxes for enabled groups/sub-groups.
   - “Use current state” button pre-fills from live configuration.
4. Import/export:
   - Drag JSON file or paste text; validate schema, show preview, allow selective import.
   - Export selected presets to JSON for sharing/backups.
5. Tag management:
   - Provide tag catalog UI (optional) to maintain categories/color coding.
   - Cards show up to 3 tags; rest behind “+N”.

## 5. Technical Architecture
- **Data Store**: localStorage (existing) with versioned schema. Wrap access in `PresetStore` module responsible for CRUD, filtering, sorting, import/export. Future: switch to API or IndexedDB without touching UI.
- **State Management**: continue with vanilla JS objects for now or introduce lightweight store (e.g., Zustand/Pinia) if refactoring to framework. Key states: `presets`, `filters`, `activePresetId`, `viewMode`, `tagCatalog`.
- **Components (conceptual)**:
  - `PresetSidebar` (search + filters + toolbar + list)
  - `PresetCard` (thumbnail + metadata + actions)
  - `PresetDetailPanel` (selected preset summary)
  - `PresetEditorModal`, `BulkImportModal`
  - `TagFilterPanel`
- **Filtering/Search**: keep derived lists in memory; index text fields in lowercase. If volume grows, move filtering to Web Worker.
- **Thumbnail Handling**: use `<input type="file">` + FileReader for Base64; store small images only. Provide utilities for placeholder generation.
- **Compatibility**: loader respects `enabled` flags; when new global dimensions are added, presets default to `false` unless user edits them.

## 6. Future Enhancements
- Saved filter sets (“show only 自拍 + 清透”) and quick-access favorites.
- Cloud sync / shareable links once authentication exists.
- Automatic thumbnail generator (HTML2Canvas or AI).
- Analytics/insights: counts per tag, most-used presets, etc.

This plan should be the reference for upcoming implementation tasks. As features ship, keep this document updated so new contributors can onboard quickly.
