# SRS Modul CAT: Katalog Produk — Frontend (FE)

**Module:** Katalog Produk
**Module Code:** `CAT`
**Route:** `/products`
**Priority:** High / Medium
**Version:** 1.0
**Implementation Status:** Planned — implementation follows PRD and current repository stack.
**Source of truth:** `01_PRD.md`, `00_PROJECT_CONTEXT.md`, `TECH_STACK_AND_TANSTACK_QUERY.md`.

---

## 1. Scope and Objective

Modul ini mendefinisikan pekerjaan Frontend (FE) untuk Katalog Produk. Dokumen ini sengaja hanya membahas satu modul agar dapat dikerjakan paralel dengan pasangan dokumen lainnya.
Target Tahap 1 adalah alur stabil, dapat diuji juri, mobile-first, aman, serta konsisten dengan kontrak API. Fitur P1/P2 tetap dicatat tetapi tidak boleh diklaim aktif sebelum implementasi dan integrasi diverifikasi.

### 1.1 PRD Feature Coverage

- F-CAT-01 Daftar Produk & Status Promosi
- F-CAT-02 Tambah/Edit Produk
- F-CAT-03 Produk Otomatis dari Chat/Import

### 1.2 Actors and Access

| Actor | Access | Constraint |
|---|---|---|
| `CRAFTER` | Read/write tenant sendiri | Tidak melihat usaha lain |
| `DEMO_JUDGE` | Sandbox sesuai modul | Data contoh, reset terbatas |
| `ADMIN` | Monitoring/dukungan sesuai kebutuhan | Tidak membaca token mentah |
| `ADVISOR` | P2 read-only ringkasan | Tidak menjadi scope Tahap 1 |

### 1.3 Shared Contract

- TypeScript strict mode; naming mengikuti module code dan Feature ID.
- Semua request memakai session cookie Supabase; client tidak mengirim atau menyimpan secret.
- Semua response error memakai `{ error: { code, message, details? } }`.
- Semua timestamp memakai ISO 8601 UTC; UI mengonversi ke timezone pengguna.
- Semua list memakai `{ items, nextCursor }` bila berpotensi besar.

## 2. FE Responsibilities

### FR-FE-CAT-001: Screen and Route

FE menyediakan layar: Product list; product detail; add/edit form; image uploader; promotion history. Route harus mengikuti App Router dan protected-route policy.

### FR-FE-CAT-002: Shared Components

Komponen minimum: ProductCard, ProductForm, ProductStatusBadge, ImageUploader, EmptyCatalog, Pagination.
Komponen reusable diprioritaskan; jangan membuat state global baru jika local state cukup.

### FR-FE-CAT-003: Server State

- Gunakan TanStack Query untuk response API; query key harus memasukkan businessId/entity ID yang relevan.
- `staleTime` mengikuti karakter data; mutation tidak auto-retry.
- Invalidate hanya key terdampak setelah server mutation sukses.
- Polling hanya untuk status asinkron dan berhenti pada status terminal.
- Jangan memasukkan token, credential, atau payload sensitif ke query key/cache.

### FR-FE-CAT-004: Form and Validation

- Validasi client memberi umpan balik cepat; server tetap menjadi sumber validasi final.
- Field wajib diberi label, hint, error text, dan `aria-describedby`.
- Tombol submit disabled saat mutation berjalan; cegah double submit.
- Input file memvalidasi format, ukuran, jumlah, dan preview sebelum upload.
- Data draft yang aman dipertahankan saat offline/session expired; jangan menyimpan secret.

## 3. UI/UX Specification

### 3.1 Layout

1. Layar utama menampilkan konteks Katalog Produk tanpa mengandalkan warna saja.
2. Mobile-first pada lebar 360–414 px; layout desktop mendukung minimal 1280 px untuk juri.
3. Primary action selalu terlihat; secondary action tidak mengalahkan aksi aman.
4. State loading menggunakan skeleton/progress yang menjelaskan pekerjaan sebenarnya.
5. Empty state selalu memberi langkah berikutnya, bukan hanya teks kosong.
6. Error state menyediakan retry atau jalur manual yang jelas.

### 3.2 Accessibility and i18n

- Kontras minimum WCAG AA; target sentuh sekitar 44–48 px.
- Semua input memiliki label; dialog mengatur focus trap dan tombol Escape.
- Status selalu memakai kombinasi ikon, teks, dan warna.
- Mendukung Bahasa Indonesia sebagai default dan Inggris sebagai pilihan.
- Teks panjang, nama usaha, caption, dan zoom browser tidak boleh merusak layout.
- Hormati `prefers-reduced-motion`; jangan menjadikan animasi sebagai satu-satunya indikator.

