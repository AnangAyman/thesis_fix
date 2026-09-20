# Review Draft Paper — Apa yang Perlu Dibenerin

**Verdict:** struktur argumen dan flow-nya udah bener. Masalahnya draft ini **campuran dua versi paper** — banyak angka, tabel, dan kalimat dari run lama yang belum diupdate, dan sebagian bertentangan langsung sama tabel di halaman yang sama.

Urutan kerja: benerin angka dulu (Bagian 1–3), baru yang lain.

---

## 1. 🔴 Angka yang bertentangan

Ini bukan soal gaya. Ini kontradiksi internal yang bakal langsung ketauan reviewer.

| # | Lokasi | Ditulis | Seharusnya | Catatan |
|---|---|---|---|---|
| 1 | Abstract | 319.8 KB | **371.4 KB** | Table IV |
| 2 | Abstract | LOSO ~67% | **71.67%** | Sec. IV-E |
| 3 | Abstract | "latency (unknown value) ms" | — | Placeholder |
| 4 | Abstract | "accuracy reached is approximately" | — | Grammar |
| 5 | IV-C ¶2 | "319.8 KB, 71.2% smaller" | **371.4 KB, 52.1%** | Bertentangan sama Table IV di halaman yang sama |
| 6 | IV-C ¶4 | INT8 "99.09%, 346.5 KB" | **98.41%, 998.7 KB** | Dua-duanya salah |
| 7 | IV-C ¶5 | "319.8 KB Dynamic Range artifact" | **371.4 KB** | |
| 8 | **Table II** | 279,176 params | **144,296** | Lihat Bagian 2 |
| 9 | III-C-2 | "Input shape (30, 447), 149 landmarks" | **(30, 142)** | Model final ABC |
| 10 | III-C-2 | GRU 32 unit → 256 unit | **128 → 64** | Fig. 4 yang bener |
| 11 | III-C-2 | "narrow-to-wide configuration" | **wide-to-narrow** | Ikut #10 |
| 12 | III-C-3 | Early stopping patience 10 | vs IV-B-1 bilang 15 | Patience = 10 |
| 13 | IV-B-1 | "epoch 80", "epoch-65 weights" | — | Cek di train_final.ipynb jas |
| 14 | IV-B-1 | Validation 98.64% | — | Ini bener kok ignore aja |
| 15 | IV-B-2 | Wilson CI 98.01–99.77% | **≈97.06–99.37%** | CI lama dihitung buat 437/440 |
| 16 | IV-B-2 | "three off-diagonal predictions" | **enam** | 434/440 = 6 error |
| 17 | IV-B-2 | Pair baik–apa, berapa–dia, mereka–tidur | — | Dari confusion matrix lama. Cek train_final.ipynb jas |
| 18 | IV-D | "mean standard deviation" | **mean ± standard deviation** | |
| 19 | Table IV kolom Inf. (ms) | 1.029 / 1.331 / dst | — | Angka N=1 lama, tapi IV-D klaim N=10 |
| 20 | Fig. 9 legend | "permuted control 0.0%" | **≈99%** | Plot bug |
| 21 | III-B-5 | "final compact 146 values" | **142 (ABC)** | Model final pakai 3 blok. Lihat Bagian 8 |
| 22 | II-E | "149 selected hand, face, and pose landmarks" | **142 kinematic features** | |
| 23 | Conclusion | "do not yet establish generalization to unseen signers, quantify the contribution of facial landmarks" | — | Dua-duanya udah dikerjain di paper ini |

---

## 2. 🔴 Table II salah arsitektur

Table II masih arsitektur lama (32/256 GRU, 279,176 params). Tapi Conclusion nyebut **144,296 params**, dan Fig. 4 nunjukin **128/64**.

Cek aritmatiknya — 128/64 dengan input 142:

| Layer | Hitungan | Params |
|---|---|---|
| GRU-1 (128) | 3 × (142×128 + 128×128 + 2×128) | 104,448 |
| GRU-2 (64) | 3 × (128×64 + 64×64 + 2×64) | 37,248 |
| Dense (40) | 64×40 + 40 | 2,600 |
| **Total** | | **144,296** ✅ |

