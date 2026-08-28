# SignLingo Pre-Thesis — Next Steps (Round 2, v3 — Annotated)

**Compact Feature Redesign · Validation Hardening · Recording-Protocol Finding**

> Versi ini nggantiin v2. Isinya sama, tapi tiap keputusan dikasih alasannya — biar orang yang baca (dosen pembimbing, anggota tim, atau kamu sendiri tiga bulan lagi) ngerti **kenapa**, bukan cuma **apa**.

---

## Bagian 0 — Konteks: Kenapa Round 2 Ini Ada

### Masalah asalnya

Paper melaporkan akurasi 99.32%. Angka itu didapat dari **class-stratified sequence-level split** — 4.400 sequence diacak, 80% train / 10% val / 10% test, dijaga tiap kelas proporsinya seimbang.

Yang **nggak** dijaga: siapa yang ngerekam sequence-nya.

Akibatnya, satu orang yang sama muncul di training **dan** di testing. Model bisa dapet nilai tinggi cuma dengan hafal kebiasaan 4 orang spesifik ini — cara mereka bikin handshape, postur badan mereka, bentuk wajah mereka — tanpa beneran belajar sign-nya.

### Kenapa itu masalah beneran, bukan cuma teoretis

Sistem ini nantinya dipakai orang yang **nggak pernah** direkam. Kalau model cuma hafal 4 orang, akurasi 99.32% itu nggak nyeritain apapun soal performa di dunia nyata.

Standar yang bener di literatur SLR itu **LOSO (Leave-One-Signer-Out)**: tiap fold nyisain satu orang **utuh** buat testing. Model dipaksa ngenalin orang yang belum pernah dia lihat.

### Hasilnya

| Protokol | Akurasi | Pertanyaan yang dijawab |
|---|---|---|
| Stratified 5-fold | 99.16% ± 0.26 | "Bisa ngenalin rekaman baru dari orang yang udah dikenal?" |
| **LOSO** | **53.61%** (Full) | **"Bisa ngenalin orang baru?"** |

**Gap-nya ~45 poin.** Itu bukan bug — itu jawaban dari dua pertanyaan yang beda. Dan pertanyaan kedua yang penting buat deployment.

---

## Bagian 1 — Hasil Round 1 & Interpretasinya

### 1.1 Mirror augmentation

| Held-out signer | Baseline | +mirror | Change |
|---|---|---|---|
| A_numeric | 46.67% | 51.04% | +4.36pp |
| B_dash | 19.33% | 54.90% | **+35.56pp** |
| C_underscore | 46.29% | 49.14% | +2.84pp |
| D_bisindo | 53.01% | 59.38% | +6.37pp |
| **Mean** | **41.33%** | **53.61%** | **+12.28pp** |

**Kenapa signer_B cuma 19.33% di baseline?**

Kemungkinan besar tangan dominannya beda dari tiga signer lainnya. Waktu signer_B jadi held-out, model dilatih cuma dari tiga orang dengan handedness yang sama. Model belajar "sign X = tangan kanan begini, tangan kiri begitu". Signer_B ngelakuin versi cerminnya — dan buat model itu keliatan kayak gerakan yang sama sekali beda.

**Kenapa mirror augmentation ngangkat 35.6pp?**

Mirroring bikin tiap training sequence punya pasangan cerminnya. Model jadi lihat kedua konfigurasi handedness buat tiap kelas, jadi dia nggak bisa lagi ngandelin "tangan mana yang aktif" sebagai pembeda. Handedness jadi irrelevant.

Naik dari 19% ke 55% artinya **handedness adalah penyebab kegagalan terbesar** buat signer itu — bukan sekadar salah satu faktor.

**Kenapa ini temuan yang berharga:** fix-nya satu baris kode, biayanya nol, dan efeknya paling besar dari semua intervensi. Di paper ini harus dapet space yang layak.

### 1.2 Ablation

| Arm | Mean LOSO | Std |
|---|---|---|
| **Hand-only** | **67.54%** | **±2.70%** |
| Hand+Pose | 58.35% | ±5.79% |
| Full (+face) | 53.61% | ±3.92% |
| Full + delta | 48.68% | ±10.51% |

**Bacanya:** makin banyak landmark, makin **jelek** generalization ke orang baru. Monoton, nggak ada pengecualian.

Hand-only ngalahin Full 13.93pp, arah sama di 4/4 fold (−10.4, −12.5, −15.3, −17.4).

**Detail yang sering kelewat: hand-only juga paling stabil** (±2.70%, variance terendah dari semua arm). Artinya bukan cuma rata-ratanya lebih tinggi — performanya juga paling konsisten antar orang. Buat sistem yang mau dipakai publik, konsistensi itu sama pentingnya sama rata-rata.

