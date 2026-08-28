# thesis_fix

# SignLingo — Paper Revision Map

**Skenario: Round 2 berhasil (compact NMS ngangkat LOSO di atas hand-only)**

Dokumen ini nggak buat dikerjain sekarang. Ini peta buat nanti, biar pas angkanya masuk kamu nggak bingung mulai dari mana.

---

## Cara Pakai

Edit-nya dibagi tiga kategori:

| Tag | Artinya |
|---|---|
| 🔵 **WAJIB** | Perlu diedit apapun hasil Round 2 — karena LOSO & ablation-nya udah dijalanin |
| 🟢 **KALAU BERHASIL** | Cuma berlaku kalau compact NMS beneran ngangkat LOSO |
| 🔴 **KALAU GAGAL** | Branch alternatif (ada di bagian akhir) |

---

## ⚠️ Keputusan Besar yang Harus Diambil Duluan

**Model mana yang jadi headline deployment artifact?**

Ini nentuin seberapa besar revisinya. Jangan mulai ngedit sebelum ini diputusin.

Kalau compact model jadi model utama, input dim berubah **447 → ~152**. Konsekuensinya berantai:

| Yang ikut berubah | Kenapa |
|---|---|
| **Table I** (layer config) | GRU-1 params: `3×(447×32 + 32×32 + 2×32) = 46,176` → `3×(152×32 + 1024 + 64) ≈ 17,856`. Total 279,176 → **~250,856** |
| **Table V** (quantization) | Model lebih kecil → 319.8 KB kemungkinan turun signifikan. Harus di-convert & diukur ulang |
| **Latency** | Input lebih kecil → 1.029 ms geser |
| **Fig 7** (arsitektur) | Input shape `(30, 447)` → `(30, 152)` |
| **Confusion matrix (Fig 8)** | Model beda = confusion beda |
| **Held-out test 99.32%** | Harus diukur ulang pakai model compact |

### Tiga opsi

| Opsi | Isi | Effort | Koherensi |
|---|---|---|---|
| **A** | Compact jadi model utama, semua angka deployment diukur ulang | Berat | Tinggi |
| **B** | 149-landmark tetap headline, compact cuma ablation finding | Ringan | **Rendah** — kenapa deploy model yang kamu buktiin lebih jelek? |
| **C** ⭐ | Report dua-duanya: 149-landmark buat sequence-level, compact sebagai konfigurasi signer-robust dengan angka deployment sendiri | Sedang | Tinggi |

**Rekomendasi: C.** Cukup satu konversi TFLite + satu pengukuran latency tambahan buat model compact. Table V nambah baris, nggak diganti total. Dan ceritanya jujur: satu model menang di sequence-level, satu menang di signer-level, dan kamu jelasin kenapa.

**Bonus:** compact model kemungkinan besar **jauh** di bawah 319.8 KB. Klaim deployment-mu jadi lebih kuat, bukan lebih lemah.

> ❓ Cek juga: Flex delegate-nya masih muncul nggak di model compact? Itu isu GRU op, bukan input dim — jadi kemungkinan tetap ada. Kalau ternyata **hilang**, itu temuan bagus dan langsung nutup Fix 3 sepenuhnya.

---

## Peta Edit per Section

| Section | Aksi | Tag |
|---|---|---|
| Title | **Jangan diubah** | — |
| Abstract | Tambah LOSO + ablation + mirror; update angka size/latency | 🔵 |
| Keywords | Tambah signer-independent evaluation, landmark ablation | 🔵 |
| I. Introduction | Tambah paragraf kontribusi evaluasi (jangan ubah 3 objectives) | 🔵 |
| II-D | Sambungin ke Wiguna & Rojali single-signer limitation | 🔵 |
| II-E Positioning | Update deskripsi feature set | 🟢 |
| II (baru) | Paragraf singkat: LOSO sebagai standar signer-independent | 🔵 |
| III-A1 Vocabulary | **JANGAN DIUBAH** | 🚫 |
| III-A2 Recording | Tambah signer identity + recording-protocol finding | 🔵 |
| III-A3 Augmentation | **Tambah mirror augmentation** (belum ada di paper!) | 🔵 |
| III-B1 Keypoint Selection | Tambah deskripsi compact feature | 🟢 |
| III-B2 Normalization | Tambah catatan invariance sudut/rasio | 🟢 |
| III-C1 Partitioning | Deskripsikan dua protokol (stratified + LOSO) | 🔵 |
| III-E Evaluation Plan | Tambah LOSO, ablation, permuted-group control | 🔵 |
| IV-A | Caveat HP di-tune pakai protokol non-signer-disjoint | 🔵 |
| IV-B2 | Reposisi 99.32% jadi sequence-level | 🔵 |
| IV-B3 | 5-fold jadi *partition stability*, bukan generalization | 🔵 |
| **IV-B4 (baru)** | Signer-Independent Evaluation (LOSO + permuted control) | 🔵 |
| **IV-B5 (baru)** | Mirror Augmentation | 🔵 |
| **IV-B6 (baru)** | Landmark Ablation (4 arm + split 20/20) | 🔵 |
| IV-C Artifact Size | Fix 3 rephrase (teks udah siap) | 🔵 |
| IV-C Latency | Fix 4 N=10 (teks udah siap) | 🔵 |
| V-A Conclusion | Ganti kalimat limitation dengan angka beneran | 🔵 |
| V-B Future Work | Pindahin ablation ke Results, tambah elicitation protocol | 🔵 |

