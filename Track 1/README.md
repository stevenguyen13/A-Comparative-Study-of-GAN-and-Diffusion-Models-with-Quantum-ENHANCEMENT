TRACK 1: ADVERSARIAL STEGANOGRAPHY VỚI GAN VÀ QUANTUM PAYLOAD MASK

Đường dẫn Notebook: GAN_Quantum_Final.ipynb

Dự án này là phân hệ Track 1 (thuộc học phần CSC14101), tập trung vào việc tạo ra các nhiễu đối kháng (adversarial perturbations) bằng mạng sinh đối kháng (GAN) nhằm vô hiệu hóa hệ thống phân tích giấu tin. Trọng tâm của nghiên cứu là hướng các cuộc tấn công đối kháng vào mô hình CLIP. Đặc biệt, nghiên cứu tích hợp Quantum Payload Mask như một cơ chế định tuyến đạo hàm (Gradient Router) để bảo vệ tuyệt đối tính toàn vẹn của dữ liệu được giấu (MER = 100%).

Notebook được thiết kế để tự nhận biết môi trường, có thể thực thi trơn tru trên cả Google Colab (đám mây) và máy tính cá nhân (local) mà không yêu cầu tinh chỉnh mã nguồn.

--------------------------------------------------
1. KIẾN TRÚC CÂY THƯ MỤC

Để quá trình nạp dữ liệu và checkpoint không bị đứt gãy, hãy đảm bảo cấu trúc thư mục được sắp xếp như sau:

[Chạy trên Local - Máy cá nhân]
Track 1/
|-- GAN_Quantum_Final.ipynb
|-- README.md
|-- Datasets/
|   |-- boss_256_0.4/                 (Tập Dữ liệu Train)
|   |   |-- cover/                    (vd: 1.png, 2.png, ...)
|   |   |-- stego/                    (bắt buộc trùng tên file với thư mục cover)
|   |-- boss_256_0.4_test/            (Tập Dữ liệu Test)
|       |-- cover/
|       |-- stego/
|-- checkpoints/                      (Tự động tạo ra trong lúc train)
    |-- clip_pretrained.pth
    |-- generator_best.pth
    |-- generator_last.pth
    |-- discriminator_best.pth
    |-- test_results                  (Kết quả test)
    |-- |-- metrics_summary.json      (Báo cáo tổng hợp các chỉ số đánh giá)
    |-- |-- transfer_summary.json     (Báo cáo đánh giá chuyển giao - Mở rộng 125%)
    |-- |-- *.png		      (Các file ảnh kết quả)
[Chạy trên Google Colab - Trực tuyến]
MyDrive/
|-- ADL2122/                          (Khớp với biến DRIVE_ROOT trong Cell 1)
    |-- GAN_Quantum_Final.ipynb       (Mở notebook từ đây)
    |-- Datasets/                     (Cấu trúc bên trong tương tự Local)
    |-- checkpoints/                  (Tự động tạo)

Lưu ý khi chạy Colab: Khi chạy Cell 1, hệ thống sẽ tự động mount Google Drive, sao chép dữ liệu từ Drive sang bộ nhớ cục bộ của Colab (/content/Datasets/) để tăng tốc độ nạp dữ liệu, và tự động lưu model checkpoint liên tục.

--------------------------------------------------
2. THIẾT LẬP MÔI TRƯỜNG

[Môi trường Local]
- Khởi tạo môi trường ảo: python -m venv venv
- Kích hoạt (Windows): venv\Scripts\activate
- Kích hoạt (macOS/Linux): source venv/bin/activate
- Cài đặt PyTorch: Truy cập https://pytorch.org/get-started/locally/ để lấy lệnh cài đặt khớp với phiên bản CUDA.
- Cài đặt thư viện bổ trợ: pip install numpy matplotlib pillow tqdm jupyter notebook

[Môi trường Colab]
Môi trường đã tích hợp sẵn thư viện. Chỉ cần vào Runtime -> Change runtime type -> Chọn GPU (Tesla T4).

--------------------------------------------------
3. QUẢN LÝ SIÊU THAM SỐ

Notebook tự động tinh chỉnh siêu tham số tùy thuộc vào tài nguyên phần cứng:
- PRETRAIN_EPOCHS: 100 (Local) / 200 (Colab)
- GAN_EPOCHS: 30 (Local) / 50 (Colab)
- SAVE_INTERVAL: 10 (Local) / 10 (Colab)
- NUM_WORKERS: 0 (Local - tránh lỗi treo DataLoader trên Windows) / 2 (Colab)

Các siêu tham số tối ưu hóa hàm Loss (Hyper-parameters lõi):
- EPSILON: Biên độ dao động tối đa của U-Net Generator.
- PERT_BUDGET_L2: Ngưỡng ngân sách c cho hàm Hinge Loss.
- LAMBDA_ADV, LAMBDA_DIST, LAMBDA_GAN: Các trọng số điều phối sự cân bằng của hàm mất mát tổng thể.

Các tham số này được định nghĩa tại Cell 2 và có thể ghi đè (override) tùy ý.