**Kenapa ini kontra-intuitif tapi masuk akal:** biasanya nambah informasi itu bantu. Tapi di sini yang ditambahin bukan informasi — yang ditambahin **identitas orang**. Penjelasan lengkapnya di Bagian 4.

### 1.3 Yang udah tertutup

**✅ Class coverage.** Tiap signer primary ngerekam semua 40 kelas (~20/class), plus ~50/class dari secondary. Semua kelas punya keempat signer.

> **Kenapa ini perlu dicek:** kalau ada kelas yang cuma direkam satu orang, fold yang nyisain orang itu bikin model nggak pernah lihat kelas tersebut sama sekali — namanya zero-shot. Akurasi jeblok bukan karena gagal generalisasi, tapi karena secara struktural mustahil. Karena coverage-nya penuh, **67.54% itu angka generalization beneran.**

**✅ Kenapa face block nggak nyumbang.** Inspeksi rekaman nunjukin dua hal:

1. Vocabulary **under-sample facial NMS** — mayoritas kelas "NMS-focused" sebenernya postur/orientasi kepala, bukan ekspresi wajah.
2. Kelas emosi (*marah*, *sedih*, *bingung*) direkam dalam **citation form dengan muka relatif netral** — signer non-native meragain handshape yang bener tapi ekspresi datar.

> **Kenapa ini penting banget:** artinya face landmark **nggak punya sinyal ekspresif sejak awal**. Bukan "sinyalnya ketutup identity leak" — emang nggak ada yang direkam. Yang tersisa di 222 dimensi itu murni geometri wajah = identitas.
>
> Ini ngubah temuan dari *"kami buang face dan angkanya naik"* (kedengeran arbitrer) jadi *"face nggak bawa sinyal karena protokol rekamannya, jadi cuma nyisain identitas, dan itu yang ngerusak"* (kausal, bisa dipertanggungjawabkan).

---

## Bagian 2 — Ringkasan Langkah

| # | Step | Pertanyaan yang dijawab | Biaya |
|---|---|---|---|
| 1 | Permuted-group control | "Drop-nya karena signer atau karena kode baru?" | 1–3 run cepat |
| 2 | Mirror-symmetry audit | "Augmentasi mirror-nya bener nggak buat face/pose?" | 5 menit |
| 3 | Inspeksi rekaman (formal) | "Ada gerakan wajah nggak sih di rekaman?" | 30 menit |
| 4 | Extractor 4-blok | — (implementasi) | Nulis script |
| 5 | Ablation Round 2 | "NMS nyumbang nggak kalau di-representasi bener?" | 3 training run |
| 6 | Paper additions | — (penulisan) | Nulis |

---

## Bagian 3 — Validation Hardening

### Step 1 — Permuted-Group Control 🚦 GATE

#### Masalahnya

Waktu pindah dari 5-fold ke LOSO, ada **dua** hal yang berubah barengan:

1. Struktur fold-nya jadi signer-disjoint ← **ini yang mau diukur**
2. Kode harness-nya baru — split logic, augmentation re-partition, mungkin panjang training beda ← **variabel pengganggu**

#### Kenapa ini bahaya

Ini confound klasik: dua perubahan, satu hasil. Drop 99% → 67% bisa dari (1), bisa dari (2), bisa campuran. **Dari data yang ada sekarang, nggak ada cara bedain.**

Kalau ternyata sebagian besar dari (2), seluruh interpretasi Round 1 salah — dan kamu bakal nulis paper soal signer generalization padahal yang kamu ukur itu bug.

#### Yang dilakukan

Jalanin harness LOSO yang **sama persis**, tapi assign sequence ke fold secara **acak**, bukan by signer.

**Yang harus identik:** jumlah fold (4), ukuran tiap fold, augmentation pipeline, jumlah epoch, hyperparameter, seed handling.
**Yang berubah:** cuma satu — grouping-nya nggak lagi signer-aligned.

> **Kenapa harus identik semuanya:** biar cuma ada satu variabel yang beda. Kalau ukuran fold-nya beda dikit aja, hasilnya nggak bisa diinterpretasi lagi.

Idealnya diulang 3× dengan seed beda, biar nggak ketipu satu draw yang kebetulan bagus/jelek.

#### Cara baca hasilnya

| Hasil | Artinya | Aksi |
|---|---|---|
| ~99% | Harness bener, drop murni efek signer ✅ | Lanjut |
| ~67% | Ada yang rusak/bocor di harness | **Stop, debug** |
| ~80% | Campuran keduanya | Investigasi dulu |

#### Kenapa ini juga masuk paper

Reviewer pasti mikir hal yang sama: "gimana kamu tau ini bukan bug di kode barunya?" Punya control-nya duluan itu ngejawab sebelum ditanya.