---

## Detail per Section

### Abstract 🔵

Kalimat yang sekarang cuma nyebut 99.32%. Itu overclaim di bawah protokol yang bener.

**Yang harus masuk:**
1. 99.32% tetap, tapi dilabelin sequence-level
2. Angka LOSO
3. Temuan ablation (satu klausa)
4. Mirror augmentation (satu klausa)
5. Update angka size & latency kalau opsi A/C

**Kerangka:**
> ...achieved 99.32% accuracy under class-stratified sequence-level partitioning. Under leave-one-signer-out evaluation, accuracy fell to [X]%, indicating that sequence-level partitioning substantially overstates signer-independent performance. A landmark ablation showed that [temuan]. Mirror augmentation recovered [X]pp on average and [X]pp on the most affected signer...

⚠️ Jangan buang 99.32%. Itu angka valid buat pertanyaan yang dia jawab. Yang salah itu nyajiin dia sendirian.

### I. Introduction 🔵

**Tiga objectives jangan diubah.** Itu tujuan desain yang emang kamu tetapkan di awal.

Tambahin paragraf **setelah** daftar objectives:

> In addition to these design objectives, this paper reports a signer-independent evaluation and a landmark ablation that were not part of the original protocol. These analyses were motivated by the observation that the stratified partition does not enforce signer disjointness.

Ini jujur soal urutan kerjanya, dan nggak bikin objectives keliatan post-hoc.

### II. Related Work 🔵

**II-D** udah nyebut Wiguna & Rojali yang single-signer dan ngaku itu generalization limitation. Sambungin ke temuanmu — sekarang kamu punya bukti kuantitatif buat masalah yang mereka cuma sebut kualitatif. Itu positioning yang bagus.

**Tambahan paragraf singkat** (2–3 kalimat) soal protokol evaluasi signer-independent di literatur SLR. Sekarang nggak ada sama sekali, padahal Section IV baru butuh landasan itu.

**II-E Positioning** 🟢 — kalimat *"It uses 149 selected hand, face, and pose landmarks"* harus update kalau konfigurasi finalnya berubah.

### III-A1 Vocabulary Selection 🚫

**JANGAN DISENTUH.**

Klasifikasi kelas tetap *"facial expression, head posture, or shoulder movement"*. Kelasnya tetap *marah*, *sedih*, *bingung*.

Alasan praktis, bukan cuma etis: kalau rasionale-nya diganti jadi "postur & orientasi kepala", reviewer baca itu terus lihat daftar isinya kata emosi → nggak nyambung. Kamu malah ngundang pertanyaan yang lagi dihindarin.

### III-A2 Recording Environment 🔵

Dua tambahan:

**1. Signer identity & coverage:**
> Recordings were contributed by four signers. Three signers each contributed approximately 20 sequences per class, and one signer contributed approximately 50 per class. All 40 classes were recorded by all four signers, so no leave-one-signer-out fold contained unseen classes. Signer identity was confirmed via [filename grouping / visual inspection].

**2. Recording-protocol finding** (dari inspeksi Step 3):
> Post-hoc inspection of the source recordings indicated that emotion-referencing signs (marah, sedih, bingung) were produced in citation form with largely neutral facial expression. [N] recordings per class were reviewed. The retained face landmarks therefore encoded static facial geometry rather than expressive variation.

Paragraf kedua ini yang bikin hasil ablation-mu **kausal**, bukan sekadar "kami buang face dan angkanya naik".

### III-A3 Data Augmentation 🔵

**Mirror augmentation belum ada di paper sama sekali.** Yang tercantum cuma Gaussian noise, spatial scaling, rotation, temporal perturbation.

Harus ditambah, termasuk mekanisme swap-nya:
> Horizontal mirroring was applied by negating the x-coordinate and exchanging paired left-right landmark indices (hands, shoulders, elbows, wrists, eyes, ears). This augmentation addresses handedness variation across signers.

Sebutin swap index-nya eksplisit — itu detail yang bikin hasilnya reproducible, dan bukti kamu ngerjain bener.

### III-B1 Spatial Keypoint Selection 🟢

