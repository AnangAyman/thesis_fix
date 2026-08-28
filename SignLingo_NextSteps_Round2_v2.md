# SignLingo Pre-Thesis — Next Steps (Round 2, v2)

**Compact Feature Redesign · Validation Hardening · Recording-Protocol Finding**

---

## Di Mana Kita Sekarang

### Hasil Round 1

**Mirror augmentation — berhasil**

| Held-out signer | Baseline | +mirror | Change |
|---|---|---|---|
| A_numeric | 46.67% | 51.04% | +4.36pp |
| B_dash | 19.33% | 54.90% | **+35.56pp** |
| C_underscore | 46.29% | 49.14% | +2.84pp |
| D_bisindo | 53.01% | 59.38% | +6.37pp |
| **Mean** | **41.33%** | **53.61%** | **+12.28pp** |

**Ablation — nambah landmark bikin signer generalization makin jelek, monoton**

| Arm | Mean LOSO | Std |
|---|---|---|
| **Hand-only** | **67.54%** | **±2.70%** |
| Hand+Pose | 58.35% | ±5.79% |
| Full (+face) | 53.61% | ±3.92% |
| Full + delta | 48.68% | ±10.51% |

Hand-only ngalahin Full 13.93pp, arah sama di 4/4 fold (−10.4, −12.5, −15.3, −17.4), p = 0.0029. Hand-only juga paling stabil (±2.70%).

### Yang udah CLOSED

**✅ Class coverage.** Scenario A: tiap signer primary ngerekam semua 40 kelas (~20/class), plus ~50/class dari secondary. Semua kelas punya keempat signer, nggak ada fold yang zero-shot. 67.54% itu angka generalization beneran.

**✅ Kenapa face block nggak nyumbang — akar masalahnya ketemu.** Inspeksi awal nunjukin dua hal barengan:

1. Vocabulary-nya **under-sample facial NMS** — mayoritas kelas "NMS-focused" itu sebenernya postur/orientasi kepala, bukan ekspresi wajah.
2. Kelas emosi (*marah*, *sedih*, *bingung*) direkam dalam **citation form dengan muka relatif netral** — signer non-native meragain handshape yang bener tapi ekspresi datar.

Konsekuensinya: **face landmark nggak punya sinyal ekspresif — by construction.** Yang tersisa di 222 dim itu murni geometri wajah = identity. Ini penjelasan primer buat hasil ablation, lebih kuat daripada sekadar "dense landmark bocor identity".

---

## Apa yang Berubah dari Draft Sebelumnya

| Item | Draft lama | Sekarang | Alasan |
|---|---|---|---|
| Face block | Compact 8-dim gabungan | Dipecah: postural (3) vs facial (5), jadi blok terpisah | Head orientation itu NMS juga — kalau digabung ke "pose", kontribusi NMS jadi invisible di tabel ablation |
| Subgroup tagging head-posture vs expression | Step wajib | **DIBUANG** | Split-nya bakal ~18/2, nggak ada power. Diganti split 20/20 yang udah ada di paper |
| Mekanisme "jenis NMS mana yang penting" | Dari class subgrouping | Dari **arm delta** | Lebih langsung, nol tagging, nol tuduhan post-hoc |
| Paper reframing | 3 poin umum | Aturan eksplisit: methodology **jangan** diubah, temuan **ditambahin** | Ngubah rasionale vocabulary = ngerapiin desain studi biar cocok hasil |

---

## Ringkasan Cepat

| # | Step | Kenapa Penting | Biaya |
|---|---|---|---|
| 1 | Permuted-group control | Misahin "efek signer" dari "bug harness". **GATE** | 1–3 run cepat |
| 2 | Mirror-symmetry audit | Kalau mirror face/pose cuma negate-x tanpa swap index, arm Full di-handicap artifisial | 5 menit |
| 3 | Inspeksi rekaman (formal) | Dasar buat kalimat limitation di paper + nentuin apakah mouth aperture punya sinyal | 30 menit |
| 4 | Extractor 4-blok | Inti Round 2 | Nulis script |
| 5 | Ablation Round 2 (4 arm) | Tiap delta jawab persis satu pertanyaan | 3 training run |
| 6 | Paper additions | Presisi klaim | Nulis |