> *"To verify that the performance drop reflects signer identity rather than a change in the evaluation harness, we re-ran the identical pipeline with randomly permuted fold assignments of matched size, which recovered [X]% accuracy."*

---

### Step 2 — Mirror-Symmetry Audit

#### Kenapa perlu dicek

Mirroring buat **tangan** itu gampang: tuker blok tangan kiri ↔ kanan, negate x. Kemungkinan udah bener — buktinya signer_B naik 35.6pp.

Mirroring buat **face dan pose** lebih halus. Nggak cukup cuma negate x — **index kiri-kanan juga harus di-swap**:

| Pasangan | Index |
|---|---|
| Alis kiri ↔ kanan | dari 16 eyebrow points |
| Cheek kiri ↔ kanan | dari 20 cheek points |
| Shoulder | 11 ↔ 12 |
| Elbow | 13 ↔ 14 |
| Wrist | 15 ↔ 16 |
| Mata | 2 ↔ 5 |
| Telinga | 7 ↔ 8 |

#### Kenapa swap-nya wajib

Kalau x di-negate tapi index-nya nggak di-swap, "alis kiri" sekarang berada di posisi sebelah kanan tapi masih dilabeli kiri. Hasilnya **struktur yang secara anatomis mustahil** — model dikasih training data yang bukan cuma salah, tapi nggak mungkin ada di dunia nyata.

#### Konsekuensinya kalau bug

Arm **Full** dan **Hand+Pose** dapat augmentasi rusak, sementara **Hand-only** dapat yang bener. Berarti gap 13.93pp itu sebagian artifak augmentasi, bukan murni identity leakage.

Arahnya kemungkinan tetap sama, tapi **magnitude-nya nggak bisa dipercaya** — dan itu hal yang gampang banget di-poke reviewer.

#### Cara cek — 5 menit

Ambil satu sequence, mirror-kan, terus scatter-plot ulang landmark-nya. Masih keliatan wajah (cuma nyerong ke arah sebaliknya) = aman. Berantakan = ketauan langsung.

#### Efek samping bagus dari compact feature

Salah satu untungnya pindah ke compact feature: **mirroring jadi trivial dan nggak bisa salah.**

| Fitur | Transformasi mirror |
|---|---|
| Head roll | ganti tanda |
| Head yaw | ganti tanda |
| Eyebrow raise | tuker L ↔ R |
| Mouth aperture / width | nggak berubah |

Delapan angka, semua transformasinya obvious. Nggak ada index remapping yang bisa keliru.

---

### Step 3 — Inspeksi Rekaman (Formalkan)

#### Kenapa ini perlu didokumentasikan

Temuan neutral-face udah jadi bagian dari narasi paper — dia yang ngejelasin **kenapa** face landmark gagal. Kalau nggak didokumentasikan basisnya, itu cuma klaim tanpa bukti.

Sama kayak konfirmasi signer identity: kamu harus bisa jawab "dari mana kamu tau?"

#### Yang dilakukan

Buka 3–4 video per kelas untuk *marah*, *sedih*, *bingung*, plus beberapa kelas manual sebagai pembanding. Catat:

- Ada gerakan ekspresi wajah nggak? (alis, mulut, mata)
- **Ada mouthing nggak** — bibir bentuk kata pas signing?
- Konsisten antar signer atau beda-beda?

#### Kenapa mouthing dicek terpisah

Mouthing (bibir membentuk kata Indonesia sambil signing) itu fenomena yang beda dari ekspresi emosi. Bisa aja mukanya datar tapi mouthing-nya aktif.

Kalau **ada mouthing konsisten**, mouth aperture punya sinyal beneran, dan blok D berubah dari **konfirmatori** (mastiin hasilnya nol) jadi **eksploratori** (mungkin ada gain). Itu ngubah ekspektasi, bukan ngubah desain.

Referensi [7] di paper kamu itu Koller et al., *"Deep learning of mouth shapes for sign language."* Kalau ternyata ada mouthing, referensi itu jadi relevan langsung ke hasilmu.

#### Output

Satu paragraf siap tempel:
> *Post-hoc inspection of the source recordings indicated that emotion-referencing signs (marah, sedih, bingung) were produced in citation form with largely neutral facial expression. [N] recordings per class were reviewed. The retained face landmarks therefore encoded static facial geometry rather than expressive variation.*

---

## Bagian 4 — Redesign Feature

### 4.1 Diagnosis: kenapa landmark tambahan bikin jelek

Ada empat sebab yang saling nguatin. Penting buat ngerti semuanya, karena solusinya beda-beda.

#### Sebab 1 — Face block mendominasi input