149 landmark tetap didokumentasikan sebagai representasi awal. Tambahin subsection compact feature: blok A/B/C/D, dimensi, dan formula tiap fitur.

### III-B2 Normalization 🟢

Shoulder normalization tetap. Tambahin satu kalimat:
> Because the compact features are expressed as angles and distance ratios, they are invariant to the translation and uniform scaling applied by shoulder normalization; facial distances are additionally normalized by inter-ocular distance.

### III-C1 Data Partitioning 🔵

Kalimat sekarang: *"The reported protocol uses class-stratified sequence-level partitioning and does not document a signer-disjoint grouping constraint."*

Itu jujur, tapi sekarang jadi ketinggalan. Update jadi deskripsi **dua protokol**: stratified (buat model selection & sequence-level performance) dan LOSO (buat signer-independent). Jelasin protokol mana dipakai buat klaim mana.

### III-E Evaluation Plan 🔵

Tambah tiga subsection:
1. **LOSO protocol** — 4 fold, augmentasi train-only per fold, HP dibekukan
2. **Landmark ablation protocol** — 4 arm, semua kondisi lain identik
3. **Permuted-group control** — random fold assignment ukuran sama, buat misahin efek signer dari efek harness

### IV-A Hyperparameter Search 🔵

Tambah caveat:
> Hyperparameters were selected under a non-signer-disjoint protocol and were not re-optimized for the signer-independent setting; the reported LOSO figures may therefore understate achievable signer-independent performance.

### IV-B2 Held-Out Test 🔵

Angka 99.32% tetap. Yang berubah cuma label — sequence-level, bukan generalization umum. Kalimat *"these values represent held-out-sequence performance and not confirmed unseen-signer generalization"* udah bener, tinggal kasih forward reference ke IV-B4.

Kalimat *"Establishing the value of NMS features would require an ablation comparing otherwise identical manual-only and manual-plus-NMS inputs"* → sekarang **udah dikerjain**. Ganti jadi pointer ke IV-B6.

### IV-B3 Cross-Validation Stability 🔵

5-fold tetap dipertahankan, tapi diposisikan eksplisit sebagai **partition stability**, bukan generalization. Kalimat *"These results do not address signer-independent generalization"* sekarang jadi jembatan ke section berikutnya.

### IV-B4 (BARU) — Signer-Independent Evaluation 🔵

Isi:
- Tabel LOSO per-signer (baseline)
- Permuted-group control result
- Catatan asimetri fold: signer besar leave-out = held-out gede tapi training kecil, jadi fold itu drop paling jauh dan **itu bukan noise**

### IV-B5 (BARU) — Mirror Augmentation 🔵

Tabel 4 signer, baseline vs +mirror. signer_B 19.33% → 54.90% itu headline. Kasih space yang layak — jangan sampai ketutup cerita ablation yang lebih rumit.

Ini temuan paling actionable di seluruh studi: satu augmentasi, satu baris kode, +35.6pp di fold terburuk.

### IV-B6 (BARU) — Landmark Ablation 🔵

- Tabel 4-arm (hand-only → +articulator pose → +postural NMS → +facial NMS)
- Breakdown split 20/20 (NMS-focused vs standard manual)
- Interpretasi

**Pelaporan statistik:** n=4 fold itu tipis. Report **konsistensi arah + magnitude per fold**, jangan lean ke p-value. Yang meyakinkan reviewer itu "4/4 fold, −10.4 s/d −17.4" plus variance terendah, bukan p = 0.0029 dari n=4.

**Presisi klaim** — jaga di garis ini:

| ❌ Nggak bisa diklaim | ✅ Yang beneran dibuktiin |
|---|---|
| "Facial NMS nggak membantu BISINDO recognition" | "Di dataset ini, face landmark nggak bawa sinyal ekspresif; masukin blok high-dimensional yang uninformative merusak signer generalization" |

Yang pertama klaim tentang bahasa isyarat — kamu nggak punya datanya.

**Kalau 🟢 berhasil**, tambahin:
> A compact postural NMS representation recovered [X]pp over the hand-only baseline, indicating that non-manual information contributes to recognition when represented as normalized head-orientation features rather than as raw facial coordinates.

Itu yang nyelametin klaim di judul — **dengan bukti**, bukan asumsi.

### IV-C Model Size & Latency 🔵

- **Fix 3:** rephrase Flex delegate — teks udah siap tempel di roadmap Round 1
- **Fix 4:** latency N=10, update mean±std, sesuaikan rasio "98.1x"
- **Kalau opsi A/C:** tambah baris/kolom buat model compact

### V-A Conclusion 🔵

Kalimat sekarang: *"They do not yet establish generalization to unseen signers, quantify the contribution of facial landmarks, demonstrate end-to-end camera latency, or validate deployment on a mobile or microcontroller platform."*

Dua dari empat sekarang **udah kejawab**. Ganti dengan angka beneran, sisain dua yang masih terbuka.

