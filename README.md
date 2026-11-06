# DOTS vs MonoBehaviour — Gesture Zoom Dua Tangan di Meta Quest 3S

Testbed, perekam gesture, dataset, data hasil, dan skrip analisis untuk penelitian
**"Analisis Perbandingan Performa Pendekatan Object-Oriented dan Data-Oriented pada Pemrosesan Gesture Zoom Dua Tangan di Meta Quest VR"**
(Hibah Penelitian DIPA 2026, Fakultas Ilmu Komputer, Universitas Brawijaya).

Penelitian membandingkan dua implementasi yang setara fungsi:

| Versi | Pendekatan | Cara kerja |
|---|---|---|
| MonoBehaviour | Object-Oriented | Setiap cube adalah GameObject dengan `Update()` sendiri |
| DOTS/ECS | Data-Oriented | Cube adalah *entity*; deteksi oleh sistem Burst, skala oleh *job* paralel |

Kedua versi menerima input identik: dataset rekaman gesture (91 trial, 10.204 frame) diputar ulang satu baris per frame.
Beban diuji pada 1.000, 4.000, dan 8.000 cube, masing-masing 10 run per metode.

## Isi repositori

```
Assets/
├── Penelitian/                  testbed DOTS vs MonoBehaviour
│   ├── Scripts/Core/            dataset, detektor gesture, logger, stopwatch (LogicTimer)
│   ├── Scripts/Mono/            versi MonoBehaviour
│   ├── Scripts/DOTS/            versi DOTS/ECS
│   ├── Scripts/Runner/          runner otomatis (mode Pilot / Utama)
│   ├── Editor/                  menu "Penelitian → Setup Testbed"
│   ├── Scenes/                  Penelitian_Runner, Penelitian_Mono, Penelitian_DOTS
│   ├── P01_20260930_225934.csv  dataset gesture (responden P01)
│   ├── Hasil/Penelitian/        60 CSV pengujian utama + Pilot/ (16 CSV uji pendahuluan)
│   ├── Hasil/Analisis~/         hasil_statistik.xlsx, tabel CSV, grafik
│   └── Analysis~/               skrip Python: analisis.py, statistik.py, grafik.py
└── Recorder/                    perekam gesture (RecordingScene) + Dataset Inspector
```

Folder berakhiran `~` diabaikan oleh Unity (tidak diimpor sebagai aset).

## Kebutuhan

- Unity **6000.3.9f1** dengan modul **Android Build Support** (OpenJDK, Android SDK & NDK)
- Meta Quest 3S (atau Quest lain) dengan Mode Developer aktif, serta `adb`
- Python 3 dengan `pandas`, `numpy`, `scipy`, `statsmodels`, `matplotlib`, `openpyxl` (untuk analisis)

Paket Unity (Entities 1.5.0, Entities Graphics 1.5.0, OpenXR 1.17.1, Meta OpenXR 2.5.1, Input System 1.19.0, XR Interaction Toolkit 3.5.1)
dan seluruh pengaturan proyek sudah tersimpan di repositori, sehingga tidak perlu dipasang manual.

## Membuka proyek

1. Clone repositori, lalu **Unity Hub → Add → pilih folder ini**. Pilih platform **Android** saat membuka.
2. Tunggu impor pertama selesai (10–30 menit; Unity membangun ulang folder `Library`).
3. Bila Console menampilkan error terkait Burst, tutup dan buka ulang Unity sekali.

## Menjalankan pengujian

1. Buka `Assets/Penelitian/Scenes/Penelitian_Runner.unity`, pilih objek **BenchmarkRunner**, atur `mode`:
   - **Pilot** — sapuan 100–16.000 cube, 1 run singkat per metode (kalibrasi kategori beban)
   - **Utama** — 3 kategori × 10 run × 2 metode, dataset diputar penuh (±3,5 jam)
2. **File → Build Profiles → Build And Run** ke Quest.
3. Letakkan headset diam. Agar aplikasi tidak berhenti saat headset tidak dipakai:
   `adb shell am broadcast -a com.oculus.vrpowermanager.prox_close`
4. Data tersimpan di headset pada
   `/sdcard/Android/data/id.ac.ub.filkom.dotsvsmono/files/Penelitian/`
   dan dapat diambil dengan `adb pull`. Run yang berkasnya sudah ada otomatis dilewati.

## Analisis

```
cd Assets/Penelitian/Analysis~
python analisis.py ../Hasil/Penelitian          # ringkasan per run
python statistik.py ../Hasil/Penelitian         # uji asumsi, Mann-Whitney U, Kruskal-Wallis, ART ANOVA
python grafik.py ../Hasil/Penelitian            # box plot dan grafik interaksi
python analisis.py --pilot ../Hasil/Penelitian/Pilot
```

Keluaran ditulis ke folder data yang sama. Hasil analisis yang dilaporkan sudah tersedia di `Assets/Penelitian/Hasil/Analisis~/`.

## Metrik

| Metrik | Cara memperoleh |
|---|---|
| FPS | jumlah frame dibagi durasi pemutaran dataset |
| Frame time (median, p99) | selisih waktu awal fase Update antar frame |
| Waktu pemrosesan logika | stopwatch awal–akhir fase Update di PlayerLoop |
| Waktu persiapan rendering | stopwatch awal PreLateUpdate–akhir PostLateUpdate (main thread) |
| Waktu respons gesture | awal frame deteksi sampai akhir frame semua cube berubah skala |

## Merekam dataset baru

Buka `Assets/Recorder/Scenes/RecordingScene.unity`, build ke Quest, lalu rekam gesture dengan dua controller.
Berkas tersimpan di `persistentDataPath/Dataset/` pada headset. Periksa hasilnya lewat
**Tools → Gesture Benchmark → Dataset Inspector**.

## Penulis

Muhammad Aminul Akbar, S.Kom., M.T. (Ketua) · Ir. Issa Arwani, S.Kom., M.Sc. · Ir. Tri Afirianto, S.T., M.T.
Fakultas Ilmu Komputer, Universitas Brawijaya