| Blok | Dim | % input |
|---|---|---|
| Face | 74 × 3 = 222 | **~50%** |
| Pose | 33 × 3 = 99 | 22% |
| Hands | 42 × 3 = 126 | 28% |

Separuh dari yang dilihat model itu wajah, padahal tangan yang bawa makna utama sign.

#### Sebab 2 — Koordinat wajah hampir konstan sepanjang sequence

Dalam satu sequence 30 frame, posisi relatif titik-titik wajah nyaris nggak berubah — kecuali kalau orangnya ngomong atau ekspresif. Yang mostly diukur itu **bentuk wajah**, bukan **gerakan wajah**.

Bentuk wajah itu konstanta per orang. Dari sudut pandang model, kamu ngasih dia 222 angka yang praktis berfungsi sebagai **nomor identitas**.

#### Sebab 3 — Shoulder normalization nggak nge-cancel bentuk wajah

Normalisasi kamu `P̃ = (P − O)/D` ngilangin **posisi** dan **skala** relatif bahu. Yang **nggak** dia hilangin: bentuk wajah, panjang leher, proporsi wajah-ke-badan, sudut kamera.

Jadi identity channel-nya lolos utuh lewat normalisasi.

#### Sebab 4 — Nggak ada sinyal ekspresif yang direkam

Dari Step 3: mukanya netral. Jadi bahkan kalau ketiga sebab di atas nggak ada, **face block emang nggak punya apa-apa buat dikasih.**

#### Gimana model memanfaatkannya

Di stratified split, tiap kelas punya keempat signer — jadi identitas nggak langsung nunjuk ke label. Tapi model bisa **hafal pasangan (siapa, kelas apa)**: "ini signer C lagi bikin kelas 17" — bukan "ini kelas 17".

Cara itu jalan sempurna waktu test set isinya orang yang sama. **Gagal total waktu ketemu orang baru.**

Itu persis yang keliatan di angka: 99% di stratified, 53% di LOSO.

#### Kenapa pose juga kena

Perhatiin: Hand+Pose udah turun (58.35% vs 67.54%) **padahal nggak ada wajahnya sama sekali.** Artinya pose block juga bocor identitas.

MediaPipe selalu ngeluarin 33 titik pose, termasuk pinggul (23,24), lutut (25,26), pergelangan kaki (27,28) — **terlepas dari apakah bagian itu keliatan di frame atau enggak.** Di rekaman upper-body, titik-titik itu di-extrapolate dari badan yang keliatan.

> **Kenapa extrapolated point itu nol informasi:** kalau posisi pinggul dihitung dari posisi bahu, maka dia **fungsi deterministik** dari sesuatu yang model udah punya. Nggak nambah informasi baru sama sekali. Tapi dia **nambah dimensi** dan **bawa proporsi tubuh per orang** — jadi murni identity + noise.

**Bonus yang perlu dicek:** kalau framing videonya nggak konsisten (sebagian pinggang keliatan, sebagian enggak), pinggul itu *real* di sebagian video dan *extrapolated* di sebagian lain. Perbedaan itu kemungkinan berkorelasi sama siapa yang ngerekam — jadi pinggul bukan cuma noise, dia **literally signer ID channel**.

### 4.2 Prinsip solusinya

**Ganti koordinat absolut jadi besaran relatif.**

| Representasi | Yang diukur | Bisa dibandingin antar orang? |
|---|---|---|
| Koordinat alis | *"alis ada di posisi (x,y)"* | ❌ tergantung wajah siapa |
| Rasio eyebrow raise | *"alis naik sekian % dari netralnya wajah ini"* | ✅ |

Itu inti idenya: **geometri absolut → deformasi relatif.** Yang pertama bawa identitas, yang kedua enggak.

Efek keduanya: 222 dimensi jadi 5. Koordinat mentah bawa seluruh geometri wajah; lima skalar cuma bawa besaran deformasinya.

> **Mana yang lebih ngaruh?** Pengurangan dimensi. Pemilihan denominator itu efek kedua. Tapi dua-duanya searah, jadi dipakai dua-duanya.

### 4.3 Kenapa dipecah 4 blok, bukan 2

Alasannya bukan teknis — **struktural buat ablation.**

Head orientation itu **non-manual marker**. Kalau dia dilebur ke "compact pose", kontribusinya nggak keliatan terpisah di tabel ablation — padahal itu **klaim inti judul paper**. Kamu bakal punya angka bagus tanpa bisa nunjukin dari mana asalnya.

| Blok | Isi | Dim | Normalisasi |
|---|---|---|---|
| **A. Hands** | 42 titik × 3, nggak diubah | 126 | Shoulder norm (sama kayak sekarang) |
| **B. Articulator pose** | shoulder (11,12), elbow (13,14), wrist (15,16) × 3 | 18 | Shoulder norm (sama kayak sekarang) |
| **C. Postural NMS** | head roll, yaw, pitch | 3 | Ke-cancel sendiri → inter-ocular |
| **D. Facial NMS** | mouth aperture, mouth width, eyebrow raise L/R, eye openness | 5 | Ke-cancel sendiri → inter-ocular |

