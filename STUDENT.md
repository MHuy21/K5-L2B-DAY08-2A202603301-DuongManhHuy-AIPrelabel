# Thông tin học viên

- Họ và tên: Dương Mạnh Huy
- MSHV: 2A202603301
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: MHuy21
- Repo bài nộp (URL): [github.com/MHuy21/K5-L2B-DAY08-2A202603301-DuongManhHuy-AIPrelabel.git](https://github.com/MHuy21/K5-L2B-DAY08-2A202603301-DuongManhHuy-AIPrelabel.git)
- Commit chốt bài (hash): 07e3b3f
- Máy: Windows 11 Home, CPU only — không có GPU CUDA/MPS (AMD Radeon 780M tích hợp), RAM 32 GB; Python 3.12.10, torch 2.15.0+cpu, ultralytics 8.4.215
- CVAT (Docker): v2.81.0 (Docker Desktop, http://localhost:8080)

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 87.0% / 2.4% / 10.6% (mẫu số: 622 box máy); added: 3.8% (mẫu số: 578 đối tượng)
- So với gold: precision / recall của **máy** 54.5% (42/77) / 76.4% (42/55)
- Bắt bẫy: 5/12 = 41.7%
- Thời gian: gán tay 5 phút/ảnh (20'/4); duyệt có pre-label 1.67 phút/ảnh (40'/24)
- Lỗi QC: 100% theo ảnh (5/5), 10.9% theo đối tượng (13/119)
- **Quyết định:** Rework — lý do một dòng: không Hold vì 13 lỗi QC đều sửa được theo guideline hiện có; Rework vì ảnh lỗi 5/5 = 100% > 2% và bắt bẫy 5/12 = 41.7% < 80% (lô duyệt chưa đáng tin).
