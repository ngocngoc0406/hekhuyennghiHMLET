````markdown
# HMLET – Hướng dẫn cài đặt & sử dụng

## 1. Giới thiệu

**HMLET (Hierarchical Multi-Level Embedding Learning Transformer)** là mô hình gợi ý (Recommender System) theo hướng *learning-to-rank*.

- Hỗ trợ datasets:
  - `gowalla`
  - `amazon-book`
  - `yelp2018`

- Các biến thể mô hình:
  - `HMLET_All`
  - `HMLET_End`
  - `HMLET_Front`
  - `HMLET_Middle`

🔗 Source code: https://github.com/jeongwhanchoi/HMLET

---

## 2. Yêu cầu hệ thống

### 2.1 Phần cứng (khuyến nghị)

- GPU NVIDIA (tuỳ chọn, hỗ trợ CUDA)
  - Nếu VRAM thấp (~4GB) → cần giảm cấu hình (xem mục 5)
- RAM:
  - Tối thiểu: 8GB
  - Khuyến nghị: 16GB
- Ổ đĩa trống: 5–10GB

### 2.2 Phần mềm

- Windows 10 / 11
- Python 3.10 hoặc 3.11
- PyTorch (CPU hoặc CUDA)
- Thư viện:
  - `numpy`
  - `pandas`
  - `matplotlib`

---

## 3. Thiết lập môi trường

### 3.1 Tạo virtual environment (PowerShell)

```powershell
cd D:\HMLET-main
python -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install -U pip
````

### 3.2 Cài đặt thư viện

**Nếu có `requirements.txt`:**

```powershell
pip install -r requirements.txt
```

**Nếu không có:**

```powershell
pip install numpy pandas matplotlib
pip install torch --index-url https://download.pytorch.org/whl/cpu
```

> ⚠️ Nếu dùng GPU, cài PyTorch đúng phiên bản CUDA theo hướng dẫn chính thức.

---

## 4. Dataset

Giá trị hợp lệ:

```bash
--dataset {gowalla, amazon-book, yelp2018}
```

Ví dụ cấu trúc thư mục:

```
./data/gowalla
./data/amazon-book
./data/yelp2018
```

---

## 5. Huấn luyện (Train)

### 5.1 Cú pháp cơ bản

```bash
python train.py --dataset gowalla --model HMLET_Front
```

### Model hợp lệ

```bash
--model {HMLET_All, HMLET_End, HMLET_Front, HMLET_Middle}
```

---

### 5.2 Cấu hình cho GPU yếu (4GB VRAM)

**Trường hợp 1:**

```bash
python train.py --dataset gowalla --model HMLET_Front --embedding_dim 64 --bpr_batch 128 --epochs 50
```

**Trường hợp 2 (nếu vẫn lỗi OOM):**

```bash
python train.py --dataset gowalla --model HMLET_Front --embedding_dim 32 --bpr_batch 64 --epochs 30
```

---

### 5.3 Chạy bằng CPU

```powershell
$env:CUDA_VISIBLE_DEVICES=""
python train.py --dataset gowalla --model HMLET_Front --embedding_dim 64 --bpr_batch 128 --epochs 50
```

---

## 6. Lưu checkpoint

Sau mỗi epoch, model được lưu tại:

```
./checkpoints/<MODEL>/<DATASET>/
```

Ví dụ:

```
./checkpoints/HMLET_Front/gowalla/HMLET_Front_gowalla_50.pth.tar
```

---

## 7. Load checkpoint để đánh giá

```bash
python train.py --dataset gowalla --model HMLET_Front --load_epoch 50 --epochs 50
```

> ⚠️ Đặt `--epochs = --load_epoch` để tránh train tiếp.

---

## 8. Vẽ biểu đồ (Loss / Precision / Recall / NDCG)

### 8.1 Cách thu thập dữ liệu

* Ghi log console → file `.txt`
* Parse sang `.csv`

### 8.2 File cần có

* `train_log.csv` → (epoch, loss)
* `precision.csv`
* `recall.csv`
* `ndcg.csv`

Mỗi file gồm các cột:

```
epoch, @10, @20, @30, @40, @50
```

---

### 8.3 Vẽ biểu đồ

```bash
python plot_all.py
```

---

## 9. Giải thích các chỉ số

* **Loss (BPR Loss)**
  → Độ sai lệch xếp hạng (càng thấp càng tốt)

* **Precision@K**
  → Tỷ lệ item đúng trong top-K

* **Recall@K**
  → Khả năng thu hồi item đúng trong top-K

* **NDCG@K**
  → Đánh giá chất lượng ranking (ưu tiên item đúng ở vị trí cao)

---

## 10. Lỗi thường gặp & cách khắc phục

| Lỗi                              | Nguyên nhân          | Cách sửa                                   |
| -------------------------------- | -------------------- | ------------------------------------------ |
| PowerShell lỗi `{dataset}`       | Dùng sai cú pháp     | Dùng trực tiếp `gowalla`                   |
| `invalid choice: coco`           | Dataset không hỗ trợ | Dùng: `gowalla`, `amazon-book`, `yelp2018` |
| CUDA out of memory               | VRAM không đủ        | Giảm `embedding_dim`, `bpr_batch`          |
| `Invalid device string: cuda:-1` | Sai cấu hình GPU     | Dùng: `$env:CUDA_VISIBLE_DEVICES=""`       |
| Matplotlib style lỗi             | Style không tồn tại  | Dùng `ggplot` hoặc `bmh`                   |
| `FileNotFoundError CSV`          | Thiếu file           | Kiểm tra cùng thư mục (`dir`)              |

---

## 11. Ghi chú

* Không dùng `{}` khi chạy lệnh thật
* Ưu tiên chạy CPU nếu GPU yếu
* Luôn kiểm tra đường dẫn dataset và checkpoint

```
```
