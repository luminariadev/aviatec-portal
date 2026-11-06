# Panduan Scraping & Cloning Website Pixel-Perfect (Studi Kasus: FlowFest)

Dokumen ini menjelaskan teknik yang digunakan untuk melakukan *cloning* sebuah website dari URL (seperti `https://www.flowfest.co.uk/`) agar berjalan 100% identik di *localhost*, lengkap dengan animasi, *layout*, dan *styling*-nya.

## Mengapa Pendekatan Biasa Sering Gagal?
Biasanya, ketika kita mencoba "meniru" website kompleks ke dalam *framework* seperti React/Next.js, hasilnya sering berantakan atau animasinya membeku (*stuck*). Ini karena:
1. **Konflik Virtual DOM**: React mengubah struktur DOM secara dinamis yang merusak skrip animasi bawaan website asli (seperti GSAP atau Webflow.js).
2. **Missing Assets**: Link gambar, CSS, dan JS pada web asli sering menggunakan *relative path* (contoh: `src="/images/logo.png"`). Saat dijalankan di lokal, file ini akan *NotFound* (404).
3. **Event Listener Timing**: Skrip animasi bergantung pada *event* `DOMContentLoaded`. Di Next.js, HTML dimuat oleh React *setelah* event tersebut berlalu.

---

## Langkah-langkah Cloning (Metode Bypass)

Berikut adalah algoritma dari skrip Node.js yang saya gunakan untuk mengkloning FlowFest:

### 1. Download Raw HTML (Scraping Dasar)
Langkah pertama adalah mengunduh payload HTML murni dari URL target menggunakan modul bawaan Node.js (`https`) atau melalui terminal:
```javascript
const fs = require('fs');
const https = require('https');

https.get('https://www.flowfest.co.uk/', (res) => {
  let data = '';
  res.on('data', (chunk) => { data += chunk; });
  res.on('end', () => { fs.writeFileSync('raw.html', data); });
});
```

### 2. Memperbaiki Aset (Absolute Path Correction)
Setelah HTML didapat, semua *link* relatif yang mengarah ke file CSS, JS, dan Gambar harus diubah menjadi URL absolut yang menunjuk ke server asli.
Saya menggunakan *Regular Expression* (RegEx) untuk menemukan dan menggantinya:
```javascript
let html = fs.readFileSync('raw.html', 'utf8');

// Mengubah href="/css/style.css" menjadi href="https://www.flowfest.co.uk/css/style.css"
html = html.replace(/href="\/(?!\/)/g, 'href="https://www.flowfest.co.uk/');
html = html.replace(/src="\/(?!\/)/g, 'src="https://www.flowfest.co.uk/');
html = html.replace(/srcset="\/(?!\/)/g, 'srcset="https://www.flowfest.co.uk/');
```

### 3. Trik "The Next.js Bypass" (Sangat Penting!)
Untuk memastikan semua animasi rumit (seperti *loading screen* dan transisi) tidak rusak oleh siklus Next.js, kita **tidak** memasukkan kode tersebut ke dalam komponen React (seperti `page.tsx`).

Sebagai gantinya:
1. Simpan HTML yang sudah diperbaiki ke dalam folder `public/` sebagai `clone.html`.
2. Hapus/ubah nama `src/app/page.tsx` agar tidak terjadi konflik.
3. Konfigurasi **Next.js Rewrites** di `next.config.ts` agar saat *user* membuka `localhost:3000/`, server langsung menyajikan file HTML murni tersebut tanpa campur tangan React.

```typescript
// next.config.ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  async rewrites() {
    return {
      beforeFiles: [
        {
          source: '/',
          destination: '/clone.html', // Mengarah ke file HTML murni di folder public
        },
      ],
    };
  },
};
export default nextConfig;
```

## Hasil Akhir
Dengan teknik ini, browser memuat *website clone* seolah-olah ia sedang memuat situs aslinya secara tradisional. 
Semua library JavaScript (Webflow, Barba.js, Lenis, GSAP) akan terinisialisasi secara natural dan animasi berjalan 100% sempurna. 

Setelah kerangka ini berjalan sukses di lokal, barulah kita mulai masuk ke dalam file HTML tersebut untuk mencari dan mengganti (*Find and Replace*) teks, URL gambar, atau logo menjadi *brand* milik kita sendiri (dalam kasus ini: Sabit Academy).
