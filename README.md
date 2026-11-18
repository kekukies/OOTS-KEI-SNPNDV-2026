<<<<<<< HEAD
# OOTS 2026 — Portofolio Pribadi

Repo kerja Workshop OOTS 2026. Isinya satu halaman portofolio yang dibangun
dalam 8 checkpoint, dari scaffold kosong sampai URL publik.

Setiap checkpoint punya branch sendiri. Semua branch bisa dijalankan.

| CP  | Branch        | Isi                                         |
| --- | ------------- | ------------------------------------------- |
| 0   | `00-start`    | Scaffold + Tailwind + token, halaman kosong |
| 1   | `01-tailwind` | Nav, hero, band, 1 baris proyek, footer     |
| 2   | `02-darkmode` | Toggle tema                                 |
| 3   | `03-each`     | Data + `{#each}`                            |
| 4   | `04-komponen` | Komponen + props + section Tentang          |
| 5   | `05-form`     | Form kontak + status                        |
| 6   | `06-resend`   | Kirim email sungguhan                       |
| 7   | `07-deploy`   | Deploy ke Vercel                            |

## Yang dibutuhkan

- [bun](https://bun.sh) 1.4 atau lebih baru
- VS Code (disarankan) dengan ekstensi Svelte

Node.js tidak perlu dipasang; bun menjalankan semuanya.

## Menjalankan

```sh
bun install
bun run dev
```

Buka `http://localhost:5173`.

```sh
bun run check   # TypeScript + Svelte, harus hijau
bun run build   # pastikan tidak ada yang rusak sebelum push
```

## Cara membaca checkpoint

`CHECKPOINT.md` di repo ini **selalu milik branch yang sedang kalian buka**.
Kalau kalian mengerjakan `00-start`, isinya CP0 — bukan CP6.

Karena itu, **baca `CHECKPOINT.md` di GitHub, bukan di laptop.** File di laptop
tidak ikut berubah saat kalian pindah branch, jadi bisa menyesatkan.

## Kalau tertinggal

Empat cara, dari yang paling aman.

```sh
# 1 — clone terpisah sebagai pembanding. Kerjaan kalian tidak hilang.
git clone -b 04-komponen https://github.com/Dinus-Open-Source-Community/oots-portfolio-2026.git ref-04

# 2 — ambil satu folder dari checkpoint ke repo sendiri
git remote add ref https://github.com/Dinus-Open-Source-Community/oots-portfolio-2026.git
git fetch ref
git checkout ref/04-komponen -- src/

# 3 — reset penuh. Hanya kalau benar-benar tersesat.
git add -A && git commit -m "wip"
git reset --hard ref/04-komponen
```

```text
# 4 — tanpa terminal
Buka branch yang dituju di GitHub, tekan tombol "Copy raw file",
tempel ke file yang sama di VS Code.
```

## Mengganti isi

Semua isi di sini masih contoh. Ganti nama, tagline, tiga proyek, dan
foto dengan milik kalian sendiri sebelum 24.00 di hari kedua.
=======
# OOTS-KEI-2026
>>>>>>> 3e6f8fd28f748ff672cf7ed18d387b975b7b933c