Total: **152 dim** (dari 447).

### 4.4 ❗ Shoulder normalization TETAP DIPAKAI

Ini yang paling gampang bikin bingung, jadi ditulis eksplisit.

**Pipeline extraction nggak berubah sama sekali.** Array `(30, 447)` shoulder-normalized yang udah ada tetap dipakai apa adanya. Yang ditambahin cuma **satu langkah derivasi setelahnya** — milih blok A + B dari array itu, dan ngitung C + D dari titik-titik yang udah tersimpan di dalamnya.

Inter-ocular itu **lapisan kedua di atas** shoulder norm, bukan pengganti.

#### Kenapa blok C & D "ke-cancel sendiri"

Shoulder normalization itu cuma **translasi + scaling seragam**. Konsekuensi matematisnya:

- **Sudut** (roll, yaw) *invariant* terhadap translasi dan scaling seragam. Dihitung dari koordinat mentah atau dari koordinat shoulder-normalized, **hasilnya angka yang sama persis**.
- **Rasio dua jarak** juga invariant. Mouth aperture dan inter-ocular dua-duanya ke-scale `1/D`, jadi waktu dibagi, `D`-nya coret.

Jadi bukan kamu ngeganti normalisasi buat blok C/D — blok C/D itu **kebal** sama shoulder norm. Makanya aman dipakai bareng blok A/B yang masih koordinat mentah shoulder-normalized.

#### Kenapa inter-ocular, bukan shoulder width?

Dua-duanya konstanta per-orang, jadi wajar kalau kebayang sama aja. Bedanya:

| Denominator | Yang diukur | Identity leak |
|---|---|---|
| `/ shoulder_width` | proporsi wajah-ke-badan orang itu | **kuat** |
| `/ inter_ocular` | proporsi internal wajah | jauh lebih lemah |

Bayangin: orang berbadan kecil dengan wajah normal punya rasio wajah-ke-bahu yang khas. Itu identitas. Tapi rasio bukaan-mulut-ke-jarak-mata itu jauh lebih seragam antar orang.

### 4.5 Blok C — head orientation

Semua diambil dari **pose block**, bukan face block. Pose landmark 0–10 udah cukup: nose (0), mata (1–6), telinga (7,8), sudut mulut (9,10).

| Fitur | Cara hitung | Kenapa fitur ini |
|---|---|---|
| **Roll** | sudut garis mata kiri–kanan (pose 2↔5) | Kepala miring — grammatical marker di BISINDO |
| **Yaw** | offset horizontal nose (0) vs midpoint telinga (7,8) | Kepala noleh — arah pandang/referensi |
| **Pitch** | jarak vertikal nose (0) ↔ midpoint mata (2,5) | Kepala angguk/nunduk |

Roll & yaw itu **sudut**, jadi udah invariant — nggak butuh denominator sama sekali. Pitch berupa **jarak**, jadi dibagi inter-ocular.

Alternatif kalau inter-ocular noisy: ear-to-ear distance (7↔8).

> **Kenapa pitch paling lemah:** proyeksi 2D bikin "nunduk" susah dibedain dari "kepala turun" — dua-duanya keliatan sebagai nose bergerak ke bawah. Formula jarak-vertikal-nya workable tapi noisy.
>
> Coba pakai koordinat **z** (MediaPipe relative depth) buat nolong. Kalau tetap nggak stabil, **buang pitch, sisain roll + yaw** — dua itu paling relevan buat BISINDO dan paling robust di 2D. Lebih baik dua fitur yang bersih daripada tiga dengan satu yang berisik.

### 4.6 Blok D — facial NMS

| Fitur | Sumber | Kenapa dipertahankan |
|---|---|---|
| Mouth aperture | jarak vertikal lip atas ↔ bawah (38 lip points) | Mouthing — referensi [7] paper kamu sendiri |
| Mouth width | jarak sudut mulut (bisa dari pose 9↔10 kalau lebih stabil) | Bentuk mulut waktu mouthing |
| Eyebrow raise L/R | jarak vertikal alis ↔ mata sisi sama | Question marking |
| Eye openness | rata-rata bukaan kelopak | Ekspresi kaget/fokus |

Semuanya jarak → semuanya **dibagi inter-ocular** → jadi rasio tak berdimensi.

#### Kenapa nggak dihapus aja, padahal mukanya netral?

Pertanyaan wajar, dan sempat jadi opsi. Tiga alasan kenapa tetap diukur:

