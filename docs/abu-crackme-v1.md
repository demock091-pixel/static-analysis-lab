# Analisis Statis: abu_crackme_v1 (Crack me 1.exe)

**Sumber:** crackmes.one (originally crackmes.de, by gauri)
**Platform:** Windows
**Arsitektur:** x86 (32-bit)
**Bahasa:** Visual Basic 6

## 1. Hash

| Algoritma | Nilai |
|---|---|
| SHA256 | `A3CDF254C9DAE4AE6D5EEF7FCBCD873BB5B7DD8A5E5286163D46EF13D0060473` |
| MD5 | `A05B1164FA00BA420C1CA95956B843A8` |

## 2. Tipe File

- **Format:** Portable Executable (PE)
- **Arsitektur:** x86:LE:32:default:windows (dikonfirmasi via Ghidra import dialog)
- **Nama file asli:** `Crack me 1.exe`
- **Runtime:** Visual Basic 6 (bergantung penuh pada `MSVBVM60.DLL`)

## 3. Strings (Hasil Ekstraksi)

Berikut string signifikan yang ditemukan dari hasil `strings`:

| String | Kategori | Keterangan |
|---|---|---|
| `MSVBVM60.DLL` | Runtime dependency | Runtime resmi Visual Basic 6, konfirmasi binary dikompilasi dengan VB6 |
| `VBA6.DLL` | Runtime dependency | Library VBA yang menyediakan fungsi-fungsi `__vba*` |
| `Crack me` / `Crack me 1` | Metadata proyek | Judul form/aplikasi |
| `Project1` | Metadata proyek | Nama default proyek VB6 (developer tidak mengganti nama proyek) |
| `Form1` | UI Component | Nama form utama |
| `Command1` (`&Check`) | UI Component | Tombol untuk memvalidasi serial |
| `Command2` (`&About`) | UI Component | Tombol info aplikasi |
| `Text1` | UI Component | Textbox tempat user memasukkan serial |
| `Label1` (`Serial :`) | UI Component | Label penanda kolom input serial |
| `C:\Program Files\VB98\VB6.OLB` | Development path | Path type library VB6 di komputer developer |
| `__vbaStrCmp` | Fungsi runtime VB | Membandingkan dua string |
| `__vbaVarMul` | Fungsi runtime VB | Perkalian antar variant (dipakai di rantai perkalian slot serial) |
| `__vbaVarTstEq` | Fungsi runtime VB | Membandingkan kesetaraan dua variant (dipakai di validasi akhir) |
| `_CIcos`, `_CIsin`, `_CItan`, `_CIatan`, `_CIsqrt`, `_CIlog`, `_CIexp` | Fungsi matematika VB runtime | Konfirmasi validasi serial memakai operasi trigonometri/logaritma floating point |
| `_adj_fdiv_m64`, `_adj_fprem`, dll | Floating-point adjustment | Fungsi bantu internal VB6 runtime untuk operasi FPU |

## 4. Import Table

| DLL | Fungsi yang Diimpor |
|---|---|
| `MSVBVM60.DLL` | `__vbaChkstk`, `__vbaExceptHandler`, `__vbaFPException`, `__vbaFreeObj`, `__vbaFreeStr`, `__vbaFreeVar`, `__vbaFreeVarList`, `__vbaHresultCheckObj`, `__vbaObjSet`, `__vbaStrCmp`, `__vbaVarMul`, `__vbaVarTstEq`, dan fungsi VB runtime lainnya |

![Import Table dari Ghidra Symbol Tree](abu-crackme-import-table.png)

## 5. Kesimpulan

Binary ini merupakan aplikasi Visual Basic 6 yang bergantung sepenuhnya
pada runtime `MSVBVM60.DLL` — hampir seluruh operasi (string, variant,
matematika) didelegasikan ke runtime tersebut, sehingga import table
terlihat ringkas meski logika program sebenarnya cukup kompleks.

Validasi serial number pada program ini **tidak berupa perbandingan
string/integer statis sederhana**, melainkan melibatkan rangkaian operasi
floating-point (fungsi trigonometri seperti `cos` dan `tan`) yang
dikombinasikan dari beberapa nilai, kemudian dikonversi ke representasi
heksadesimal sebelum dibandingkan dengan nilai target. Hal ini membuat
proteksinya jauh lebih sulit ditembus hanya lewat pembacaan statis
dibanding binary sederhana yang cuma membandingkan angka literal.

Ditemukan pula jejak informasi developer yang tidak dibersihkan sebelum
rilis, seperti nama proyek default (`Project1`, `Form1`) dan path lokal
(`C:\Program Files\VB98\VB6.OLB`) — indikasi bahwa binary ini adalah
hasil kompilasi langsung tanpa proses cleaning/hardening lebih lanjut.
