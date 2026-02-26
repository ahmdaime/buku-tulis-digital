# Buku Tulis Digital

Aplikasi web single-file (`index.html`) untuk pelajar sekolah rendah Malaysia (darjah 4-6) menulis nota digital dengan bantuan AI Gemini.

## Teknologi
- Pure HTML/CSS/JS dalam satu fail `index.html`
- API: Google Gemini 2.0 Flash (vision) untuk analisis gambar buku teks
- jsPDF (CDN) untuk export PDF
- Tiada framework, tiada build step

## Arkitektur Utama

### Canvas & Pages
- `CANVAS_WIDTH` / `CANVAS_HEIGHT` — saiz halaman
- `LINE_HEIGHT = 32` — jarak antara baris
- `pages[]` — array halaman; setiap halaman ada `textElements[]` dan `shapeElements[]`
- `savePage()` — simpan text elements (properties: `text`, `html`, `left`, `top`, `fontSize`, `color`, `fontWeight`, `fontStyle`, `textDecoration`). **TIDAK** simpan `width` atau `textAlign`.

### Fungsi Penting
- `createTextElementFromData(data)` — cipta text element pada canvas
- `createMindMapShape(type, x, y, w, h, color, sw, origW, origH)` — cipta shape element
- `generateShapeSVG()` — shape types: `rectangle`, `ellipse`, `line`, `arrow`
- `showToast(message, type)` — notifikasi toast (success/error/warning/info)
- `sanitizeHTML(html)` — sanitasi HTML untuk text elements
- `exportToPDF()` — export semua halaman ke PDF
- `exportJSON()` / `importJSON()` — backup/restore notebook penuh

### Tools
- **Pen (P)**: Lukisan biasa
- **Highlighter (H)**: Warna separuh lutsinar (`multiply`, `alpha 0.3`, `square lineCap`)
- **Eraser (E)**: True erasing (`destination-out`)
- **Text (T)**: Teks dengan bold/italic/underline
- **Shape (B)**: Bentuk geometri (rectangle, ellipse, line, arrow)
- **Table (J)**: Jadual dengan Enter=sel bawah, Tab=sel sebelah
- **Image (G)**: Import gambar
- **Cursor (C)**: Pilih dan gerak elemen

### Shortcut Keyboard
- `P`=pen, `H`=highlighter, `E`=eraser, `T`=text, `B`=shape, `J`=table, `G`=image, `C`=cursor
- `A`=AI, `F`=fullscreen
- `Ctrl+Z`=undo, `Ctrl+Shift+Z`/`Ctrl+Y`=redo
- `Ctrl+←/→`=navigasi halaman, `Ctrl+S`=save PNG

---

## Sistem AI

### Pipeline
1. Pengguna upload/paste gambar buku teks
2. `processImageWithAI(base64DataUrl)` hantar ke Gemini 2.0 Flash
3. Respons JSON `{title, content}` dipaparkan dalam preview modal
4. Pengguna boleh edit, tukar subjek/darjah, atau jana semula
5. Pilih "Isi sebagai Nota" → `autoFillNotebook()` atau "Isi sebagai Peta Minda" → `autoFillMindMap()`

### Prompt (`getAIPrompt()`)
- Role: guru berpengalaman sekolah rendah Malaysia
- `aiSelectedSubject` — subjek (auto/BM/BI/Matematik/Sains/Sejarah/Geografi/Pend. Moral/Pend. Islam/RBT)
- `aiSelectedDarjah` — darjah (4/5/6), mempengaruhi tahap bahasa
- Prinsip: nota untuk BELAJAR, bukan sekadar senarai fakta
- Setiap bahagian WAJIB ada contoh (`> Contoh:`)
- Fakta penting ditandai dengan `! Ingat:`
- Had: maks 15 point, maks 8 patah perkataan selepas `→`

### API Call (`processImageWithAI()`)
- Endpoint: `generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent`
- API key dalam header: `x-goog-api-key`
- `generationConfig`: `temperature: 0.7`, `topP: 0.9`, `topK: 40`, `maxOutputTokens: 2048`
- `lastAIImageDataUrl` — simpan gambar terakhir untuk "Jana Semula"

