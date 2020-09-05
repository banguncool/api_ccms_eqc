# CCMS API Integrasi untuk EQC

## CCMS Hostname & Port (Pasuruan - Japanan)
- IP CCMS 192.168.38.2
- Port Integrasi 9500
- Token key "CCMS_EQC_2F"


## Format Token
Format token => md5([KEY]-[YYYY]/[MM]/[DD])
Keterangan:
- KEY => Kode unik, ini berbeda disetiap tempat
- YYYY => Tahun (4 digit)
- MM => Bulan (2 digit)
- DD => Hari (2 digit)

Contoh: md5(CCMS_EQC_2F-2020/08/15)\
Hasil: 4797d2b1e24612178b624c68afb979bd


## Format URL
IP akan berbeda beda disetiap tempat
Format URL sebagai berikut
```sh
http://[hostname]:[port]/eqc/[perintah]/[token]
```
Contoh:
```sh
http://192.168.38.2:9500/eqc/insert/4797d2b1e24612178b624c68afb979bd
```

## Respon status
Setelah request data akan mendapatkan respon feedback.\
**Struktur Data** 
```json
{
  "status": "success",
  "data": []
}
```

Keterangan:
- status (string) => status pesan feedback
- data (any)

Berikut daftar respon dalam (string):
- success => berhasil
- json_error => format request json error
- error => internal error
- track_not_available => jalur tidak tersedia
- no_data => tidak ada data saat menginput
- token_invalid => token salah
- url_not_found => format url salah


## Daftar Track
Setiap tempat mungkin tidak semua jalur sudah terpasang Batching Plant CCMS, atau Jalur sedang perbaikan dalam jangka waktu lama. Daftar track berfungsi untuk mendeteksi jalur mana saja yang siap.

**Tujuan** 
- Request: POST
- Perintah: get_track


**Struktur Data** \
_daftar track tidak memerlukan data apapun_

**Respon**
```json
{
  "status": "success",
  "data": [1, 2, 5, 6]
}
```
Keterangan:
- status (string) => status respon
- data (array) => nomor jalur yang aktif


## Input Data Mix Design
Mix Design adalah parameter dari Batching Plant, ketika diinput secara ototmatis parameter dari Batching Plant akan berubah saat itu juga walaupun sedang beroperasi.

**Tujuan** 
- Request: POST
- Perintah: insert

**Struktur Data** 
```json
{
  "track": 1,
  "data": [{
    "mix_code": "K600NFAGN",
    "mix_design": "k-600",
    "mix_class": "Tiang Pancang",
    "load": 862,
    "density": 2425,
    "wc_ratio": 0.4,
    "slump": 12,
    "target": [
      {
        "name": "Semen",
        "code": "1234",
        "target": 1234,
        "density": 12.34
      },
      {
        "name": "Fly Ash",
        "code": "5678",
        "target": 5678,
        "density": 56.78
      },
      {
          // dan seterusnya
      }
    ]
  },
  {
    // dan seterusnya
  }]
}
```

Keterangan:
- mix_code (string) => kode mix
- mix_design (string) => target mix_design
- mix_class (string) => Nama spesifik mix_design
- load (int) => kuat tekan untuk target uji tekan dalam (kN)
- data -> density (int) => berat jenis per material 
- wc_ratio (float) => pebandingan target air dibagi target semen
- slump (int) => keenceran beton dalam (cm)
- track (number) => Jalur mana yang akan diubah parameternya
- data (array object) => data dari parameter
- name (string) => nama material
- code (string) => kode sumber daya
- target (int) => target (kg) material dalam 1 m³
- target -> density (float) => berat jenis

Catatan:
- Field data adalah target semua mix yang nanti jadi pilihan operator Batching Plant

**Respon**
```json
{
  "status": "success"
}
```
Keterangan:
- status (string) => status respon 

## Pencarian data
Data yang sudah dikirim oleh CCMS ke EQC dapat dicari kembali.

**Tujuan** 
- Request: POST
- Perintah: get_data


**Struktur Data** 
```json
{
  "start": 1597634345935,
  "stop": 1597634545935,
  "press_id": "2F012020080300404012",
  "batch_id": "2F01202008030040",
  "track": [1, 4, 6],
  "shift": [1, 2, 3],  
}
```
Keterangan:
- start (timestamp) => waktu dan tanggal awal (wajib di isi)
- stop (timestamp) => waktu dan tanggal akhir (wajib di isi)
- press_id (string) => ID uji tekan (optional, jangan diserta)
- batch_id (string) => ID Batching Plant (optional)
- track (array int) => jalur, dapat terdiri dari beberapa jalur (optional)
- shift (array int) => shift, dapat terdiri dari beberapa shift (optional)


