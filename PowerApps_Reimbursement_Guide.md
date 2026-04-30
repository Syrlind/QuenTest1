# Panduan Aplikasi Reimbursement Sederhana - Power Apps

Dokumen ini berisi langkah-langkah dan kode formula (Power Fx) untuk membuat aplikasi reimbursement karyawan.

## 1. Persiapan Data (SharePoint List / Dataverse)

Buatlah sebuah List di SharePoint dengan nama **"ReimbursementRequests"** yang memiliki kolom berikut:

| Nama Kolom | Tipe Data | Keterangan |
| :--- | :--- | :--- |
| `Title` | Single line of text | Judul pengeluaran (Otomatis) |
| `EmployeeName` | Person or Group | Nama Karyawan |
| `ExpenseDate` | Date and Time | Tanggal pengeluaran |
| `Category` | Choice | Kategori (Transport, Makan, Akomodasi, Lain-lain) |
| `Amount` | Currency | Jumlah uang |
| `ReceiptImage` | Image | Foto struk/bukti bayar |
| `Status` | Choice | Pending, Approved, Rejected |
| `ManagerComment` | Multiple lines of text | Komentar manajer |

---

## 2. Struktur Aplikasi (Canvas App)

Buat Canvas App baru di Power Apps, hubungkan ke data source **ReimbursementRequests**. Buat 3 layar (Screen):

### Layar 1: BrowseScreen1 (Daftar Pengajuan)
Tampilan daftar riwayat reimbursement karyawan.

**Komponen:**
- **Gallery1**: Menampilkan data.
  - *Items Property*: `SortByColumns(ReimbursementRequests, "Created", SortOrder.Descending)`
- **Button_New**: Tombol "+" untuk pengajuan baru.
  - *OnSelect*: `NewForm(Form1); Navigate(ScreenInput, ScreenTransition.Fade)`
- **Icon_Details**: Ikon panah di dalam galeri.
  - *OnSelect*: `Navigate(ScreenDetail, ScreenTransition.Fade); Set(varSelectedItem, ThisItem)`

### Layar 2: ScreenInput (Form Input)
Tampilan untuk mengisi form reimbursement baru.

**Komponen:**
- **Form1**: Form edit/create.
  - *DataSource*: `ReimbursementRequests`
  - *Mode*: `FormMode.New`
  - *Fields*: EmployeeName, ExpenseDate, Category, Amount, ReceiptImage.
  
- **Button_Submit**: Tombol Simpan.
  - *OnSelect*:
    ```powerfx
    SubmitForm(Form1);
    If(
        Form1.Error = NoError,
        Notify("Pengajuan berhasil dikirim!", NotificationType.Success);
        Navigate(BrowseScreen1, ScreenTransition.Fade),
        Notify("Gagal mengirim data.", NotificationType.Error)
    )
    ```

- **Button_Cancel**: Tombol Batal.
  - *OnSelect*: `Navigate(BrowseScreen1, ScreenTransition.Fade)`

### Layar 3: ScreenDetail (Detail & Approval)
Tampilan detail untuk melihat status dan (opsional) approval manajer.

**Komponen:**
- **Label_Title**: Menampilkan judul.
  - *Text*: `varSelectedItem.Title`
- **Label_Amount**: Menampilkan jumlah.
  - *Text*: `Text(varSelectedItem.Amount, "Rp [#.##0]")`
- **Image_Receipt**: Menampilkan foto struk.
  - *Image*: `varSelectedItem.ReceiptImage`
- **Label_Status**: Status saat ini.
  - *Text*: `varSelectedItem.Status.Value`
  - *Color*: 
    ```powerfx
    If(
        varSelectedItem.Status.Value = "Approved", Green,
        varSelectedItem.Status.Value = "Rejected", Red,
        Orange
    )
    ```

- **Dropdown_Status** (Hanya untuk Manajer):
  - *Items*: `["Pending", "Approved", "Rejected"]`
  - *Default*: `varSelectedItem.Status.Value`

- **Button_Approve**: Tombol simpan keputusan.
  - *OnSelect*:
    ```powerfx
    Patch(
        ReimbursementRequests,
        varSelectedItem,
        {
            Status: Value(Dropdown_Status.SelectedText.Value),
            ManagerComment: TextInput_Comment.Text
        }
    );
    Notify("Status diperbarui.", NotificationType.Success);
    Navigate(BrowseScreen1, ScreenTransition.Fade)
    ```

---

## 3. Logika Tambahan (Tips)

### Validasi Input
Pada properti **DisplayMode** tombol Submit, pastikan semua field terisi:
```powerfx
If(
    IsBlank(DataCardValue_EmployeeName.Text) || 
    IsBlank(DataCardValue_Amount.Text) || 
    IsBlank(DataCardValue_Category.Selected.Value),
    DisplayMode.Disabled,
    DisplayMode.Edit
)
```

### Format Mata Uang Rupiah
Gunakan fungsi `Text` untuk memformat angka menjadi Rupiah:
```powerfx
Text(ThisItem.Amount, "Rp [#.##0]", "id-ID")
```

### Filter Data Berdasarkan User Login
Agar karyawan hanya melihat data mereka sendiri (pada Gallery1):
```powerfx
Filter(
    ReimbursementRequests,
    EmployeeName.Email = User().Email
)
```

---

## Cara Menggunakan Kode Ini:
1. Buka [make.powerapps.com](https://make.powerapps.com).
2. Pilih **Create** > **Canvas App from blank**.
3. Hubungkan konektor **SharePoint** dan pilih list yang sudah dibuat.
4. Salin rumus-rumus di atas ke dalam properti komponen yang sesuai di panel Properties sebelah kanan.