---

## Step 1 — Permuted-Group Control (GATE)

Waktu pindah dari 5-fold ke LOSO, ada dua hal berubah barengan: fold-nya jadi signer-disjoint (yang mau diukur), **dan** harness-nya baru (variabel pengganggu). Dari hasil sekarang nggak bisa dibedain.

**Yang dilakukan:** jalanin harness LOSO yang sama persis, tapi assign sequence ke fold secara **acak**, bukan by signer. Identik semuanya — jumlah fold (4), ukuran fold, augmentation, epoch, HP. Yang berubah cuma grouping-nya nggak signer-aligned. Idealnya 3× seed beda.

| Hasil | Artinya | Aksi |
|---|---|---|
| ~99% | Harness bener, drop murni efek signer ✅ | Lanjut |
| ~67% | Ada yang rusak/bocor di harness | **Stop, debug** |
| ~80% | Campuran | Investigasi |

**Masuk paper juga:**
> *"To verify that the performance drop reflects signer identity rather than a change in the evaluation harness, we re-ran the identical pipeline with randomly permuted fold assignments of matched size, which recovered [X]% accuracy."*

---

## Step 2 — Mirror-Symmetry Audit

Mirror buat hand gampang (tuker blok L↔R, negate x) dan kemungkinan udah bener — buktinya signer_B naik 35.6pp.

Buat face & pose lebih halus: nggak cukup negate-x, index kiri-kanan harus **di-swap** juga. Alis kiri↔kanan, cheek kiri↔kanan, shoulder 11↔12, elbow 13↔14, wrist 15↔16, mata 2↔5, telinga 7↔8.

**Kalau kelewat:** arm Full & Hand+Pose dapat augmentasi rusak sementara Hand-only dapat yang bener → gap 13.93pp jadi sebagian artifak. Arahnya kemungkinan tetap sama, tapi magnitude-nya nggak bisa dipercaya.

**Cek:** mirror satu sequence, scatter-plot ulang landmark-nya. Masih keliatan wajah = aman.

**Bonus compact feature:** mirroring jadi trivial dan nggak bisa salah — roll ganti tanda, yaw ganti tanda, eyebrow L↔R, mouth aperture/width nggak berubah. Nggak ada index remapping yang bisa keliru.

---

## Step 3 — Inspeksi Rekaman (Formalkan)

Temuan neutral-face itu udah jadi bagian dari narasi paper, jadi basisnya harus didokumentasikan — sama kayak konfirmasi signer identity.

**Yang dilakukan:** buka 3–4 video per kelas untuk *marah*, *sedih*, *bingung*, plus beberapa kelas manual sebagai pembanding. Catat:

- Ada gerakan ekspresi wajah nggak? (alis, mulut, mata)
- **Ada mouthing nggak** — bibir bentuk kata pas signing?
- Konsisten antar signer atau beda-beda?

**Kenapa mouthing penting:** kalau ternyata ada mouthing konsisten, mouth aperture punya sinyal beneran, dan arm facial NMS berubah dari konfirmatori (mastiin nol) jadi eksploratori (mungkin ada gain). Itu ngubah ekspektasi, bukan ngubah desain.

**Output:** satu paragraf siap tempel:
> *Post-hoc inspection of the source recordings indicated that emotion-referencing signs (marah, sedih, bingung) were produced in citation form with largely neutral facial expression. [N] recordings per class were reviewed. The retained face landmarks therefore encoded static facial geometry rather than expressive variation.*

---

## Step 4 — Extractor Compact 4-Blok

### Kenapa dipecah 4 blok, bukan 2

