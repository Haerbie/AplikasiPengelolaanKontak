# Tugas 3: Aplikasi Pengelolaan Kontak

### Pembuat
- **Nama**: Muhammad Irwan Habibie
- **NPM**: 2210010461

---

## 1. Deskripsi Program
Aplikasi ini memungkinkan pengguna untuk:
- Mengelola kontak dengan menambah, mengedit, menghapus, dan menampilkan daftar kontak.
- Menyimpan daftar kontak ke dalam file CSV dan memuat data kontak dari file CSV.
- Mencari kontak berdasarkan nama atau nomor telepon.
- Memastikan nomor telepon yang dimasukkan hanya berupa angka dengan panjang yang sesuai.

## 2. Komponen GUI
- **JFrame**: Window utama aplikasi.
- **JPanel**: Panel untuk menampung komponen.
- **JLabel**: Label untuk menampilkan informasi input.
- **JTextField**: Input untuk nama, nomor telepon, kategori, dan pencarian.
- **JButton**: Tombol untuk menambah, mengedit, menghapus, mencari, mengekspor, dan mengimpor kontak.
- **JComboBox**: Dropdown untuk memilih kategori kontak.
- **JTable**: Tabel untuk menampilkan daftar kontak.

## 3. Logika Program
- Menggunakan database SQLite untuk menyimpan data kontak secara lokal.
- Menampilkan daftar kontak di **JTable**.
- Menyimpan dan memuat daftar kontak dari file CSV.
- Validasi nomor telepon agar hanya berupa angka dan memiliki panjang yang sesuai.

## 4. Events
Menggunakan **ActionListener** untuk menangani interaksi pengguna:

### A. Tombol Tambah Kontak
Menambahkan kontak baru berdasarkan input dari pengguna.

private void btnTambahActionPerformed(java.awt.event.ActionEvent evt) {                                          
    String nama = txtNama.getText();
    String noTelepon = txtNoTelepon.getText();
    String kategori = (String) cmbKategori.getSelectedItem();

    if (nama.isEmpty() || noTelepon.isEmpty() || kategori == null) {
        JOptionPane.showMessageDialog(this, "Semua data harus diisi!", "Error", JOptionPane.ERROR_MESSAGE);
        return;
    }

    if (noTelepon.length() < 10 || noTelepon.length() > 13) {
        JOptionPane.showMessageDialog(this, "Nomor telepon harus memiliki panjang antara 10 hingga 13 digit.", "Error", JOptionPane.ERROR_MESSAGE);
        return;
    }

    PengelolaDatabase.tambahKontak(nama, noTelepon, kategori);

    txtNama.setText("");
    txtNoTelepon.setText("");
    cmbKategori.setSelectedIndex(0);

    JOptionPane.showMessageDialog(this, "Kontak berhasil ditambahkan!");
    tampilkanKontak();
}

### B. Tombol Cari Kontak
Mencari kontak berdasarkan nama atau nomor telepon yang dimasukkan dan menampilkan hasil di **JTable**.

private void btnCariActionPerformed(java.awt.event.ActionEvent evt) {                                        
    String keyword = txtCari.getText().trim();

    if (keyword.isEmpty()) {
        JOptionPane.showMessageDialog(this, "Masukkan nama atau nomor telepon untuk mencari.", "Error", JOptionPane.ERROR_MESSAGE);
        return;
    }

    List<Map<String, String>> hasilPencarian = PengelolaDatabase.cariKontak(keyword);
    DefaultTableModel model = (DefaultTableModel) jTableKontak.getModel();
    model.setRowCount(0);

    for (Map<String, String> data : hasilPencarian) {
        model.addRow(new Object[]{
            data.get("id"),
            data.get("nama"),
            data.get("no_telepon"),
            data.get("kategori")
        });
    }
}

## 5. Variasi
Aplikasi ini memiliki variasi tambahan berikut:

### A. Pencarian Kontak
Pengguna dapat mencari kontak berdasarkan nama atau nomor telepon. Hasil pencarian akan ditampilkan di **JTable**.

