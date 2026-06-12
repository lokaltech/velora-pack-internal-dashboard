# Project Development Plan
## Project: Velora Pack - Internal Dashboard Admin & CRM
**Tech Stack:** Next.js, TypeScript, Tailwind CSS, shadcn/ui, Prisma ORM, PostgreSQL, Vercel.

---

## 🚀 Alur Kerja & Metodologi Pembuatan
Pengembangan akan dibagi menjadi **6 Fase Terstruktur** untuk memastikan kestabilan kode, keamanan otentikasi, dan fungsionalitas fitur yang akurat sesuai dengan PRD.

---

## 📅 Detail Fase Pengembangan

### 🔹 Fase 1: Desain Skema Database & Inisialisasi (Hari 1-2)
Fokus pada pembuatan arsitektur tabel menggunakan Prisma ORM untuk mendukung relational mapping (Sales ke Prospek).
* **Tugas Utama:**
    * Definisikan model `User` (id, nama, email, password, role [OWNER, SALES]).
    * Definisikan model `Product` (id, name, slug, description, category, specs, moq, images).
    * Definisikan model `Prospect` (id, companyName, picName, phone, email, status, checklistJson, salesId).
    * Definisikan model `Lead` (id, name, email, company, notes, status, createdAt).
    * Definisikan model `WebsiteSetting` dan `Blog`.
    * Lakukan *Prisma Migration* ke basis data PostgreSQL.
    * Buat skrip `seed.ts` untuk membuat 1 akun Owner default, 2 akun Sales uji coba, dan data produk awal.

### 🔹 Fase 2: Otentikasi & Authorization Guard (Hari 3-4)
Membangun pintu masuk sistem yang aman serta membatasi halaman berdasarkan hak akses.
* **Tugas Utama:**
    * Integrasi sistem otentikasi menggunakan Auth.js (Next-Auth) atau sistem session berbasis JWT pada Next.js App Router.
    * Buat halaman `/login` dengan desain clean memakai komponen dari shadcn/ui.
    * Implementasi `middleware.ts` Next.js untuk proteksi route:
        * Mencegah user anonim masuk ke rute `/admin/*`.
        * Mencegah user dengan role `SALES` masuk ke rute `/admin/settings`, `/admin/products`, dan `/admin/users`.

### 🔹 Fase 3: Pembuatan Logika Bisnis & Server Actions (Hari 5-7)
Memperkuat *backend layer* menggunakan Server Actions untuk penanganan CRUD data secara aman.
* **Tugas Utama:**
    * Buat fungsi Server Actions untuk penanganan Produk (Create, Update, Delete).
    * Buat Server Actions untuk Prospek dengan validasi keamanan ketat:
        * *Skenario Sales:* Aksi `getProspects()` otomatis menyertakan filter `where: { salesId: session.user.id }`.
        * *Skenario Owner:* Aksi `getProspects()` mengembalikan seluruh data tanpa filter sales.
    * Buat Server Actions untuk menyimpan inquiry form dari website publik ke tabel `Lead`.

### 🔹 Fase 4: Pengembangan Komponen UI Dashboard (Hari 8-11)
Membangun antarmuka dashboard yang responsif dan interaktif menggunakan Tailwind CSS dan shadcn/ui.
* **Tugas Utama:**
    * Buat *Sidebar Layout* utama yang dinamis (menu berubah otomatis menyesuaikan role login).
    * **Halaman Product Management:** Tampilan tabel (`shadcn/ui/table`) dengan fitur filter, tombol tambah produk dengan dialog modal.
    * **Halaman Client Management (CRM):** Tampilan daftar prospek, indikator progress bar berdasarkan jumlah checklist follow-up yang terisi, dan form edit status prospek yang ramah perangkat mobile.
    * **Halaman Leads & Website Settings:** Form terstruktur untuk mengubah konfigurasi perusahaan dan pelacakan status leads masuk.

### 🔹 Fase 5: Integrasi & Hubungan Antar Modul (Hari 12-13)
Menghubungkan frontend website publik dengan dashboard admin internal.
* **Tugas Utama:**
    * Modifikasi komponen Form di website utama agar memicu Server Action penyimpanan lead sebelum menjalankan logika pemanggilan WhatsApp API.
    * Hubungkan komponen statis website utama (seperti nomor WA di Navbar/Footer, alamat di halaman kontak, metrik kapasitas produksi) agar langsung mengambil data dari tabel `WebsiteSetting` melalui data fetching asinkron di Server Components.

### 🔹 Fase 6: Pengujian, Optimasi, & Deployment (Hari 14-15)
Memastikan aplikasi bebas dari bug keamanan dan siap digunakan secara publik.
* **Tugas Utama:**
    * *Security Testing:* Mencoba menembus halaman admin settings menggunakan akun sales secara langsung melalui URL untuk memastikan middleware bekerja 100%.
    * *Data Leakage Testing:* Memastikan Sales A tidak dapat memanipulasi ID di payload untuk melihat data Sales B.
    * Deployment database ke penyedia cloud (seperti Supabase/Neon PostgreSQL) dan frontend/dashboard ke platform Vercel.
    * Konfigurasi environment variables (`DATABASE_URL`, `NEXTAUTH_SECRET`, dll) pada dashboard Vercel.

---

## 🛠️ Mitigasi Risiko & Strategi Teknis
1.  **Risiko Sinkronisasi Data WhatsApp:** User menutup browser sebelum sempat dialihkan ke WA padahal form sudah terkirim.  
    * *Solusi:* Data wajib masuk ke database dashboard di langkah pertama (fase pemicu awal), setelah status respons database mengembalikan nilai sukses (`true`), browser baru mengeksekusi perpindahan halaman WhatsApp.
2.  **Risiko Kebocoran Data Antar Sales:** * *Solusi:* Jangan pernah mengandalkan kiriman `salesId` dari form frontend saat memperbarui data prospek. Server Action harus selalu membaca ID pengguna aktif dari session token yang valid di sisi server (`auth()` session).
