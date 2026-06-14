# GAN + Quantum Payload Mask — Adversarial Attack on SRNet

Notebook: **`GAN_Quantum_Final.ipynb`** (Track 1, ADL2122 / CSC14101).

Notebook **tự nhận biết môi trường** — chạy được cả Google Colab (online) và máy local (offline) mà không cần sửa code, chỉ cần chuẩn bị đúng cấu trúc thư mục.

---

## 1. Cấu trúc thư mục

### 1.1. Local (chạy trên máy bạn)

```
<thư mục dự án>/
├── GAN_Quantum_Final.ipynb
├── README.md
├── Datasets/
│   ├── boss_256_0.4/                 ← TRAIN set
│   │   ├── cover/                    (vd: 1.png, 2.png, ...)
│   │   └── stego/                    (cùng tên file với cover/)
│   └── boss_256_0.4_test/            ← TEST set
│       ├── cover/
│       └── stego/
└── checkpoints/                       (tự tạo khi train)
    ├── srnet_pretrained.pth
    ├── generator_best.pth
    ├── generator_last.pth
    ├── generator_epoch_25.pth
    ├── ...
    ├── discriminator_best.pth
    └── ...
```

### 1.2. Google Colab (online)

Trên Drive bạn để như sau:

```
MyDrive/
└── ADL2122/                           ← khớp DRIVE_ROOT trong Cell 1
    ├── GAN_Quantum_Final.ipynb        (mở từ đây)
    ├── Datasets/
    │   ├── boss_256_0.4/
    │   │   ├── cover/
    │   │   └── stego/
    │   └── boss_256_0.4_test/
    │       ├── cover/
    │       └── stego/
    └── checkpoints/                    (tự tạo)
```

Khi chạy, Cell 1 sẽ tự:
1. Mount Drive vào `/content/drive/MyDrive`
2. Copy 2 folder dataset từ Drive sang `/content/Datasets/` (đọc local nhanh hơn Drive ~10×)
3. Tạo `MyDrive/ADL2122/checkpoints/` để lưu model persistent (không mất khi runtime disconnect)

Nếu folder dự án trên Drive của bạn tên khác (ví dụ `MyDrive/CS300/...`), sửa biến `DRIVE_ROOT` trong **Cell 1**:
```python
DRIVE_ROOT = "/content/drive/MyDrive/<tên_folder_của_bạn>"
```

> **Yêu cầu bắt buộc:** file ở `cover/` và `stego/` phải **trùng tên** từng cặp. Quantum Payload Mask được tính bằng `|cover - stego| > 0`.

---

## 2. Setup môi trường

### 2.1. Local

```bash
# Tạo venv
python -m venv venv
# Windows
venv\Scripts\activate
# macOS / Linux
source venv/bin/activate

# Cài torch (xem https://pytorch.org/get-started/locally/ cho phiên bản CUDA của bạn)
pip install torch torchvision

# Các package còn lại
pip install numpy matplotlib pillow tqdm jupyter notebook
```

### 2.2. Colab

Không cần cài gì — Colab có sẵn tất cả. Chỉ cần:
- **Runtime** → **Change runtime type** → **GPU** (T4 free là đủ).
- Chạy notebook lần lượt từ Cell 1.

---

## 3. Hyper-parameters tự động

Notebook **tự điều chỉnh** dựa trên môi trường:

| | Local (offline) | Colab (online) |
|---|---|---|
| `PRETRAIN_EPOCHS` | 15 | **25** |
| `GAN_EPOCHS` | 100 | **200** |
| `SAVE_INTERVAL` | 25 | 50 |
| `NUM_WORKERS` | **0** (tránh DataLoader treo trên Windows) | 2 |

Tất cả set trong **Cell 2**, bạn có thể override nếu muốn.

---

## 4. Thứ tự chạy notebook

| Cell | Nội dung | Thời gian (T4 GPU) |
|------|----------|-------------------|
| 1 | Environment setup (detect Colab, mount Drive, copy dataset) | 5-15 phút (lần đầu Colab) / <1s (local) |
| 2 | Imports + config | <1s |
| 3 | Dataset (train + test loader) | vài giây |
| 4 | Định nghĩa SRNet | <1s |
| 5 | Định nghĩa Generator + Discriminator | <1s |
| 6 | Quantum Phase Lock | <1s |
| 7 | **Pre-train SRNet** | ~10-30 phút |
| 8 | **GAN adversarial training** | 1-3 giờ |
| 9 | Evaluation + Visualization | <1 phút |
| 10 | Standalone inference helper | <1s |

---

## 5. Checkpoint & Resume

Lưu trong `checkpoints/` (local) hoặc `MyDrive/ADL2122/checkpoints/` (Colab):

| File | Khi nào lưu |
|------|-------------|
| `srnet_pretrained.pth` | Sau pretrain (best accuracy) |
| `generator_epoch_<N>.pth` | Mỗi `SAVE_INTERVAL` epoch |
| `generator_last.pth` | Sau mỗi epoch (rolling) |
| `generator_best.pth` | Khi ASR đạt cao nhất |

**Resume training:** Trong Cell 8, sửa biến `RESUME_FROM`:
```python
RESUME_FROM = 50   # Load generator_epoch_50.pth và tiếp tục
```

---

## 6. Nguyên nhân hay khiến notebook bị "đứng" và cách fix

### 6.1. Treo ở batch đầu tiên khi pretrain SRNet
- **Nguyên nhân:** `num_workers > 0` + Windows (hoặc Drive chưa copy về local trên Colab).
- **Fix:** Đã set `NUM_WORKERS = 0` mặc định cho local. Trên Colab, đảm bảo Cell 1 chạy xong (dataset copy về `/content/Datasets/`) trước khi sang Cell 3.

### 6.2. CUDA out of memory
- **Fix:** Trong Cell 2 giảm `BATCH_SIZE` từ 8 xuống 4 hoặc 2.

### 6.3. SRNet pretrain accuracy thấp (<70%)
- **Fix:** Tăng `PRETRAIN_EPOCHS`. Hoặc kiểm tra cặp cover/stego có đúng (cùng nội dung, chỉ khác ở pixel payload) không.

### 6.4. ASR không tăng (mãi ~50%)
- SRNet chưa đủ tốt → tăng `PRETRAIN_EPOCHS`.
- Tăng `LAMBDA_ADV` (15-20) hoặc `EPSILON` (16/255).

### 6.5. Trên Colab bị mất kết nối giữa chừng
- Vào lại Cell 1, mount Drive lại.
- Trong Cell 8 set `RESUME_FROM = <epoch_gần_nhất>` để tiếp tục.

---

## 7. Metric

| Metric | Ý nghĩa | Target |
|--------|---------|--------|
| **ASR** | % adv stego bị SRNet nhận nhầm thành cover | >60% (goal 100%) |
| **PSNR** | Chất lượng ảnh adv vs stego (dB) | >35 dB |
| **MER** | % pixel payload còn nguyên vẹn | **100%** (đảm bảo bởi Quantum Mask) |

---

## 8. Track 2 (Diffusion + Quantum Seal)

Notebook này chỉ là Track 1 (GAN-based). Track 2 do Nguyễn Nhật Long phụ trách, nằm trong notebook riêng.