### autoFillNotebook — Parsing & Rendering

| Pattern | Render |
|---------|--------|
| `## teks` | 32px bold + garisan oren bawah |
| Baris selepas `##` | 18px italic kelabu (penerangan topik) — auto-detect |
| `### teks` | 24px bold + bar biru menegak di kiri |
| `istilah → maksud` | Bullet (●) bold + anak panah oren + meaning indented |
| `> Contoh: teks` | 💡 label hijau bold + teks italic hijau, indented |
| `! Ingat: teks` | ⭐ rectangle kuning + teks bold amber |
| `1. teks` | Nombor biru bold + teks normal |
| `---` | Garisan horizontal kelabu (#d1d5db) |
| Baris kosong | Spacing `LINE_HEIGHT` |
| Teks biasa | 20px normal hitam |

Helper `ensureSpace(needed)` — semak overflow halaman dan tambah halaman baru jika perlu.

### autoFillMindMap — Peta Minda Radial
- Nod pusat: ellipse merah jambu (#ec4899) 260×70px dengan tajuk
- Cabang disusun secara radial (radius 33% canvas)
- 8 warna bergilir untuk nod cabang (rectangle)
- Garisan kelabu (#9ca3af) menghubungkan pusat ke cabang
- Markdown markers (`##`, `###`, `---`) dibuang sebelum parse

### Preview Modal UI
- Dropdown **Subjek** dan **Darjah** di bahagian atas
- Input tajuk + textarea kandungan (boleh diedit oleh pengguna)
- Butang: Batal | ⟳ Jana Semula | Isi sebagai Nota | Isi sebagai Peta Minda

---

## Keselamatan
- API key disimpan dalam header (`x-goog-api-key`), bukan URL param
- HTML disanitasi sebelum dimasukkan ke DOM (hanya benarkan b, i, u, br, font, span)
- Validasi saiz halaman: min 100px, maks 5000px
- Validasi jadual: baris 1-50, lajur 1-20
- Amaran merah dalam dialog API key tentang penyimpanan localStorage

## Prestasi
- Canvas data disimpan sebagai JPEG (0.7 quality) — ~70% lebih kecil
- Dirty flag (`isDirty`) untuk autosave — skip jika tiada perubahan
- DocumentFragment untuk batch insert dalam `loadPage()`

## UI/UX
- Toast notification system (slide-in dari kanan, auto-dismiss 3s, 4 jenis: success/error/warning/info)
- Undo/Redo butang dalam toolbar dengan disabled state
- Mode indicator dengan ikon dan animasi pulse
- Zoom controls (+/−/reset) dan pinch-to-zoom pada mobile
- Spell check toggle
- ARIA labels dan `:focus-visible` indicators
- Tooltips (`title`) pada semua butang dan kontrol

## Responsive & Mobile
- Breakpoints: 1249px (tablet landscape), 1023px (tablet portrait), 767px (mobile landscape), 480px (mobile portrait)
- Touch-friendly: min 44px targets via `@media (hover: none) and (pointer: coarse)`
- Font size 16px pada input (elak iOS auto-zoom)
- Print stylesheet: sembunyikan toolbar, skala ke A4, `page-break-after: always`

## Export
- PNG: `Ctrl+S` — save halaman semasa
- PDF: jsPDF, loop semua halaman, render ke canvas, simpan sebagai `{tajuk}_{tarikh}.pdf`
- JSON: export/import backup penuh notebook (termasuk settings, pages, canvas data)

---

## Peraturan Pembangunan
- **JANGAN** tambah `width` atau `textAlign` pada text elements yang perlu di-save — `savePage()` tidak simpan properties ini
- Shape type `'arrow'` sedia ada dan boleh digunakan untuk anak panah dekoratif
- `createMindMapShape()` boleh digunakan semula untuk cipta shapes dalam nota (bukan mind map sahaja)
- Semua positioning guna `left`/`top` sahaja (absolute positioning)
- Guna `showToast()` bukan `alert()` untuk semua notifikasi
- Semua HTML dari user input mesti melalui `sanitizeHTML()` sebelum dimasukkan ke DOM
- Prompt AI mesti sentiasa sertakan contoh dan format JSON output
