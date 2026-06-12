# Product Requirement Document (PRD)
## Project: Velora Pack - Internal Dashboard Admin & CRM
**Version:** 1.0  
**Status:** Approved  
**Target Release:** 2026  
**Author:** LokalTech  

---

## 1. Executive Summary & Objectives
Velora Pack adalah platform B2B manufaktur plastik OPP. Untuk mendukung pertumbuhan bisnis, efisiensi operasional, dan efektivitas tim sales, diperlukan sebuah **Internal Dashboard**. 

Dashboard ini berfungsi sebagai pusat kendali operasional bagi **Owner** dan alat kerja harian bagi **Sales Team**. Fokus utamanya adalah menjembatani *leads* organik yang masuk dari website publik dengan aktivitas prospek taktis (offline) yang dilakukan oleh tim sales di lapangan, serta memberikan fleksibilitas penuh untuk mengelola konten dan pengaturan website tanpa ketergantungan pada developer.

---

## 2. User Roles & Access Control (RBAC)
Sistem ini membagi hak akses secara ketat menggunakan *Role-Based Access Control* (RBAC):

| Fitur / Modul | Role: Owner (Super Admin) | Role: Sales (Field Agent) |
| :--- | :--- | :--- |
| **Product Management** | Full Access (CRUD) | Read Only |
| **Client Management (Prospects)** | Lihat Semua Prospek, Ganti Mapping Sales | Hanya Lihat & Edit Prospek Buatan Sendiri |
| **Lead & Inquiry Management** | Full Access (Follow-up & Assign) | No Access / Read Only (Opsional) |
| **Website Settings & CMS** | Full Access (Ubah data info pabrik & blog) | No Access |
| **User/Sales Management** | Full Access (Tambah/Hapus Akun Sales) | No Access |

---

## 3. Detailed Functional Requirements

### 3.1. Product Management (Katalog Internal & Publik)
* **Deskripsi:** Mengelola produk yang tampil di halaman *Product Showcase* website publik.
* **Fitur Utama:**
    * **CRUD Produk:** Menambah, melihat, mengubah, dan menghapus data produk (Plastik OPP, Printed Packaging, dll).
    * **Atribut Produk:** Nama produk, slug (SEO), kategori, deskripsi detail, spesifikasi teknis (ketebalan mikron, ukuran min/max), Minimum Order Quantity (MOQ), dan galeri foto produk.
    * **Kategori & Tag:** Pengelompokan berdasarkan jenis industri (Fashion, F&B, Retail) untuk mempermudah filter di website utama.

### 3.2. Client & Prospect Management (Offline CRM)
* **Deskripsi:** Mengelola data prospek/klien potensial yang ditemukan oleh sales di luar website (pameran, cold calling, kunjungan pabrik).
* **Fitur Utama:**
    * **Sales Isolation Area:** Sales A hanya bisa melihat, menambah, dan mengedit prospek yang mereka buat sendiri. Owner bisa melihat total rekapitulasi dari seluruh sales.
    * **Data Prospek:** Nama Perusahaan, Nama PIC, Jabatan, Nomor Telepon/WhatsApp, Email, Industri, Potensi Volume Order (Ton/Bulan).
    * **Follow-up Checklist:** Status tahapan prospek dalam bentuk checklist interaktif:
        * [ ] Perkenalan / Cold Call
        * [ ] Pengiriman Sampel Plastik
        * [ ] Negosiasi Harga / Penawaran (Quotation)
        * [ ] Trial Produksi / QC Approved
        * [ ] Closing / First Order
    * **Mapping & Reassign:** Owner memiliki kemampuan untuk memindahkan (reassign) prospek dari Sales A ke Sales B jika diperlukan.

### 3.3. Lead & Inquiry Management (Inbound Web Leads)
* **Deskripsi:** Menampung data dari visitor website publik yang menekan tombol *Request Quotation* sebelum mereka diarahkan ke WhatsApp Sales.
* **Fitur Utama:**
    * **Data Capture System:** Menyimpan data form (Nama, Email, Perusahaan, Kebutuhan Spesifikasi, Estimasi Kuantitas) ke database internal sesaat sebelum fungsi `window.open(whatsappUrl)` berjalan. Hal ini mencegah hilangnya data akibat *drop-off* di WhatsApp.
    * **Owner Inbox & Tracking:** Halaman khusus bagi Owner untuk melihat performa konversi website. Owner dapat menandai status lead ini: *New, Contacted, In-Negotiation, Deal, or Spam*.

### 3.4. Website Settings & CMS (Dynamic Configuration)
* **Deskripsi:** Mengubah data-data statis pada website publik agar tetap relevan tanpa melakukan *hardcode*.
* **Fitur Utama:**
    * **Company Profile Data:** Mengubah nama perusahaan, alamat pabrik fisik, Google Maps embed link, email resmi, dan nomor WhatsApp utama.
    * **Manufacturing Capabilities & Capacity Manager (Fitur Tambahan):** Mengubah metrik kapasitas produksi yang dipajang di website (misal: mengedit teks "100 Tons / Month" menjadi "120 Tons / Month" saat pabrik melakukan ekspansi mesin).
    * **Portfolio & Certification Manager (Fitur Tambahan):** Menunggah sertifikasi baru (ISO, Food Grade, Halal) dan foto hasil produksi kemasan terbaru untuk menjaga kredibilitas website.
    * **Blog CMS (Fitur Tambahan):** Pembuatan artikel SEO (Input judul, meta deskripsi, isi artikel teks kaya/Rich Text, gambar utama) untuk mendongkrak traffic organik.

### 3.5. Sales Performance Monitor (Fitur Tambahan untuk Owner)
* **Deskripsi:** Grafik ringkas pada halaman utama dashboard Owner.
* **Fitur Utama:**
    * Jumlah prospek aktif per sales.
    * Rasio konversi dari prospek offline menjadi pembeli.
    * Jumlah total lead inbound dari website setiap bulannya.

---

## 4. Non-Functional Requirements (Teknis & Keamanan)
1.  **Keamanan Data:** Enkripsi password menggunakan bcrypt. Isolasi query di level database/ORM (Prisma) menggunakan filter `where: { salesId: currentUser.id }` untuk role Sales.
2.  **Performa (Fast Loading):** Pemanfaatan Next.js Server Actions untuk mutasi data yang cepat dan efisien tanpa API routing overhead yang berat.
3.  **Desain UI/UX:** Dashboard menggunakan tema profesional, bersih, dan fungsional. Menggunakan komponen dasar dari shadcn/ui dengan palet warna dominan gelap/abu-abu netral yang dikombinasikan dengan aksen korporat.
4.  **Mobile Friendly:** Mengingat sales sering berada di lapangan (pabrik klien), modul Client Management harus dioptimalkan untuk tampilan *mobile web browser*.

---

## 5. Kriteria Penerimaan (Acceptance Criteria)
* Sales **tidak bisa** mengakses URL `/admin/settings` atau `/admin/products` untuk melakukan perubahan data.
* Ketika sales membuka `/admin/prospects`, data yang keluar *hanya* yang terikat dengan ID sales tersebut.
* Ketika form *Request Quotation* diisi di website publik, data tersimpan di PostgreSQL dalam waktu kurang dari 2 detik sebelum berpindah ke WhatsApp.