--------------------------------------------------
4. QUY TRÌNH THỰC THI NOTEBOOK

Toàn bộ quá trình chạy đã được tích hợp thanh tiến trình (tqdm), giúp bạn dễ dàng quan sát thời gian và tiến độ huấn luyện của từng epoch theo thời gian thực.

Khi thiết lập thư mục hoàn tất, hãy chọn "Restart Kernel and Run All Cells" trên giao diện Jupyter/Colab để hệ thống tự động chạy xuyên suốt toàn bộ pipeline.

- Cell 1: Khởi tạo môi trường (Nhận dạng Colab, Mount Drive, Sao chép Dataset).
- Cell 2: Nạp thư viện (Imports) và Cấu hình các hằng số (Config).
- Cell 3: Xử lý Dữ liệu (Khởi tạo train & test loader).
- Cell 4: Xây dựng kiến trúc mô hình CLIP (Mục tiêu tấn công).
- Cell 5: Xây dựng kiến trúc U-Net Generator và PatchGAN Discriminator.
- Cell 6: Khởi tạo Quantum Phase Lock thông qua hàm apply_quantum_lock().
- Cell 7: Tiền huấn luyện (Pre-train) mô hình mục tiêu.
- Cell 8: Huấn luyện Đối kháng GAN (Adversarial Training).
- Cell 8b: Nạp Checkpoints (Nếu muốn bỏ qua quá trình train để đánh giá ngay).
- Cell 9: Đánh giá Thống kê (Evaluation) & Biểu đồ trực quan.
- Cell 10: Tích hợp hàm attack_one_image() phục vụ quá trình chạy suy luận (inference) độc lập trên từng ảnh.
- PHẦN MỞ RỘNG (125%): Trong luồng thực thi của notebook có tích hợp thêm phần đánh giá khả năng chuyển giao nhiễu (Transferability) trên kiến trúc phân tích XuNet (kịch bản Black-box attack) nhằm kiểm chứng độ bền vững của mô hình đề xuất ngoài thực tế.

--------------------------------------------------
5. QUẢN LÝ CHECKPOINTS & RESUME TRAINING

Mô hình được lưu tự động tại thư mục checkpoints/.
- clip_pretrained.pth: Lưu ngay sau quá trình pretrain.
- generator_epoch_<N>.pth: Lưu định kỳ theo tham số SAVE_INTERVAL.
- generator_last.pth: Cập nhật liên tục sau mỗi epoch.
- generator_best.pth: Cập nhật khi chỉ số ASR đạt mức cao kỷ lục.
- discriminator_best.pth: Trọng số tốt nhất của mạng phân biệt.
- metrics_summary.json: Tệp xuất dữ liệu tự động, lưu trữ so sánh định lượng (ASR, PSNR, L2, MER) giữa GAN và các thuật toán Baseline (FGSM, PGD).
- transfer_summary.json: Tệp xuất dữ liệu ghi nhận kết quả đánh giá ở phần mở rộng 125% (Black-box attack trên XuNet).

- Cơ chế Resume: Để tiếp tục huấn luyện từ một checkpoint cũ, điều chỉnh biến RESUME_FROM tại Cell 8 (ví dụ: RESUME_FROM = 10 để nạp file generator_epoch_10.pth).

--------------------------------------------------
6. CÁC CHỈ SỐ ĐO LƯỜNG (METRICS)

- ASR (Attack Success Rate): Xác suất mô hình CLIP phân loại sai ảnh adversarial stego thành cover. Kỳ vọng tuyệt đối: 100%.
- PSNR (Peak Signal-to-Noise Ratio): Đo lường độ giảm sút chất lượng ảnh. Mục tiêu: > 35 dB.
- MER (Message Extraction Rate): Tỷ lệ các pixel mang thông điệp ẩn được giữ nguyên vẹn. Đảm bảo đạt 100% nhờ cơ chế Quantum Mask.

*Lưu ý về phương pháp thống kê: Trong việc đánh giá các chỉ số định lượng, quá trình tính toán thống kê (phương sai, độ lệch chuẩn) xử lý toàn bộ tập dữ liệu kiểm thử như một quần thể (population) hoàn chỉnh, chứ không xem như một mẫu (sample), nhằm đưa ra các giới hạn đo lường chuẩn xác nhất.

--------------------------------------------------
7. DỮ LIỆU ĐẦU VÀO & PRE-TRAINED MODELS

Để tuân thủ giới hạn kích thước tệp của Github, hệ thống mã nguồn không bao gồm thư mục Datasets/ và checkpoints/. Bạn hãy tải tập dữ liệu về qua liên kết Google Drive dưới đây:
👉 Link tải: https://drive.google.com/drive/folders/1siVE-WqbtNGUQSDUebamOO46dLnov3SO?usp=sharing

Hướng dẫn: Sau khi tải về, giải nén và đặt hai thư mục Datasets/ cùng checkpoints/ thẳng vào cấp thư mục gốc Track 1/ trước khi tiến hành chạy Jupyter notebook.
