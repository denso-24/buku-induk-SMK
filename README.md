# 📘 Buku Induk SMK – Integrasi Google Apps Script + GitHub Pages

Aplikasi ini menyimpan data Buku Induk SMK langsung ke Google Drive menggunakan **Google Apps Script Web App**.  
Semua data (JSON + foto) tersimpan otomatis dalam folder sesuai NISN.

---

## 🚀 Cara Menggunakan

### 1. Upload ke GitHub
1. Unggah file **index_integrated.html** ke repository GitHub.
2. Rename menjadi **index.html** jika diperlukan.
3. Aktifkan GitHub Pages:
   - Settings → Pages  
   - Branch: `main`  
   - Folder: `/root` atau `/`  

Website akan tampil di:
```
https://<username>.github.io/<repository>/
```

---

## 🔧 Integrasi Google Apps Script

Web App URL Anda yang sudah ditanam di halaman:

```
https://script.google.com/macros/s/AKfycbw_RPAWDGV8DeCOGZaor8Bu1Gii0e7855230-2szqtXCYZX4IP9SwtMjS_t3gu8Ml81/exec
```

### Struktur penyimpanan di Google Drive:

```
Buku Induk SMK/
 └── <NISN>/
      ├── <NISN>_data.json
      └── foto_siswa.jpg/png
```

---

## 📦 File yang Disediakan

| File | Fungsi |
|------|--------|
| `index_integrated.html` | Form lengkap + upload ke Apps Script |
| `Code.gs` | Backend Apps Script penyimpanan ke Google Drive |
| `README.md` | Panduan instalasi & penggunaan |

---

## 🛠 Kode Apps Script (Backend)

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);

    const nisn = data.nisn || ("NISN_" + Date.now());
    const ROOT_FOLDER_NAME = "Buku Induk SMK";

    const rootFolder = getOrCreateFolder(ROOT_FOLDER_NAME);
    const folderSiswa = getOrCreateFolder(nisn, rootFolder);

    const jsonFileName = nisn + "_data.json";
    const jsonData = JSON.stringify(data, null, 2);
    folderSiswa.createFile(jsonFileName, jsonData, "application/json");

    if (data.foto && data.fotoNama && data.fotoMime) {
      const blob = Utilities.newBlob(
        Utilities.base64Decode(data.foto),
        data.fotoMime,
        data.fotoNama
      );
      folderSiswa.createFile(blob);
    }

    const result = {
      status: "success",
      message: "Data berhasil disimpan ke Google Drive",
      folderId: folderSiswa.getId(),
      folderUrl: "https://drive.google.com/drive/folders/" + folderSiswa.getId()
    };

    return ContentService
      .createTextOutput(JSON.stringify(result))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    const result = {
      status: "error",
      message: err.toString()
    };

    return ContentService
      .createTextOutput(JSON.stringify(result))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function getOrCreateFolder(name, parent) {
  const base = parent || DriveApp;
  const folders = base.getFoldersByName(name);
  if (folders.hasNext()) return folders.next();
  return base.createFolder(name);
}
```

---

## 🎉 Selesai!

Aplikasi siap digunakan. Jika Anda ingin:
- Generate PDF
- Upload banyak file (KK, Akta, Raport, dll.)
- Rekap ke Google Sheets
- Membuat dashboard admin

Cukup bilang **“Tambahkan fitur X”**.
