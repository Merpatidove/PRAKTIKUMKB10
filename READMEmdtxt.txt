# Algoritma Genetika - Knapsack Problem

Proyek ini adalah implementasi dari **Algoritma Genetika (Genetic Algorithm / GA)** untuk menyelesaikan **Knapsack Problem**, menggunakan antarmuka grafis Tkinter dan visualisasi matplotlib.

Berikut adalah penjelasan baris per baris atau bagian per bagian dari kode yang ada di `main.ipynb`.

---

## 1. Import Library
```python
import random
import matplotlib.pyplot as plt
import tkinter as tk
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
```
- `import random`: Digunakan untuk menghasilkan angka acak, yang sangat penting dalam Algoritma Genetika (untuk inisialisasi populasi, pemilihan titik crossover, dan mutasi).
- `import matplotlib.pyplot as plt`: Library untuk menggambar grafik (plot) yang akan menunjukkan perkembangan fitness terbaik dari generasi ke generasi.
- `import tkinter as tk`: Library standar Python untuk membuat antarmuka pengguna grafis (GUI).
- `from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg`: Modul ini digunakan untuk menempelkan (embed) grafik Matplotlib ke dalam jendela aplikasi Tkinter agar bisa tampil di GUI.

---

## 2. Inisialisasi Data dan Batasan
```python
BARANG = [
    ("Barang1", 10, 5),
    ("Barang2", 40, 4),
    ("Barang3", 30, 6),
    ("Barang4", 50, 3),
    ("Barang5", 35, 7)
]
MAKS_UKURAN = 15
```
- `BARANG`: Sebuah list of tuples yang merepresentasikan daftar barang yang tersedia. Setiap tuple berisi `(Nama Barang, Keuntungan, Ukuran/Berat)`.
- `MAKS_UKURAN = 15`: Menentukan kapasitas maksimal dari tas/gudang (Knapsack). Jika total ukuran barang yang dipilih melebihi batas 15 ini, maka solusi tersebut dianggap tidak valid (mendapatkan penalti).

---

## 3. Parameter Algoritma Genetika
```python
UKURAN_POPULASI = 10
JUMLAH_GENERASI = 25
MUTATION_RATE = 0.1
PANJANG_KROMOSOM = len(BARANG)
```
- `UKURAN_POPULASI = 10`: Menentukan jumlah individu (solusi potensial) dalam satu generasi. Ada 10 solusi acak per generasi.
- `JUMLAH_GENERASI = 25`: Algoritma akan melakukan iterasi pencarian evolusi sebanyak 25 kali (generasi).
- `MUTATION_RATE = 0.1`: Peluang mutasi terjadi pada kromosom keturunan (child) ditetapkan sebesar 10% (0.1).
- `PANJANG_KROMOSOM`: Jumlah sel gen dalam satu kromosom, disesuaikan agar sama persis dengan jumlah pilihan barang yang ada (`len(BARANG)` bernilai 5).

---

## 4. Fungsi Evaluasi / `hitung_fitness`
```python
def hitung_fitness(kromosom):
    total_keuntungan = 0
    total_ukuran = 0
    
    for i in range(PANJANG_KROMOSOM):
        if kromosom[i] == 1:
            total_keuntungan += BARANG[i][1]
            total_ukuran += BARANG[i][2]
            
    if total_ukuran > MAKS_UKURAN:
        return 0 
    return total_keuntungan
```
- Tujuan fungsi ini adalah mengevaluasi seberapa bagus dan solutif sebuah kromosom (individu). Semakin besar, semakin baik nilainya.
- `for i in range(PANJANG_KROMOSOM):`: Melakukan perulangan untuk mengecek setiap gen di dalam array kromosom.
- `if kromosom[i] == 1:`: Memeriksa apakah barang ke-`i` diambil. Kode `1` berarti dibawa, `0` berarti ditinggal. Jika dibawa, tambahkan nilai keuntungan dan ukurannya.
- `if total_ukuran > MAKS_UKURAN:`: Ini adalah fungsi *Penalty*. Jika total ukuran barang melebihi 15, solusinya dianggap buruk sehingga Fitness-nya dihancurkan menjadi `0` dan otomatis gugur.
- Jika memenuhi standar, fungsi mengembalikan *total keuntungan* sebagai nilai **Fitness**.

---

## 5. Fungsi Inisialisasi Populasi
```python
def inisialisasi_populasi(ukuran):
    return [[random.randint(0, 1) for _ in range(PANJANG_KROMOSOM)] for _ in range(ukuran)]
```
- Menghasilkan sekumpulan populasi individu awal (Generasi Pertama).
- Fungsi mengaplikasikan perulangan dua dimensi (List Comprehension). Gen diisi nilai bilangan bulat acak `0` atau `1` yang dihasilkan melalui `random.randint(0, 1)`.

---

## 6. Fungsi Seleksi (Roulette Wheel Selection)
```python
def seleksi_rws(populasi, fitness_populasi):
    total_fitness = sum(fitness_populasi)
    if total_fitness == 0:
        return random.choice(populasi)
    
    pick = random.uniform(0, total_fitness)
    current = 0
    for i, fitness in enumerate(fitness_populasi):
        current += fitness
        if current > pick:
            return populasi[i]
    return populasi[-1]
```
- Berfungsi memilih indukan (parent) terbaik dengan metode RWS.
- `total_fitness`: Penjumlahan kumulatif dari seluruh nilai fitness dalam populasi tersebut.
- `if total_fitness == 0`: *Safety Check*, jika tidak ada satupun yang selamat (misal semua fitness `0`), induk diambil acak bebas.
- `pick = random.uniform(0, total_fitness)`: Memutar panah rolet dan mendapatkan target angka *pick*.
- Di dalam perulangan `for`, algoritma menumpuk bobot (`current`) hingga menyentuh atau melampaui `pick`. Jika melebihi `pick`, individu tersebutlah yang memenangi rolet dan dijadikan indukan.