1. **Confusion pair-nya bilang lain.** Error di paper: *baik–apa*, *berapa–dia*, *mereka–tidur*. Dua dari tiga melibatkan kata tanya (*apa*, *berapa*). Eyebrow raise itu penanda kanonik buat question marking. Kalau eyebrow yang jadi pembeda, ngehapusnya bakal bikin error itu makin parah — dan kamu nggak bakal tau kenapa.
2. **Referensi [7] kamu sendiri** itu soal mouth shapes. Ngehapus semua lip landmark = ngebuang persis hal yang kamu kutip sebagai justifikasi NMS.
3. **Biayanya 5 dimensi.** Identity leak dari 5 rasio ternormalisasi itu praktis nol. Satu training run buat ngubah asumsi jadi pengukuran.

> **Prinsipnya:** *"kami ukur dan hasilnya nol"* jauh lebih kuat dari *"kami lihat videonya terus mutusin nggak usah diukur."*

#### Cheek landmarks (20 titik): buang

Cheek encode **lebar wajah** — itu identitas hampir murni. Deformasi pipi waktu senyum itu halus, dan dengan rekaman yang mukanya netral, praktis nggak ada. Cost tinggi, benefit nol.

#### Opsional (jangan dulu): per-sequence baseline subtraction

Kalau blok D ternyata masih bocor identity, bisa ditambah: kurangi tiap fitur blok D sama median-nya sendiri dalam sequence itu. Jadi yang diukur *"seberapa jauh dari netral **orang ini**"*, bukan nilai absolut.

> ⚠️ **Tradeoff:** kalau ada sign yang mulutnya kebuka **konstan** sepanjang sequence, baseline-subtraction bakal ngehapus sinyalnya. Buat isolated word 30 frame, risikonya nyata. **Skip dulu** — coba versi biasa, baru pertimbangin kalau blok D masih bocor.

### 4.7 Yang dibuang dari pose

**Buang:** hip (23,24), knee (25,26), ankle (27,28), foot.

**Alasan:** di rekaman upper-body semuanya extrapolated dari bahu — fungsi deterministik dari titik yang udah ada. Nol informasi baru, tapi bawa bias per-orang. Detail mekanismenya di Bagian 4.1 (Sebab: kenapa pose juga kena).

**Keep:** shoulder (11,12), elbow (13,14), wrist (15,16) — ini articulator beneran, gerakannya bagian dari sign.

### 4.8 ⚠️ Isu yang perlu dicek: shoulder movement vs shoulder normalization

Definisi NMS di paper kamu termasuk **shoulder movement**. Tapi normalisasi kamu pakai shoulder midpoint (centering) + shoulder distance (scaling). Berarti kamu pakai bahu sebagai **kerangka acuan** — dan gerakan bahu diukur relatif terhadap bahu itu sendiri.

| Jenis gerakan | Nasibnya |
|---|---|
| Translation (bahu geser) | Dihapus centering |
| Scale (lean maju/mundur → lebar apparent berubah) | Dihapus scaling |
| **Rotation (bahu miring)** | **Tetap ada** ✅ |

Jadi shoulder tilt selamat, tapi shrug/lean sebagian ke-cancel.

**Kenapa perlu dicek:** kalau ada kelas yang bergantung ke shoulder movement, sinyalnya mungkin udah hilang duluan sebelum model lihat. Kalau iya, itu limitation yang harus ditulis — **bukan diem-diem diklaim ketangkep.**

### 4.9 Prasyarat implementasi

> **Kamu butuh index mapping dari script preprocessing** — 74 face point mana aja yang dipilih dari 468, dan urutannya di array flattened.
>
> - **Kalau list-nya masih ada di kode** → cukup derive dari array `(30, 447)` yang udah ada. **Nggak perlu re-extract video.**
> - **Kalau ilang** → fallback re-extract dari Drive. Ini ngubah timeline signifikan, jadi **cek ini duluan sebelum yang lain.**

---

## Bagian 5 — Ablation Round 2

### 5.1 Empat arm, tiga run baru

| # | Arm | Blok | Dim | Status |
|---|---|---|---|---|
| 1 | Hand-only | A | 126 | ✅ udah ada (67.54%) |
| 2 | + articulator pose | A+B | 144 | run baru |
| 3 | + postural NMS | A+B+C | 147 | run baru |
| 4 | + facial NMS | A+B+C+D | 152 | run baru |

### 5.2 Kenapa tangga-nya disusun begini

Tiap arm cuma nambah **satu** blok. Jadi selisih antar arm yang berurutan itu murni kontribusi blok itu — nggak ketuker sama yang lain.

