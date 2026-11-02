# SRS Modul CHAT: Chat Agent — Frontend (FE)

**Module:** Chat Agent
**Module Code:** `CHAT`
**Route:** `/chat`
**Priority:** Very High / High / Medium
**Version:** 1.0
**Implementation Status:** Planned — implementation follows PRD and current repository stack.
**Source of truth:** `01_PRD.md`, `00_PROJECT_CONTEXT.md`, `TECH_STACK_AND_TANSTACK_QUERY.md`.

---

## 1. Scope and Objective

Modul ini mendefinisikan pekerjaan Frontend (FE) untuk Chat Agent. Dokumen ini sengaja hanya membahas satu modul agar dapat dikerjakan paralel dengan pasangan dokumen lainnya.
Target Tahap 1 adalah alur stabil, dapat diuji juri, mobile-first, aman, serta konsisten dengan kontrak API. Fitur P1/P2 tetap dicatat tetapi tidak boleh diklaim aktif sebelum implementasi dan integrasi diverifikasi.

### 1.1 PRD Feature Coverage

- F-CHAT-01 Chat Web
- F-CHAT-02 Upload Foto Produk
- F-CHAT-03 Kartu Opsi Poster
- F-CHAT-04 Status Asinkron
- F-CHAT-05 Pertanyaan Data Kurang
- F-CHAT-06 Batas Marketing
- F-CHAT-07 Voice Note
- F-CHAT-08 Riwayat
- F-CHAT-09 Agent Core Lintas Kanal

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

### FR-FE-CHAT-001: Screen and Route

FE menyediakan layar: Conversation list; chat thread; attachment preview; processing state; poster option card. Route harus mengikuti App Router dan protected-route policy.

### FR-FE-CHAT-002: Shared Components

Komponen minimum: ChatThread, MessageBubble, AttachmentPicker, VoiceRecorder, QuickReplyChips, PosterOptionCarousel, ProcessingNotice.
Komponen reusable diprioritaskan; jangan membuat state global baru jika local state cukup.

### FR-FE-CHAT-003: Server State

- Gunakan TanStack Query untuk response API; query key harus memasukkan businessId/entity ID yang relevan.
- `staleTime` mengikuti karakter data; mutation tidak auto-retry.
- Invalidate hanya key terdampak setelah server mutation sukses.
- Polling hanya untuk status asinkron dan berhenti pada status terminal.
- Jangan memasukkan token, credential, atau payload sensitif ke query key/cache.

### FR-FE-CHAT-004: Form and Validation

- Validasi client memberi umpan balik cepat; server tetap menjadi sumber validasi final.
- Field wajib diberi label, hint, error text, dan `aria-describedby`.
- Tombol submit disabled saat mutation berjalan; cegah double submit.
- Input file memvalidasi format, ukuran, jumlah, dan preview sebelum upload.
- Data draft yang aman dipertahankan saat offline/session expired; jangan menyimpan secret.

## 3. UI/UX Specification

### 3.1 Layout

1. Layar utama menampilkan konteks Chat Agent tanpa mengandalkan warna saja.
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

Endpoint yang dikonsumsi: GET /api/chat/conversations; GET /api/chat/messages; POST /api/chat/messages; POST /api/chat/attachments; GET /api/generations/{id}.
Setiap hook menggunakan `lib/api-client.ts`; response dipetakan ke tipe domain, bukan `any`.

### 4.1 Query/Mutation Table

| Operation | Query key / mutation | Invalidation |
|---|---|---|
| Read list/detail | `chatList`, `chatDetail` | Refetch bila stale |
| Create/update | `chatMutation` | Invalidate affected entity/list |
| Delete/cancel | `chatDelete` | Remove detail; invalidate list |
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

Alur spesifik: User sends text/photo; agent validates and asks missing facts or creates generation; client shows queued/processing status; result returns as cards; user selects, revises, or opens review flow.

## 6. Acceptance Criteria

- `AC-FE-CHAT-001`: Messages persist in order.
- `AC-FE-CHAT-002`: upload preview works.
- `AC-FE-CHAT-003`: invalid files rejected before send.
- `AC-FE-CHAT-004`: status survives leaving page.
- `AC-FE-CHAT-005`: generation polling stops when terminal.
- `AC-FE-CHAT-006`: out-of-scope question receives concise redirect.
- `AC-FE-CHAT-007`: old messages paginate.
- `AC-FE-CHAT-099`: UI memenuhi keyboard navigation, responsive layout, ID/EN text, dan safe error handling.

## 7. Edge Cases and Error Handling

- **AI timeout:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **unsupported attachment:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **transcription failure:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **queue unavailable:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **duplicate send:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **offline:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **generation failed:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **conversation inaccessible:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- **rate limit:** tampilkan feedback yang dapat ditindaklanjuti; pertahankan data aman; jangan bocorkan detail internal.
- Double click tidak membuat mutation ganda.
- Navigasi saat request aktif tidak menyebabkan crash atau state korup.
- Unmount menghentikan polling dan object URL lokal dibersihkan.

## 8. FE File Convention

- Route: `app/chat/page.tsx`.
- Feature components: `modules/chat/components/`.
- Hooks: `hooks/use-chat.ts` atau module-local hook.
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

*End of SRS_FE_CHAT — Chat Agent*