Head orientation itu **non-manual marker**. Kalau dia dilebur ke "compact pose", kontribusi NMS nggak keliatan di tabel ablation — padahal itu klaim inti paper. Jadi dipisah:

| Blok | Isi | Dim | Normalisasi |
|---|---|---|---|
| **A. Hands** | 42 titik × 3, nggak diubah | 126 | Shoulder norm (sama kayak sekarang) |
| **B. Articulator pose** | shoulder (11,12), elbow (13,14), wrist (15,16) × 3 | 18 | Shoulder norm (sama kayak sekarang) |
| **C. Postural NMS** | head roll, yaw, pitch | 3 | Shoulder norm ke-cancel sendiri → inter-ocular |
| **D. Facial NMS** | mouth aperture, mouth width, eyebrow raise L/R, eye openness | 5 | Shoulder norm ke-cancel sendiri → inter-ocular |

Total kalau semua: **152 dim** (dari 447).

### ❗ Shoulder normalization TETAP DIPAKAI — nggak ada yang diganti

Ini poin yang paling gampang bikin bingung, jadi ditulis eksplisit:

**Pipeline extraction kamu nggak berubah sama sekali.** Array `(30, 447)` shoulder-normalized yang udah ada tetap dipakai apa adanya. Yang ditambahin cuma **satu langkah derivasi setelahnya** — milih blok A + B dari array itu, dan ngitung C + D dari titik-titik yang udah tersimpan di dalamnya.

Inter-ocular itu **lapisan kedua di atas** shoulder norm, bukan pengganti.

#### Kenapa blok C & D "ke-cancel sendiri"

Shoulder normalization itu `P̃ = (P − O)/D` — cuma **translasi + scaling seragam**. Konsekuensinya:

- **Sudut** (roll, yaw) itu *invariant* terhadap translasi dan scaling seragam. Dihitung dari koordinat mentah atau dari koordinat shoulder-normalized, **hasilnya angka yang sama persis**.
- **Rasio dua jarak** juga invariant. Mouth aperture dan inter-ocular dua-duanya ke-scale `1/D`, jadi waktu dibagi, `D`-nya coret.

Jadi bukan kamu ngeganti normalisasi buat blok C/D — blok C/D itu emang **kebal** sama shoulder norm. Makanya aman dipakai bareng blok A/B yang masih koordinat mentah shoulder-normalized.

#### Kenapa inter-ocular, bukan shoulder width?

Dua-duanya konstanta per-orang, jadi wajar kalau kebayang sama aja. Bedanya:

| Denominator | Yang diukur | Identity leak |
|---|---|---|
| `/ shoulder_width` | proporsi wajah-ke-badan orang itu | **kuat** |
| `/ inter_ocular` | proporsi internal wajah | jauh lebih lemah |

Tapi jujur: **pemilihan denominator itu efek kedua.** Yang bener-bener ngebunuh identity leak adalah **222 dim → 5 dim**. Koordinat mentah bawa seluruh geometri wajah; lima skalar cuma bawa besaran deformasinya.

#### Opsional (jangan dulu): per-sequence baseline subtraction

Kalau blok D ternyata masih bocor identity, bisa ditambah: kurangi tiap fitur blok D sama median-nya sendiri dalam sequence itu. Jadi yang diukur "seberapa jauh dari netral **orang ini**", bukan nilai absolut.

> ⚠️ **Tradeoff:** kalau ada sign yang mulutnya kebuka **konstan** sepanjang sequence, baseline-subtraction bakal ngehapus sinyalnya. Buat isolated word 30 frame, risikonya nyata. **Skip dulu** — coba versi biasa, baru pertimbangin kalau blok D masih bocor.

### Blok C — head orientation, semua dari pose

Pose landmark 0–10 udah cukup: nose (0), mata (1–6), telinga (7,8), sudut mulut (9,10).