| Delta | Pertanyaan yang dijawab |
|---|---|
| 1 → 2 | Gerakan lengan/bahu nyumbang di luar tangan? |
| 2 → 3 | **Head orientation nyumbang?** ← klaim NMS inti |
| 3 → 4 | Detail wajah nyumbang di atas head orientation? |

> **Kenapa ini gantiin class subgrouping:** rencana awal itu nge-tag kelas jadi "head-posture driven" vs "expression driven" terus bandingin. Tapi split-nya bakal ~18/2 — nggak ada power statistik sama sekali.
>
> Struktur arm ini jawab pertanyaan yang sama **di level fitur**, langsung, tanpa tagging dan tanpa risiko post-hoc. Lebih bersih.

### 5.3 Breakdown yang tetap dipakai: split 20/20

NMS-focused vs standard manual — split yang **udah ada di paper**.

**Kenapa yang ini aman dipakai:**
- **Gratis** — tinggal slice prediksi, nol training run
- **Balanced** — 20 vs 20
- **Pre-registered secara alami** — didefinisiin waktu desain vocabulary, jauh sebelum angka LOSO keluar. Nggak ada tuduhan cherry-picking.

**Testnya:** kalau blok C beneran nyumbang, gain-nya harus **kekonsentrasi di 20 kelas NMS-focused**, bukan nyebar rata ke 40. Kalau nyebar rata, kemungkinan yang naik itu sesuatu yang lain, bukan NMS.

### 5.4 Kondisi yang dijaga konstan

| Kondisi | Kenapa |
|---|---|
| Mirror augmentation ON di semua arm | Kalau beda treatment, perbandingannya rusak |
| HP dibekukan (32/256, dropout 0.3, L2 1e-4) | Biar yang dibandingin fitur, bukan tuning |
| Fold assignment identik | Beda fold = beda kesulitan |
| Epoch budget & early stopping identik | Beda training length = beda konvergensi |

Prinsipnya sama kayak Step 1: **cuma satu variabel yang boleh berubah.**

### 5.5 ⚠️ Caveat hyperparameter

178 Hyperband trial kamu di-tune pakai **sequence-level split**. Artinya HP-nya secara efektif dioptimasi buat setting yang **membolehkan hafalan signer**. 32/256 + dropout 0.3 belum tentu optimal buat signer-independent.

> **Jangan retune di LOSO fold terus report yang terbaik.** Itu **selecting on test** — persis error yang lagi kamu perbaiki, cuma pindah tempat. Kalau mau retune, harus nested (leave-one-signer-out di dalam training signer buat validation), dan dengan 4 signer itu tipis banget.

Opsi jujur & murah: bekukan HP, tulis di limitation:
> *"Hyperparameters were selected under a non-signer-disjoint protocol and were not re-optimized for the signer-independent setting; the reported LOSO figures may therefore understate achievable signer-independent performance."*

### 5.6 Pelaporan statistik

**n=4 fold itu tipis.** Paired t-test dengan n=4 punya power rendah, dan p-value-nya rapuh.

**Yang lebih meyakinkan reviewer:** konsistensi arah + magnitude per fold. "4/4 fold, −10.4 s/d −17.4" itu bukti yang lebih kuat daripada p = 0.0029 dari n=4, karena nunjukin efeknya konsisten bukan kebetulan satu-dua fold.

Tambahin juga variance: hand-only ±2.70% itu **terendah dari semua arm** — bukti tambahan bahwa dia bukan cuma lebih akurat tapi lebih stabil.

---

## Bagian 6 — Paper Additions

### 6.1 🚫 Aturan yang nggak boleh dilanggar

**Ubah klaim ke depan = sah. Ubah catatan apa yang dilakukan = enggak.**

Section III-A1 (vocabulary selection) **jangan diubah.** Klasifikasi tetap *"facial expression, head posture, or shoulder movement"* — itu emang niat desainnya, dan itu fakta historis.

> **Kenapa ini bukan cuma soal etika, tapi juga praktis:** kelasnya tetap *marah*, *sedih*, *bingung*. Kalau rasionale-nya diganti jadi "dipilih karena postur dan orientasi kepala", reviewer baca itu terus lihat daftar isinya kata emosi → langsung nggak nyambung. Kamu malah ngundang pertanyaan yang lagi dihindarin, tapi sekarang tanpa jawaban siap.

Yang bener: biarin methodology apa adanya, **tambahin apa yang ditemuin.**

### 6.2 Presisi klaim

| ❌ Nggak bisa diklaim | ✅ Yang beneran dibuktiin |
|---|---|
| "Facial NMS nggak membantu BISINDO recognition" | "Di dataset ini, face landmark nggak bawa sinyal ekspresif; masukin blok high-dimensional yang uninformative merusak signer generalization" |