Cocok persis. Jadi **Fig. 4 dan Conclusion yang bener, Table II dan Fig. 7 yang stale.**

Yang harus dilakukan:

- Regenerate Table II (128/64, input 142, total 144,296)
- Regenerate Fig. 7 — sekarang masih `(None, 30, 447)`, 32/256, dropout 0.3
- Benerin III-C-2: unit count, input shape, hapus "narrow-to-wide"
- Samain dropout: teks 0.4, Fig. 4 0.4, Fig. 7 0.3 → pakai **0.4**

✅ **Dikonfirmasi:** arsitektur 128/64 dipakai di keempat arm ablation, sama dengan model final. Jadi perbandingan antar arm apple-to-apple, dan nggak perlu caveat soal arsitektur seleksi beda dari arsitektur final.

---

## 3. 🔴 Sisa editing yang belum dibersihin

| Lokasi | Masalah |
|---|---|
| III-A-3, bullet Horizontal Mirroring | Isinya instruksi ke diri sendiri: *"Add a new bullet point for Horizontal Mirroring. Your code applies a 50% probability (MIRRORPROB = 0.5)..."* — kebawa ke PDF |
| III-E-1 | Judul *"LOSO & Feature Ablation:"* tapi **kosong** |
| Abstract, IV-D, Conclusion | `(unknown value)` masih ada di 3 tempat |
| IV-D | Satu paragraf masih ke-highlight kuning |

---

## 4. 🔴 Hapus semua referensi 5-fold cross-validation

5-fold nggak dijalanin lagi, tapi masih disebut di 3 tempat:

| Lokasi | Sekarang | Aksi |
|---|---|---|
| III-E-2 | Paragraf *"The fixed configuration was also retrained under stratified five-fold cross-validation..."* | **Hapus seluruh paragraf** |
| Akhir Intro | *"presents the classification, cross-validation, and computational-efficiency results"* | Ganti jadi *"signer-independent evaluation"* |
| Intro Section IV | *"reports model selection, held-out classification performance, cross-validation stability, and..."* | Ganti *"cross-validation stability"* jadi *"signer-independent evaluation"* |

Yang ketiga penting — sekarang pembaca dijanjiin sesuatu yang nggak ada di section itu.

---

## 6. 🟠 Signer D muncul tanpa perkenalan

Di IV-A-1 tiba-tiba muncul *"the sole native deaf signer (Signer D)"*. Tapi di Methodology:

- Nggak pernah disebut ada **berapa signer**
- Nggak pernah disebut mana yang **native Deaf** vs non-native
- III-A-2 cuma nyebut "two source subsets" (primary 60/class, secondary 50–59/class) tanpa ngaitin ke orang

Pembaca nggak punya cara tau dari mana "Signer D" datang, atau kenapa fold-nya 4.

**Tambahin di III-A-2:**

> Recordings were contributed by four signers. Three contributed approximately 20 sequences per class (together forming the primary subset), and one contributed approximately 50 per class (the secondary subset). One of the four(secondary) is a native Deaf BISINDO signer; the remaining three are hearing non-native signers. All 40 classes were recorded by all four signers, so no leave-one-signer-out fold contained unseen classes. Signer identity was confirmed via [filename grouping / visual inspection].

Kalimat "all 40 classes by all four signers" penting — pre-empt pertanyaan reviewer soal fold zero-shot.

Sekalian tambahin temuan protokol rekaman (ekspresi netral, mouthing) di sini, karena IV-A-1 udah ngerujuk ke situ (*"neutral facial expressions and incorrect or absent mouthing"*) tanpa dasar yang pernah ditulis.

---

## 7. 🟠 Selection-contingent estimate — harus diakui

Flow-nya bener: LOSO dipakai buat milih representasi, representasi terpilih dipakai buat training final. Itu lebih kuat dari praktik biasa — kalau arm dipilih berdasarkan stratified split, yang menang pasti 447 dim, dan balik ke masalah awal.