| Fitur | Cara hitung |
|---|---|
| Roll | sudut garis mata kiri–kanan (pose 2↔5) |
| Yaw | offset horizontal nose (0) relatif midpoint telinga (7,8) |
| Pitch | jarak vertikal nose (0) ↔ midpoint mata (2,5) |

Roll & yaw itu sudut, jadi udah invariant — nggak butuh denominator sama sekali. Pitch berupa jarak, jadi **dibagi inter-ocular distance**.

Alternatif kalau inter-ocular noisy: ear-to-ear distance (7↔8).

> **Catatan pitch:** ini yang paling lemah dari tiga sudut, karena proyeksi 2D bikin "nunduk" susah dibedain dari "kepala turun". Formula jarak-vertikal-nya workable tapi noisy. Coba pakai koordinat **z** (MediaPipe relative depth) buat nolong. Kalau tetap nggak stabil, **buang pitch, sisain roll + yaw** — dua itu yang paling relevan buat BISINDO dan paling robust di 2D.

### Blok D — facial NMS, dari face block

| Fitur | Sumber |
|---|---|
| Mouth aperture | jarak vertikal lip atas ↔ lip bawah (dari 38 lip points) |
| Mouth width | jarak sudut mulut (bisa dari pose 9↔10 kalau lebih stabil) |
| Eyebrow raise L/R | jarak vertikal alis ↔ mata sisi yang sama (16 eyebrow points + pose eye) |
| Eye openness | rata-rata bukaan kelopak |

Semuanya jarak, jadi semuanya **dibagi inter-ocular** → jadi rasio tak berdimensi.

**Cheek landmarks (20 titik): buang.** Itu praktis lebar wajah = identity murni.

> **Prasyarat:** kamu butuh **index mapping** dari script preprocessing — 74 face point mana aja yang dipilih dari 468, dan urutannya di array flattened. Kalau list itu masih ada di kode → cukup derive dari array `(30, 447)` yang udah ada, **nggak perlu re-extract video**. Kalau ilang → fallback re-extract dari Drive.

### Yang dibuang dari pose

Hip (23,24), knee (25,26), ankle (27,28), foot. Di rekaman upper-body semuanya extrapolated dari shoulder — artinya **fungsi deterministik dari titik yang udah ada**: nol informasi baru, tapi bawa bias per-orang.

Ekstra: kalau framing videonya nggak konsisten (sebagian pinggang keliatan, sebagian enggak), hip itu real di sebagian video dan extrapolated di sebagian lain — dan perbedaan itu kemungkinan berkorelasi sama siapa yang ngerekam. Jadi hip bukan cuma noise, dia **literally signer ID channel**. Alasan tambahan yang enak ditulis di paper.

### ⚠️ Satu isu yang perlu dicek: shoulder movement vs shoulder normalization

Definisi NMS di paper kamu termasuk **shoulder movement**. Tapi normalisasi kamu pakai shoulder midpoint (centering) + shoulder distance (scaling).

- **Translation** shoulder line: dihapus normalisasi
- **Scale** (lean maju/mundur yang ngubah lebar apparent): dihapus normalisasi
- **Rotation** (bahu miring): **tetap ada** ✅

Jadi shoulder tilt selamat, tapi shrug/lean sebagian ke-cancel. Kalau ada kelas yang bergantung ke shoulder movement, cek dulu apakah sinyalnya masih ada setelah normalisasi. Kalau enggak, itu limitation yang harus ditulis — bukan diem-diem diklaim ketangkep.

---

## Step 5 — Ablation Round 2

### Empat arm, tiga run baru

| # | Arm | Blok | Dim | Status |
|---|---|---|---|---|
| 1 | Hand-only | A | 126 | ✅ udah ada (67.54%) |
| 2 | + articulator pose | A+B | 144 | run baru |
| 3 | + postural NMS | A+B+C | 147 | run baru |
| 4 | + facial NMS | A+B+C+D | 152 | run baru |

### Tiap delta jawab persis satu pertanyaan