### V-B Future Work 🔵

- *"A controlled landmark ablation should then compare hand-only, hand-and-pose, and hand-pose-face inputs"* → udah dikerjain, **pindah ke Results**
- **Tambah rekomendasi elicitation protocol:**

> Selecting NMS-focused vocabulary is not sufficient on its own; an elicitation protocol that reliably produces natural non-manual marking is also required. Non-native signers producing citation forms may yield neutral facial expression, in which case retained facial landmarks encode signer-specific geometry rather than linguistic signal. Future collection should involve native signers or elicit signs in sentence context.

Ini kontribusi beneran — nggak ada di future work paper manapun yang kamu sitir.

---

## Tabel & Figure

### Baru
| Item | Isi |
|---|---|
| Table VI | LOSO per-signer + permuted-group control |
| Table VII | Mirror augmentation (4 signer × baseline/+mirror) |
| Table VIII | Landmark ablation 4-arm |
| Table IX *(atau kolom di VIII)* | Breakdown split 20/20 |
| Fig baru *(opsional)* | Diagram blok compact feature |

### Yang perlu update
| Item | Perubahan |
|---|---|
| Fig 1 | Tambah mirror ke pipeline augmentasi |
| Fig 4 | Update blok feature |
| Fig 7 🟢 | Input shape `(30, 447)` → `(30, 152)` |
| Table I 🟢 | Param count 279,176 → ~250,856 |
| Table V 🟢 | Baris model compact |
| Fig 8 🟢 | Confusion matrix model compact |

### Page budget — kandidat potong

4 tabel + 3 subsection itu nambah banyak. Kalau kena limit halaman:

| Kandidat | Kenapa aman dipotong |
|---|---|
| **Fig 5** (donut 80/10/10) | Informasi paling rendah di paper. Satu kalimat cukup |
| **Fig 6** (bar chart augmentasi) | Sama, angkanya udah ada di teks |
| **Fig 3** (class distribution) | Bisa jadi satu kalimat: "counts ranged 110–119 before balancing" |

Tiga figure itu total makan space besar dan hampir nggak bawa informasi yang nggak bisa ditulis satu kalimat. Tuker sama tabel LOSO/ablation — jauh lebih berharga.

---

## Aturan yang Nggak Boleh Dilanggar

1. **Ubah klaim ke depan = sah. Ubah catatan apa yang dilakukan = enggak.** Methodology tetap, temuan ditambahin.
2. **Vocabulary rationale jangan disentuh.**
3. **Jangan buang 99.32%.** Kasih label protokolnya.
4. **Jangan klaim melampaui data.** Temuan ini tentang dataset & representasi, bukan tentang BISINDO secara umum.
5. **Judul aman** — head orientation, postur kepala, gerakan bahu itu semua NMS. Cukup abstract-nya presisi bahwa NMS yang ketangkep itu **postural**, bukan facial.

---

## 🔴 Kalau Round 2 Gagal

Kalau compact NMS nggak ngelewatin hand-only, semua edit 🔵 tetap berlaku. Yang berubah cuma framing di IV-B6 dan V-B.

**Framing-nya jadi:**
> Non-manual information did not improve signer-independent recognition in this dataset, under either dense or compact representation. Inspection of the source recordings indicates that this reflects the recording protocol — emotion-referencing signs were produced with neutral facial expression — rather than a property of BISINDO.

Itu **tetap paper yang bagus.** Negative finding yang di-diagnose sampai akarnya, dengan bukti recording-protocol dan bukti representasi, itu kontribusi beneran. Yang lemah itu negative finding tanpa penjelasan.

Judul tetap aman selama abstract-nya jujur bahwa NMS dievaluasi tapi nggak terbukti berkontribusi **di dataset ini**.

**Yang berubah di keputusan besar:** kalau gagal, opsi B jadi masuk akal — hand-only jadi konfigurasi signer-robust, dan compact feature dilaporkan sebagai upaya yang nggak berhasil. Nggak ada re-measurement deployment yang perlu.

---

## Checklist Eksekusi

- ☐ Putusin opsi A / B / C (headline model)
- ☐ Kalau A/C: convert compact model ke TFLite, ukur size + latency
- ☐ Cek Flex delegate masih muncul nggak di model compact
- ☐ Recount param Table I
- ☐ Tulis 3 subsection baru (IV-B4, B5, B6)
- ☐ Bikin 3–4 tabel baru
- ☐ Update Fig 1, 4 (+ 7, 8 kalau A/C)
- ☐ Tempel Fix 3 & Fix 4 (teks udah siap)
- ☐ Rewrite abstract
- ☐ Rewrite conclusion + future work
- ☐ Cek page limit, potong Fig 3/5/6 kalau perlu
- ☐ Baca ulang: ada klaim yang melampaui data nggak?
