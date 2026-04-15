# SmartSchool LMS – Adaptive Learning Platform

## 1. Background / Context
Sekolah dan lembaga kursus masih menggunakan LMS statis (seperti Google Classroom atau Moodle tanpa personalisasi). Materi yang sama diberikan ke semua siswa, padahal kemampuan mereka berbeda-beda. Akibatnya, siswa cepat bosan atau merasa tertinggal.

## 2. Problem Statement
- Guru tidak bisa memberikan materi berbeda secara manual ke 30+ siswa per kelas.
- Tidak ada mekanisme otomatis yang menyesuaikan tingkat kesulitan soal berdasarkan performa siswa.
- Siswa yang sudah mahir terhambat, siswa yang lemah tidak mendapat remedial otomatis.
- Laporan perkembangan siswa hanya nilai akhir, tanpa insight area yang perlu diperbaiki.

## 3. Goals / Objectives
- Meningkatkan ketuntasan belajar (passing rate) dari 70% menjadi 85% dalam 1 semester.
- Mengurangi waktu guru dalam menyusun soal remedial sebesar 80%.
- Memberikan rekomendasi materi berikutnya dengan akurasi >90% berdasarkan mastery level.

## 4. Proposed Solution
Platform LMS dengan **adaptive learning engine** yang menggunakan Item Response Theory (IRT) dan Knowledge Tracing. Sistem akan:
- Memberikan pre-test untuk mengukur baseline.
- Menentukan level kesulitan materi dan soal secara dinamis.
- Menyusun jalur belajar (learning path) unik per siswa.

## 5. System Design / Architecture
[User (Siswa/Guru)] → [Frontend (React/Next.js)] → [API Gateway]
→ [Adaptive Engine Service] → [Database (PostgreSQL)]
→ [Redis untuk session & cache] → [Analytics Service]

**Komponen:**
- Adaptive Engine: Python/FastAPI, library `scikit-learn` untuk IRT.
- LMS Core: Laravel atau NestJS.
- Frontend: Next.js + Tailwind.
- Storage: AWS S3 untuk materi video.

## 6. Detailed Design
**Logic internal:**
1. Setiap soal memiliki parameter: difficulty (b), discrimination (a), guessing (c) – model 3PL IRT.
2. Setiap jawaban siswa mengupdate estimasi kemampuan (θ).
3. Soal berikutnya dipilih dengan fungsi informasi maksimal (Fisher Information).
4. Jika θ < threshold (misal -1), sistem beri materi remedial + soal lebih mudah.
5. Jika θ > +1, sistem beri materi pengayaan.

**Edge cases:**
- Siswa mengisi asal-asalan: deteksi dengan response time < 3 detik + pola acak → flag ke guru.
- Tidak ada cukup data awal: gunakan default difficulty berdasarkan grade.

## 7. Trade-offs & Alternatives
| Alternatif | Kelebihan | Kekurangan |
|------------|-----------|-------------|
| Bayesian Knowledge Tracing (BKT) | Lebih simpel | Tidak memodelkan difficulty soal secara kontinu |
| Deep Learning (DKVMN) | Akurasi tinggi | Butuh data besar & komputasi berat |

**Dipilih IRT** karena interpretasi mudah dan cocok untuk jumlah siswa 100-1000.

## 8. Risks & Mitigation
| Risiko | Mitigasi |
|--------|-----------|
| Adaptive engine lambat (>500ms per request) | Cache hasil prediksi, gunakan async task untuk update massal |
| Guru tidak percaya rekomendasi sistem | Tampilkan penjelasan (misal: "Soal ini diberikan karena kemampuan Anda di top X masih lemah") |
| Data privasi siswa | Enkripsi nilai, role-based access, compliance dengan UU PDP |

## 9. Metrics / Evaluation
- **Accuracy rekomendasi**: Precision@3 topik berikutnya ≥ 85%
- **Latency**: Waktu adaptasi soal < 300ms (p95)
- **User engagement**: Rata-rata sesi belajar per minggu naik 30%
- **Belajar tuntas**: % siswa yang mencapai mastery level dalam 2 minggu

## 10. Implementation Plan
- [ ] Minggu 1: Setup database & IRT model training pipeline (data sintetis)
- [ ] Minggu 2: API adaptive engine (FastAPI) + unit test
- [ ] Minggu 3: Integrasi ke LMS core (fitur quiz & materi)
- [ ] Minggu 4: Frontend dashboard siswa & guru
- [ ] Minggu 5: UAT dengan 1 sekolah mitra
- [ ] Minggu 6: Deployment & monitoring (Prometheus + Grafana)

**Tech stack rekomendasi:** Next.js (FE), NestJS (BE), PostgreSQL, Redis, Python/FastAPI (AI engine), Docker, Kubernetes.
**Kompleksitas:** Hard
**Role:** full Stack Developer