**Respon**
```json
{
  "status": "success",
  "data": [
    {
      "press_id": "2f2002271506045896",  
      "batch_id": "23498235022839",  
      "mix_design": "K-300",   
      "target_mix_class": "Slump 2-3 cm", 
      "load": 350,
      "date": 1581677835000,          
      "track": 1, 
      "shift": 1,

      "start": 1582792340071,   
      "stop": 1582792352025,   
      "actual_mix_class": 1.3,    
      "actual_day": 7,
      "type": "Silinder",
      "name": "Silinder 1", 
      "diameter": 120, 
      "width": 150,
      "height": "320",
      
      "npp": ["abcde", "abcde", "abcde"], 
      "spprb": ["abcde", "abcde", "abcde"],
      "project": ["PT. Oke", "PT. Oke", "PT. Oke"],
      
      "chart": [1, 2, 3, 4, 5],
      "test_day": 7,
      "test_load_target": 350, 
      "test_load_actual": 0,
      "note": "Ok",
    },
    {
      ... // dan seterusnya
    }
  ]
}
```
Keterangan:
- status (string) => status respon
- data (array object) => data pencarian



# OUTPUT untuk EQC

## Data Uji Tekan
Setiap selesai uji tekan akan menghasilkan data dan langsung dikirim ke EQC.

**Struktur Data**
```json
{
  "press_id": "2f2002271506045896",  
  "batch_id": "23498235022839",  
  "mix_design": "K-300",   
  "target_mix_class": "Slump 2-3 cm", 
  "load": 350,
  "date": 1581677835000,          
  "track": 1, 
  "shift": 1,

  "start": 1582792340071,   
  "stop": 1582792352025,   
  "actual_mix_class": 1.3,    
  "actual_day": 7,
  "type": "Silinder",
  "name": "Silinder 1", 
  "diameter": 120, 
  "width": 150,
  "height": "320",
  
  "npp": ["abcde", "abcde", "abcde"], 
  "spprb": ["abcde", "abcde", "abcde"],
  "project": ["PT. Oke", "PT. Oke", "PT. Oke"],
  
  "chart": [1, 2, 3, 4, 5],
  "test_day": 7,
  "test_load_target": 350, 
  "test_load_actual": 0,
  "note": "Ok",
}
```
Keterangan:
- press_id (string) => ID uji tekan
- batch_id (string) => ID Batching plant (adukan)
- mix_desgn (string) => mutu (batching plant)
- target_mix_class (string) => target slump (batching plant)
- load (int) => target uji tekan dalam kN (batching plant)
- date (timestamp) => tanggal adukan
- track (int) => jalur produksi
- shift (int) => shift produksi
- start (timestamp) => waktu start uji tekan
- stop (timestamp) => waktu stop uji tekan
- actual_mix_class (float) => aktual slum (input uji tekan)
- actual_day (int) => target hari untuk test (input uji tekan)
- type (string) => tipe benda uji "Silinder" atau "Kubus"
- name (string) => nama tipe benda uji (input uji tekan)
- diameter (int) => satuan dalam (mm) diameter benda uji, hanya tersedia jika benda uji "Silinder"
- width (int) => satuan dalam (mm) lebar benda uji, hanya tersedia jika benda uji "Kubus"
- height (int) => satuan dalam (mm) tinggi benda uji, tersedia jika benda uji "Silinder" atau "Kubus"
- npp (array string) => npp produk dalam uji tekan pada shift tersebut 
- spprb (array string) => spprb produk dalam uji tekan pada shift tersebut 
- proyek (array string) => proyek produk dalam uji tekan pada shift tersebut 
- chart (array float) => grafik uji tekan 200 ms dalam setiap array, array terakhir sama waktunya dengan stop uji tekan
- test_day (int) => hari saat pengujian dengan acuan tanggal adukan batching plant (adukan hari yang sama dihitung 1 hari, dan seterusnya)
- test_load_target (int) => satuan dalam (kN), beban target pada berdasarkan perhitungan jumlah hari dari adukan sampai pengujian
- test_load_actual (int) => satuan dalam (kN), beban aktual asat uji tekan
- note (string) => keterangan operator