private void btnCariActionPerformed(java.awt.event.ActionEvent evt) {                                        
    String keyword = txtCari.getText().trim();

    if (keyword.isEmpty()) {
        JOptionPane.showMessageDialog(this, "Masukkan nama atau nomor telepon untuk mencari.", "Error", JOptionPane.ERROR_MESSAGE);
        return;
    }

    List<Map<String, String>> hasilPencarian = PengelolaDatabase.cariKontak(keyword);
    DefaultTableModel model = (DefaultTableModel) jTableKontak.getModel();
    model.setRowCount(0);

    for (Map<String, String> data : hasilPencarian) {
        model.addRow(new Object[]{
            data.get("id"),
            data.get("nama"),
            data.get("no_telepon"),
            data.get("kategori")
        });
    }
}

### B. Validasi Input Nomor Telepon
Nomor telepon divalidasi agar hanya berisi angka dengan panjang 10 hingga 13 digit.

private void txtNoTeleponKeyTyped(java.awt.event.KeyEvent evt) {                                      
    char c = evt.getKeyChar();
    if (!Character.isDigit(c)) {
        evt.consume();
        JOptionPane.showMessageDialog(null, "Masukkan hanya angka untuk nomor telepon.");
    }
}

### C. Ekspor Data ke CSV
Aplikasi dapat mengekspor data kontak ke dalam file CSV.

private void btnExportCSVActionPerformed(java.awt.event.ActionEvent evt) {                                             
    try {
        FileWriter csvWriter = new FileWriter("daftar_kontak.csv");
        csvWriter.append("ID, Nama, Nomor Telepon, Kategori\n");

        DefaultTableModel model = (DefaultTableModel) jTableKontak.getModel();
        for (int i = 0; i < model.getRowCount(); i++) {
            csvWriter.append(model.getValueAt(i, 0).toString()).append(",");
            csvWriter.append(model.getValueAt(i, 1).toString()).append(",");
            csvWriter.append(model.getValueAt(i, 2).toString()).append(",");
            csvWriter.append(model.getValueAt(i, 3).toString()).append("\n");
        }

        csvWriter.flush();
        csvWriter.close();

        JOptionPane.showMessageDialog(this, "Data berhasil diekspor ke daftar_kontak.csv");
    } catch (IOException e) {
        JOptionPane.showMessageDialog(this, "Error saat mengekspor data: " + e.getMessage());
    }
}

### D. Impor Data dari CSV
Aplikasi dapat mengimpor data kontak dari file CSV.

private void importFromCSVActionPerformed(java.awt.event.ActionEvent evt) {                                              
    JFileChooser fileChooser = new JFileChooser();
    fileChooser.setDialogTitle("Pilih file CSV untuk diimpor");
    int result = fileChooser.showOpenDialog(this);

    if (result == JFileChooser.APPROVE_OPTION) {
        File selectedFile = fileChooser.getSelectedFile();

        try (BufferedReader br = new BufferedReader(new FileReader(selectedFile))) {
            String line;
            br.readLine();

            while ((line = br.readLine()) != null) {
                String[] values = line.split(",");
                if (values.length >= 3) {
                    String nama = values[1].trim();
                    String noTelepon = values[2].trim();
                    String kategori = values[3].trim();

                    PengelolaDatabase.tambahKontak(nama, noTelepon, kategori);
                }
            }

            JOptionPane.showMessageDialog(this, "Data berhasil diimpor dari " + selectedFile.getName());
            tampilkanKontak();
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Error saat mengimpor data: " + e.getMessage());
        }
    }
}

## 6. Tampilan Saat di Jalankan

![TampilanPengelolaanKontak](https://github.com/user-attachments/assets/f07df8f2-11b8-4022-9ada-fc9eabdc5fe4)

## 7. Indikator Penilaian

| No  | Komponen          | Persentase |
| :-: | ------------------ | :--------: |
|  1  | Komponen GUI      |     15%    |
|  2  | Logika Program    |     20%    |
|  3  | Events            |     10%    |
|  4  | Kesesuaian UI     |     15%    |
|  5  | Memenuhi Variasi  |     40%    |
|     | **TOTAL**         |  **100%**  |