| Delta | Pertanyaan |
|---|---|
| 1 → 2 | Arm/shoulder kinematics nyumbang di luar tangan? |
| 2 → 3 | **Head orientation nyumbang?** ← klaim NMS inti |
| 3 → 4 | Detail wajah nyumbang di atas head orientation? |

Ini yang gantiin class subgrouping. Pertanyaan mekanismenya dijawab di **level fitur**, langsung, tanpa tagging dan tanpa risiko post-hoc.

### Breakdown yang tetap dipakai: split 20/20 yang udah ada

NMS-focused vs standard manual. Gratis (tinggal slice prediksi), balanced, dan yang paling penting — **didefinisiin waktu desain vocabulary, jauh sebelum angka LOSO keluar.** Pre-registration-nya bersih.

Testnya: kalau blok C beneran nyumbang, gain-nya harus kekonsentrasi di 20 kelas NMS-focused, bukan nyebar rata ke 40.

### Kondisi yang dijaga konstan

- **Mirror augmentation ON di semua arm** (konfirmasi dulu — lihat checklist)
- HP dibekukan (32/256 GRU, dropout 0.3, L2 1e-4)
- Fold assignment identik
- Epoch budget & early stopping identik

### ⚠️ Caveat hyperparameter

178 Hyperband trial kamu di-tune pakai **sequence-level split** — jadi efektif dioptimasi buat setting yang membolehkan hafalan signer. 32/256 + dropout 0.3 belum tentu optimal buat signer-independent.

**Jangan retune di LOSO fold terus report yang terbaik** — itu selecting on test, persis error yang lagi diperbaiki. Opsi jujur & murah: bekukan HP, tulis di limitation:

> *"Hyperparameters were selected under a non-signer-disjoint protocol and were not re-optimized for the signer-independent setting; the reported LOSO figures may therefore understate achievable signer-independent performance."*

### Pelaporan statistik

n=4 fold itu tipis. **Report konsistensi arah + magnitude per fold**, jangan lean ke p-value. Yang meyakinkan reviewer itu "4/4 fold, −10.4 s/d −17.4" plus variance terendah (±2.70%), bukan p = 0.0029 dari n=4.

---

## Step 6 — Paper Additions

### 🚫 Aturan yang nggak boleh dilanggar

**Ubah klaim ke depan = sah. Ubah catatan apa yang dilakukan = enggak.**

Section III (vocabulary selection) **jangan diubah.** Klasifikasi kelas tetap *"facial expression, head posture, or shoulder movement"* — itu emang niat desainnya, dan itu fakta historis.

Kenapa ini penting secara praktis, bukan cuma etis: kelasnya tetap *marah*, *sedih*, *bingung*. Kalau rasionale-nya diganti jadi "dipilih karena postur dan orientasi kepala", reviewer baca itu terus lihat daftar isinya kata emosi → langsung nggak nyambung. Kamu malah ngundang pertanyaan yang lagi dihindarin, tapi tanpa jawaban siap.

Yang bener: biarin methodology apa adanya, **tambahin apa yang ditemuin.**

### Presisi klaim — jaga di garis ini

| ❌ Nggak bisa diklaim | ✅ Yang beneran dibuktiin |
|---|---|
| "Facial NMS nggak membantu BISINDO recognition" | "Di dataset ini, face landmark nggak bawa sinyal ekspresif; masukin blok high-dimensional yang uninformative merusak signer generalization" |

Yang pertama klaim tentang bahasa isyarat — kamu nggak punya datanya. Yang kedua klaim tentang representasi dan data — itu yang kamu punya.

### Judul selamat

Head orientation, postur kepala, gerakan bahu — **itu semua non-manual markers.** *"Integrating Non-Manual Markers"* tetap jujur, asal papernya presisi bahwa NMS yang ketangkep itu **postural**, bukan facial. Nggak perlu ganti judul, cukup abstract-nya spesifik.

### Daftar edit