Tapi konsekuensinya: karena ABC dipilih berdasarkan performanya di 4 fold LOSO itu, angka LOSO ABC (71.67%) yang dilaporin sebagai estimasi generalisasi itu **bias optimis**. ABC menang di fold-fold itu justru karena dia dipilih dari situ.

Estimasi unbiased butuh nested LOSO (LOSO di dalam training signer buat seleksi), yang nggak feasible dengan 4 signer.

**Tambahin satu kalimat di IV-A atau IV-E:**

> Because the ABC configuration was selected on the basis of its LOSO performance, the reported LOSO accuracy is a selection-contingent estimate rather than an unbiased held-out measurement. A nested protocol would be required for an unbiased estimate, which is not feasible with four signers.

Ngakuin duluan itu murah. Ketauan reviewer itu mahal.

---

## 8. 🟡 Table I framing — 146 vs 142

III-B-5 bilang representasi direduksi jadi *"final compact 146 values 4-block modular design"*. Tapi model final pakai **142 (ABC)**, cuma 3 blok. Kontradiksi sama kesimpulan IV-A.

**Reword:** desain modular punya 4 blok dengan total 146 dim; ablation di Section IV milih 3 blok pertama (142 dim) sebagai konfigurasi final.

Frame Table I sebagai **ruang kandidat buat ablation**, bukan arsitektur final.

---

## 9. 🟡 IV-E gabung ke IV-A

Struktur Results sekarang bener secara kronologis — ablation duluan karena hasilnya yang nentuin input model final. Itu ngikutin urutan pipeline, jadi pertahankan.

Masalahnya cuma satu: **IV-E (Full LOSO Benchmark, Round 1 vs Round 2) kepisah jauh dari IV-A**, padahal dua-duanya LOSO. Pembaca ketemu LOSO di IV-A, dilempar ke classification/size/latency tiga subsection, terus balik ke LOSO lagi.

Fig. 9 (mean across folds) ada di IV-A, Fig. 12 (Round 1 vs Round 2) di IV-E — dua-duanya nunjukin hal yang sama.

**Pindahin isi IV-E jadi subsection ketiga di IV-A:**

| Sec | Isi |
|---|---|
| IV-A-1 | Kinematic Contributions and Native Signer Validation |
| IV-A-2 | Cross-Class Performance Balance and Feature Selection |
| **IV-A-3** | **Full LOSO Benchmark: 447 vs 142 Dimensions** ← eks IV-E |
| IV-B | Classification Performance |
| IV-C | Model Size |
| IV-D | Latency |

Permuted-group control juga masuk IV-A-3.

---

## 10. Yang perlu DITAMBAH

| Item | Kenapa | Taruh di |
|---|---|---|
| Paragraf LOSO di Related Work | Paper ini intinya LOSO, tapi literatur soal protokol signer-independent nggak disinggung. Sambungin ke Wiguna & Rojali [21] yang single-signer — mereka nyebut kualitatif, kita punya kuantitatifnya | II-D atau II baru |
| Signer demographics & coverage | Lihat Bagian 6 | III-A-2 |
| Recording protocol finding | Ekspresi netral + mouthing. IV-A-1 udah ngerujuk tanpa dasar | III-A-2 |
| Isi III-E-1 | Sekarang kosong. Deskripsiin protokol LOSO (4 fold, augmentasi train-only per fold, HP dibekukan), protokol ablation, dan permuted-group control | III-E-1 |
| Paragraf kontribusi evaluasi | 3 objectives di Intro nggak nyinggung signer generalization, padahal itu kontribusi utama sekarang. Tambahin **setelah** daftar objectives — objectives-nya jangan diubah | I |
| Tabel fold-by-fold | IV-E judulnya "Fold-by-Fold Analysis" tapi nggak ada tabel per fold, cuma bar chart | IV-A-3 |
| Tabel 20/20 | Sekarang cuma Fig. 10. Tabel lebih presisi buat 8 angka | IV-A-2 |
| Permuted-group control di teks | Cuma ada di legend Fig. 9, dan angkanya salah. Butuh 1–2 kalimat — ini yang mbuktiin drop-nya bukan bug harness | IV-A-3 |
| Pernyataan HP fixed a priori | Karena nggak ada hyperparameter search sama sekali, tulis eksplisit. Ini justru nutup celah — reviewer nggak bisa nuduh ada kebocoran lewat tuning. Contoh: *"Hyperparameters were fixed a priori rather than searched; no tuning was performed on any partition, and the same configuration was used for all ablation arms and for final training."* | III-C |
| Kalimat selection-contingent | Lihat Bagian 7 | IV-A |