---

## 7. Fungsi Crossover (Two-Point Crossover)
```python
def crossover_two_point(parent1, parent2):
    if PANJANG_KROMOSOM < 3:
        return parent1[:], parent2[:]
        
    titik1 = random.randint(1, PANJANG_KROMOSOM - 2)
    titik2 = random.randint(titik1 + 1, PANJANG_KROMOSOM - 1)
    
    child1 = parent1[:titik1] + parent2[titik1:titik2] + parent1[titik2:]
    child2 = parent2[:titik1] + parent1[titik1:titik2] + parent2[titik2:]
    
    return child1, child2
```
- Melakukan persilangan dari dua indukan menjadi dua keturunan (anak) baru yang membawa genetika campuran.
- Memilih 2 garis pembatas silang secara acak (`titik1` dan `titik2`).
- Blok genetik di pertengahan batas (`[titik1:titik2]`) tersebut saling dipertukarkan. 
- Bagian sisi awal dan akhirnya tetap mengambil gen induk asalnya (`child1` = awal p1 + tengah p2 + akhir p1).

---

## 8. Fungsi Mutasi (Inversion Mutation)
```python
def mutasi_inversion(kromosom, rate):
    if random.random() < rate:
        titik1 = random.randint(0, PANJANG_KROMOSOM - 2)
        titik2 = random.randint(titik1 + 1, PANJANG_KROMOSOM - 1)
        kromosom[titik1:titik2+1] = kromosom[titik1:titik2+1][::-1]
    return kromosom
```
- Metode mutasi inversi ini menambahkan unsur diversifikasi, berguna untuk mencegah sistem macet di solusi Optimum Lokal (*Local Optima*).
- `random.random() < rate`: Terjadi peluang perulangan mutasi sebesar `rate` (10%).
- Jika kondisi terpenuhi, potong segmen array acak (`titik1` hingga `titik2`).
- `[::-1]`: Membalik total (reverse/inversi) urutan blok array dari segmen potongan tersebut (misal `0,1,1` di-inversi menjadi `1,1,0`).

---

## 9. Struktur Tkinter GUI & Proses Generasi Eksekusi Utama
```python
class KnapsackGUI:
    def __init__(self, root): ...
```
- `__init__` mendirikan kerangka dasar jendela aplikasi Tkinter (`root`), memberikan judul dan ukuran `650x650`.
- Kemudian merender tombol Button di UI dan menyediakan `self.canvas` dari FigureCanvasTkAgg agar grafiknya menempel ke program.

```python
    def jalankan_ga(self):
        populasi = inisialisasi_populasi(UKURAN_POPULASI)
        riwayat_terbaik = []
        kromosom_terbaik_global = []
        fitness_terbaik_global = -1
```
- Di sinilah loop perhitungan GA berjalan saat "Klik Jalankan". 
- Populasi generasi awal dibuat. Variabel untuk merangkum riwayat terbaik dan kromosom final (`kromosom_terbaik_global`) disiapkan di awal dengan nilai -1.

```python
        for generasi in range(JUMLAH_GENERASI):
            fitness_populasi = [hitung_fitness(ind) for ind in populasi]
            # --- evaluasi Max dari perulangan gen ---
            # ...
```
- Terjadi **Iterasi Evolusi** sepanjang `JUMLAH_GENERASI` kali berulang.
- List comprehension digunakan untuk memeriksa Fitness dari tiap solusi dalam populasi. Nilai terbaik akan dijaring dan dicatat sejarah grafiknya ke array `riwayat_terbaik`.

```python
            populasi_baru = []
            while len(populasi_baru) < UKURAN_POPULASI:
                p1 = seleksi_rws(populasi, fitness_populasi)
                p2 = seleksi_rws(populasi, fitness_populasi)
                
                c1, c2 = crossover_two_point(p1, p2)
                
                c1 = mutasi_inversion(c1, MUTATION_RATE)
                c2 = mutasi_inversion(c2, MUTATION_RATE)
                
                populasi_baru.extend([c1, c2])
                
            populasi = populasi_baru[:UKURAN_POPULASI]
```
- **Proses Reproduksi**: Dalam blok *while*, dua induk dipilih menggunakan rolet -> dua keturunan dihasilkan dari *crossover* -> keturunan berpeluang dimutasi (inversi).
- Setelah penuh, status `populasi` yang usang dioverwrite/diganti oleh entitas generasi `populasi_baru` untuk loop iterasi tahun / iterasi generasi berikutnya.

```python
        # --- kalkulasi penutup ---
        teks_hasil = ( f"--- HASIL OPTIMAL ---\n ...")
        self.lbl_hasil.config(text=teks_hasil)
        
        # Render Plot Grafik
        self.ax.clear()
        self.ax.plot(range(1, JUMLAH_GENERASI + 1), riwayat_terbaik, marker='o', color='crimson')
        # ...
        self.canvas.draw()
```
- Setelah siklus generasi berakhir, logika menelusuri balik *array barang apa saja yang terpilih* dari array kromosom final yang paling untung.
- Teks Output UI ditayangkan.
- Grafik Matplotlib dihapus lalu digambar garis plot kemajuannya `self.ax.plot(..)`. `self.canvas.draw()` memicu *refresh layar grafis*.

```python
if __name__ == "__main__":
    root = tk.Tk()
    app = KnapsackGUI(root)
    root.mainloop()
```
- Kode pemicu *entry-point*. `root.mainloop()` membuat UI tidak akan terputus alias terjebak dalam eksekusi layar hingga user secara sadar menekan X (Tutup Windows).