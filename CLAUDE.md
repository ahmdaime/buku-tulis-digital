# Buku Tulis Digital

Aplikasi web single-file (`index.html`) untuk pelajar sekolah rendah Malaysia menulis nota digital dengan bantuan AI.

## Teknologi
- Pure HTML/CSS/JS dalam satu fail `index.html`
- API: Google Gemini 2.0 Flash (vision) untuk analisis gambar buku teks
- Tiada framework, tiada build step

## Arkitektur Utama

### Canvas & Pages
- `CANVAS_WIDTH` / `CANVAS_HEIGHT` — saiz halaman
- `LINE_HEIGHT = 32` — jarak antara baris
- `pages[]` — array halaman; setiap halaman ada `textElements[]` dan `shapeElements[]`
- `savePage()` — simpan text elements (properties: `text`, `html`, `left`, `top`, `fontSize`, `color`, `fontWeight` SAHAJA). **TIDAK** simpan `width` atau `textAlign`.

### Fungsi Penting
- `createTextElementFromData(data)` (~baris 4765) — cipta text element pada canvas
- `createMindMapShape(type, x, y, w, h, color, sw, origW, origH)` (~baris 5890) — cipta shape element
- `generateShapeSVG()` (~baris 3111) — shape types: `rectangle`, `ellipse`, `line`, `arrow`
- `getAIPrompt()` (~baris 5618) — prompt untuk Gemini AI
- `autoFillNotebook(title, content)` (~baris 5752) — isi nota berstruktur pada canvas
- `autoFillMindMap(title, content)` (~baris 5925) — cipta peta minda radial pada canvas

## Perubahan Terkini: Nota Terstruktur & Cantik

### 1. AI Prompt (`getAIPrompt()`)
- Role: "guru pelbagai subjek" (bukan BM sahaja) untuk darjah 4-6
- Format wajib: `##` tajuk, `###` sub-tajuk, `istilah → maksud`, nombor untuk fakta
- Had: MAKS 5 patah perkataan selepas `→` untuk elak overflow
- 3 contoh disertakan (Sejarah, Sains, BM) supaya output konsisten untuk semua subjek
- Peraturan: 2-4 sub-tajuk, min 2 item setiap, maks 10 point, 1 baris per point

### 2. `autoFillNotebook()` — Parsing Berstruktur
Parsing setiap baris berdasarkan pattern:

| Pattern | Render |
|---------|--------|
| `## teks` | Teks 32px bold + garisan bawah oren (line shape, #f97316, stroke 3) |
| `### teks` | Teks 24px bold (#1e293b) + spacing tambahan |
| `teks → teks` | **Dua baris**: Baris 1 = term bold 20px; Baris 2 = arrow shape oren + meaning 18px (indented) |
| `---` | Garisan horizontal kelabu (#d1d5db) sebagai pemisah |
| `1. teks` / teks biasa | Teks 20px normal |
| Baris kosong | Spacing `LINE_HEIGHT` |

Layout definisi dua baris (elak overflow):
```
Kata kerja transitif              ← bold 20px, LEFT_MARGIN
   ──► perlu objek                ← arrow 40px + meaning 18px, indent LEFT_MARGIN+20
```

Helper `ensureSpace(needed)` — semak overflow halaman dan tambah halaman baru jika perlu.

### 3. `autoFillMindMap()` — Strip Markdown
Sebelum parse points untuk peta minda, markdown markers dibuang:
```javascript
const cleanContent = content
    .replace(/^##+ /gm, '')
    .replace(/^---$/gm, '')
    .trim();
```

## Peraturan Pembangunan
- **JANGAN** tambah `width` atau `textAlign` pada text elements yang perlu di-save — `savePage()` tidak simpan properties ini
- Shape type `'arrow'` sedia ada dan boleh digunakan untuk anak panah dekoratif
- `createMindMapShape()` boleh digunakan semula untuk cipta shapes dalam nota (bukan mind map sahaja)
- Semua positioning guna `left`/`top` sahaja (absolute positioning)