| Bagian | Aksi |
|---|---|
| Abstract | Sandingin LOSO sama 99.32%. Sekarang cuma nyebut 99.32% — overclaim di bawah protokol yang bener |
| III-A1 Vocabulary | **Jangan diubah** |
| III-A2 Recording | Tambah paragraf recording-protocol finding (Step 3) |
| IV-B3 Cross-Validation | Tambah subsection LOSO. 5-fold tetap, tapi diposisikan sebagai *partition stability*, bukan generalization |
| IV baru: Ablation | Tabel 4-arm + split 20/20 + permuted-group control |
| IV-C Mirror | signer_B +35.56pp — kasih space yang layak |
| Conclusion | *"do not yet establish generalization to unseen signers"* → ganti angka beneran |
| Future Work | Ablation udah dikerjain → pindah ke Results. Tambah rekomendasi elicitation protocol |

### Kontribusi baru buat Future Work

Temuannya: **milih vocabulary NMS-focused itu nggak cukup.** Kamu juga butuh protokol elicitation yang beneran ngasilin NMS natural. Signer non-native yang meragain citation form bakal ngasih muka netral, dan face landmark-nya jadi identity channel murni.

Rekomendasi konkret: rekam pakai native signer, atau elicit lewat konteks kalimat, bukan citation form. Ini nggak ada di future work paper manapun yang kamu sitir.

### Jangan sampai ketutup

Mirror result itu temuan paling bersih dan paling actionable di seluruh Round 1. 19.33% → 54.90% dari satu augmentasi. Bukti kuat bahwa handedness itu dominant failure mode dan fix-nya satu baris kode.

---

## Carry-Over dari Roadmap Round 1

- ☐ **Fix 3 — Model size / Flex delegate wording.** Rephrase-only, teks revisi udah siap tempel di roadmap Round 1.
- ☐ **Fix 4 — Latency N=1 → N=10.** Rerun `--n_sequences 10`, update mean±std, sesuaikan rasio "98.1x".

Kerjain sambil nunggu training Step 5.

---

## Urutan Kerja

1. **Step 1 (permuted control)** — GATE, jangan lanjut sebelum lolos
2. **Step 2 (mirror audit)** — 5 menit, nentuin apakah angka Round 1 bisa dipercaya apa adanya
3. **Step 3 (inspeksi rekaman)** — sekalian cek mouthing sebelum bikin blok D
4. **Cek index mapping face** — prasyarat Step 4
5. **Step 4 (extractor)** — nulis + verifikasi visual
6. **Step 5 (3 training run)** — overnight
7. **Carry-over Fix 3 & 4** — sambil nunggu
8. **Step 6 (paper)** — terakhir

---

## Yang Perlu Dikonfirmasi

- ☒ **Class coverage** — CLOSED. Scenario A, semua 40 kelas × 4 signer
- ☒ **Face block: hapus atau arm** — CLOSED. Jadi arm (blok D, 5 dim)
- ☒ **Subgroup tagging** — CLOSED. Dibuang, pakai split 20/20 yang udah ada
- ☒ **Shoulder normalization** — CLOSED. **Tetap dipakai, extraction script nggak diubah.** Blok C/D kebal sama shoulder norm (sudut & rasio itu invariant), inter-ocular cuma lapisan kedua di atasnya
- ☐ **Index mapping 74 face landmark** — masih ada di script preprocessing? Prasyarat blok D
- ☐ **Mirror ON di semua 4 arm?** Dan hand-only 67.54% itu with-mirror atau without? Kalau without, gap 13.93pp bukan apple-to-apple
- ☐ **Mouthing ada nggak** di rekaman? Nentuin ekspektasi blok D
- ☐ **z-coordinate usable** buat pitch? Kalau noisy, drop pitch, sisain roll+yaw
- ☐ **Framing video konsisten?** Kalau hip keliatan di sebagian video doang, itu amunisi tambahan buat argumen identity channel
- ☐ **Shoulder movement** masih ada sinyalnya setelah shoulder normalization?