### 3.3 State Matrix

| State | FE behavior | User action |
|---|---|---|
| Initial/loading | Skeleton atau progress kontekstual | Menunggu atau meninggalkan halaman |
| Success | Data/content tampil; toast seperlunya | Lanjut ke aksi berikut |
| Empty | Penjelasan dan CTA | Membuat data pertama |
| Validation error | Error dekat field | Memperbaiki input |
| API error | Pesan aman dan retry | Coba lagi/jalur manual |
| Offline | Banner; jangan hapus draft | Reconnect |
| Session expired | Dialog/redirect login | Login ulang |

## 4. FE Data and API Wiring

Endpoint yang dikonsumsi: GET/POST /api/products; GET/PATCH/DELETE /api/products/{id}; POST /api/products/{id}/images; GET /api/products/{id}/promotion-status.
Setiap hook menggunakan `lib/api-client.ts`; response dipetakan ke tipe domain, bukan `any`.

### 4.1 Query/Mutation Table

| Operation | Query key / mutation | Invalidation |
|---|---|---|
| Read list/detail | `catList`, `catDetail` | Refetch bila stale |
| Create/update | `catMutation` | Invalidate affected entity/list |
| Delete/cancel | `catDelete` | Remove detail; invalidate list |
| Async status | entity status key | Poll only while queued/processing |

### 4.2 API Response Expectations

- Success response harus memiliki shape stabil dan dapat ditipkan.
- HTTP 401 memicu session recovery; 403 menampilkan access restricted; 404 menampilkan not found; 409 menjelaskan konflik.
- HTTP 422 memetakan `details.field` ke form; HTTP 429 menampilkan cooldown; HTTP 5xx menyediakan retry.
- FE tidak menampilkan stack trace, SQL, provider error mentah, access token, atau API key.

## 5. User Flow

```mermaid
flowchart TD
A[Open module] --> B{Session valid?}
B -->|No| C[Session recovery / Login]
B -->|Yes| D[Load scoped data]
D --> E{State} 
E -->|Empty| F[Guided primary action]
E -->|Ready| G[User interaction]
E -->|Error| H[Safe error + Retry]
G --> I[Mutation / navigation]
I --> J[Invalidate affected query]
```

Alur spesifik: User opens catalog; sees promotion status; adds or edits product; uploads photo; saves; product becomes selectable in chat and creative generation; auto-created product remains reviewable.

## 6. Acceptance Criteria

- `AC-FE-CAT-001`: List shows photo/name/status.
- `AC-FE-CAT-002`: create and edit validate fields.
- `AC-FE-CAT-003`: delete requires confirmation.
- `AC-FE-CAT-004`: image upload reports progress.
- `AC-FE-CAT-005`: status reflects posting history.
- `AC-FE-CAT-006`: invalid businessId cannot access another tenant.
- `AC-FE-CAT-099`: UI memenuhi keyboard navigation, responsive layout, ID/EN text, dan safe error handling.

## 7. Edge Cases and Error Handling

- **Missing name:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **invalid price:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **unsupported image:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **upload timeout:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **duplicate submit:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **deleted product referenced by generation:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **empty catalog:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **pagination failure:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- Double click tidak membuat mutation ganda.
- Navigasi saat request aktif tidak menyebabkan crash atau state korup.
- Unmount menghentikan polling dan object URL lokal dibersihkan.

## 8. FE File Convention

- Route: `app/products/page.tsx`.
- Feature components: `modules/cat/components/`.
- Hooks: `hooks/use-cat.ts` atau module-local hook.
- Types: `modules/shared/types` atau domain type terdekat.
- API access: `lib/api-client.ts`; query client: `lib/query-client.ts`.
- UI tokens berasal dari designer; jangan hardcode warna/typography tanpa token.

## 9. FE Test Checklist

- Render loading, success, empty, error, offline, dan session-expired state.
- Test field validation dan keyboard/focus order.
- Test mutation success, 401/403/409/422/429/5xx.
- Test query invalidation dan tidak ada duplicate request.
- Test tenant data tidak tampil pada response yang salah.
- Jalankan `npm run lint` dan `npm run build`.

## 10. FE Handoff Notes

FE tidak boleh mengasumsikan fitur backend yang belum tersedia. Gunakan mock response berbasis kontrak ini, lalu ganti `queryFn` tanpa mengubah komponen ketika endpoint siap.
Pasangan BE wajib menjaga field, enum status, error code, dan permission yang dicantumkan dalam SRS BE modul yang sama.

*End of SRS_FE_CAT — Katalog Produk*