**Kenapa bedanya besar:** yang pertama klaim tentang **bahasa isyarat** — butuh dataset dengan ekspresi natural buat dibuktikan, dan kamu nggak punya. Yang kedua klaim tentang **representasi dan data** — itu yang kamu punya buktinya.

### 6.3 Judul selamat

Head orientation, postur kepala, gerakan bahu — **itu semua non-manual markers.** *"Integrating Non-Manual Markers"* tetap jujur, asal papernya presisi bahwa NMS yang ketangkep itu **postural**, bukan facial.

Nggak perlu ganti judul, cukup abstract-nya spesifik.

### 6.4 Kontribusi baru buat Future Work

Temuannya: **milih vocabulary NMS-focused itu nggak cukup.** Kamu juga butuh **protokol elicitation** yang beneran ngasilin NMS natural.

Mekanismenya: signer non-native yang diminta meragain kata secara terisolasi bakal fokus ke handshape yang bener — ekspresi wajahnya netral karena nggak ada konteks emosional beneran. Akibatnya face landmark yang direkam cuma geometri per orang.

Rekomendasi konkret: rekam pakai native signer, atau elicit lewat konteks kalimat, bukan citation form.

> Ini nggak ada di future work paper manapun yang kamu sitir. Peta edit lengkap per section ada di dokumen terpisah (`SignLingo_Paper_Revision_Map.md`).

### 6.5 Jangan sampai ketutup

Mirror result itu temuan paling bersih dan paling actionable di seluruh Round 1. 19.33% → 54.90% dari satu augmentasi.

Cerita ablation itu lebih rumit dan lebih panjang, jadi gampang banget nyedot semua perhatian. Pastiin mirror dapet space-nya sendiri.

---

## Bagian 7 — Carry-Over dari Roadmap Round 1

| Fix | Isi | Kenapa ringan |
|---|---|---|
| **Fix 3** | Model size / Flex delegate wording | Rephrase-only. Akar masalahnya kontradiksi kalimat, bukan kekurangan data. Teks revisi udah siap tempel |
| **Fix 4** | Latency N=1 → N=10 | Script udah support `--n_sequences`. Klaim "400ms budget" nggak kuat kalau cuma 1 sample |

Kerjain sambil nunggu training Bagian 5 jalan — nggak ada dependency.

---

## Bagian 8 — Urutan Kerja & Alasannya

| Urutan | Step | Kenapa di posisi ini |
|---|---|---|
| 1 | Permuted control | **GATE** — kalau gagal, semua angka Round 1 nggak valid. Nggak ada gunanya kerja di atas fondasi yang belum dicek |
| 2 | Mirror audit | 5 menit, tapi nentuin apakah magnitude Round 1 bisa dipercaya |
| 3 | Inspeksi rekaman | Hasilnya ngubah ekspektasi blok D — lebih baik tau sebelum nulis extractor |
| 4 | Cek index mapping | Kalau ilang, timeline berubah drastis. Cek sebelum commit ke rencana |
| 5 | Tulis extractor | Baru mulai kerja beneran setelah semua asumsi dicek |
| 6 | 3 training run | Long pole, jalanin overnight |
| 7 | Fix 3 & 4 | Sambil nunggu training — nggak ada dependency |
| 8 | Paper | Terakhir, setelah semua angka masuk |

**Pola umumnya:** yang murah tapi bisa ngebatalin semuanya dikerjain duluan. Yang mahal dikerjain setelah semua asumsinya dikonfirmasi.

---

## Bagian 9 — Checklist

### Sudah tertutup
- ☒ **Class coverage** — Scenario A, semua 40 kelas × 4 signer, nggak ada fold zero-shot
- ☒ **Face block: hapus atau arm** — jadi arm (blok D, 5 dim). Ngukur nol lebih kuat dari ngasumsi nol
- ☒ **Subgroup tagging** — dibuang, pakai split 20/20 yang udah ada. Split 18/2 nggak ada power
- ☒ **Shoulder normalization** — tetap dipakai, extraction script nggak diubah. Blok C/D kebal (sudut & rasio invariant)

### Masih terbuka
- ☐ **Index mapping 74 face landmark** — masih ada di script? **Cek duluan**, nentuin timeline
- ☐ **Mirror ON di semua 4 arm?** Dan hand-only 67.54% itu with-mirror atau without? Kalau without, gap 13.93pp bukan apple-to-apple
- ☐ **Mouthing ada nggak** di rekaman? Nentuin ekspektasi blok D
- ☐ **z-coordinate usable** buat pitch? Kalau noisy, drop pitch, sisain roll+yaw
- ☐ **Framing video konsisten?** Kalau hip keliatan di sebagian video doang, itu amunisi tambahan buat argumen identity channel
- ☐ **Shoulder movement** masih ada sinyalnya setelah shoulder normalization?