---

## 11. Yang perlu DIHAPUS atau DIPOTONG

| Item | Alasan |
|---|---|
| Instruksi *"Add a new bullet point..."* di III-A-3 | Sisa editing |
| Kalimat Hyperband di III-C-2 | *"This narrow-to-wide configuration was selected by Hyperband rather than imposed manually"* — nggak ada hyperparameter search sama sekali. Hapus, ganti sama pernyataan HP fixed a priori (Bagian 10) |
| Paragraf 5-fold di III-E-2 | Lihat Bagian 4 |
| Fig. 5 (donut 80/10/10) | Informasi paling rendah di paper. Satu kalimat cukup |
| Fig. 6 (bar chart augmentasi) | Angkanya udah ada di teks |
| Fig. 3 (class distribution) | Bisa jadi satu kalimat: *"counts ranged 110–119 before balancing"* |

Tiga figure terakhir makan space besar dengan informasi minim. Tuker sama tabel fold-by-fold dan tabel 20/20.

---

## 12. Konsistensi istilah

Satu blok, tiga nama:

| Lokasi | Istilah |
|---|---|
| Table I, teks | "Articulator Pose" |
| Fig. 4 | "Articular Pose" |
| Fig. 8 | "Arm Directions" |

Pilih satu — saran: **Articulator Pose** di semua tempat.

Cek juga blok D: Table I dan Fig. 4 sama-sama bilang 4 dims (142→146). Pastiin cocok sama isi aktualnya — kalau eyebrow kiri dan kanan dihitung terpisah, harusnya 5.

---

## 13. Abstract

Sekarang mimpin dengan 98.64%. Padahal kontribusi paper ini ceritanya LOSO dan ablation.

Kalau 98.64% ditaruh paling depan, itu semacam ngulang masalah versi lama — mimpin dengan angka dari protokol yang bocor. **Sandingin dua-duanya sejak kalimat pertama hasil.**

Yang belum ada di abstract sama sekali:

- Temuan ablation (ABC dipilih, Block D nggak nyumbang)
- Mekanisme identity leak
- Permuted-group control

---

## 14. Checklist

**Wajib sebelum submit:**

- [ ] Benerin 23 angka di Bagian 1
- [ ] Regenerate Table II (144,296 params, 128/64, input 142)
- [ ] Regenerate Fig. 7 (input 142, 128/64, dropout 0.4)
- [ ] Hapus instruksi editing di III-A-3
- [ ] Isi III-E-1
- [ ] Hapus semua referensi 5-fold (3 tempat)
- [ ] Hapus kalimat Hyperband di III-C-2, ganti sama pernyataan HP fixed a priori
- [ ] Ganti semua `(unknown value)` — butuh rerun latency N=10
- [ ] Regenerate confusion matrix + pair dari run sekarang
- [ ] Recompute Wilson CI buat 434/440
- [ ] Benerin legend Fig. 9
- [ ] Tambahin signer demographics di III-A-2
- [ ] Tambahin kalimat selection-contingent
- [ ] Putusin opsi A/B/C buat confound mirror

**Sangat disarankan:**

- [ ] Gabung IV-E ke IV-A sebagai IV-A-3
- [ ] Tambahin paragraf LOSO di Related Work
- [ ] Tambahin tabel fold-by-fold dan tabel 20/20
- [ ] Update Abstract & Conclusion
- [ ] Samain istilah blok B
- [ ] Reword Table I framing (146 kandidat → 142 final)