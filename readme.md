# Ứng dụng Học máy vào Phát hiện Hành vi Bất thường Người dùng dựa trên Log SIEM

* **Mã số đề tài:** `07-SV-2026-ATTT`
* **Đơn vị:** Khoa An toàn thông tin — Học viện Công nghệ Bưu chính Viễn thông (PTIT)
* **Giảng viên hướng dẫn:** TS. Phạm Hoàng Duy
* **Nhóm thực hiện:**
  * Trần Duy Phương — `B23DCAT243`
  * Vũ Văn Quang — `B23DCAT253`
  * Hoàng Minh Thành — `B23DCAT278`
  * Bùi Trọng Tú — `B23DCAT313`

---

## Tóm tắt nội dung nghiên cứu

Đề tài xây dựng và đánh giá quy trình **Phân tích hành vi người dùng và thực thể (UEBA)** nhằm phát hiện các sai lệch hành vi bất thường từ dữ liệu nhật ký tập trung của hệ thống SIEM:

1. **Chuẩn hóa dữ liệu đa nguồn:** Tổng hợp $32.770.222$ sự kiện từ 5 nguồn log (`logon`, `device`, `http`, `email`, `file`) kết hợp thông tin tổ chức `LDAP` của bộ dữ liệu **CERT Insider Threat r4.2** thành $499.120$ bản ghi `user-day`, giữ tách biệt trạng thái thiếu quan sát (`NULL`) với không phát sinh hoạt động ($0$).
2. **Mô hình hóa hành vi & Phát hiện bất thường:**
   * Xây dựng **đường cơ sở hành vi cá nhân nhân quả** (*Causal Personal Baseline*) trên cửa sổ trượt $60$ ngày (yêu cầu tối thiểu $21$ ngày quan sát tích lũy) để tính độ lệch hiệu chỉnh:
     $$z_{u,t}^{(j,\text{reg})} = \frac{x_{u,t}^{(j)} - \mu_{u,t}^{(j)}}{\max\left(\sigma_{u,t}^{(j)}, \sqrt{|\mu_{u,t}^{(j)}| + 1}\right)}$$
   * Sử dụng thuật toán **Isolation Forest** ($600$ cây) trên đặc trưng độ lệch giữ chiều tăng $\max(z, 0)$ để xếp hạng mức độ bất thường cho tài khoản đủ lịch sử, kết hợp nhóm tổ chức LDAP hoặc phân cụm **K-Means** ($K = 8$) dự phòng cho trạng thái *cold-start*.
3. **Chính sách cảnh báo & Kiểm chứng tích hợp:**
   * Đề xuất hàng đợi điều tra `hybrid_evidence` kết hợp cảnh báo neo điểm rủi ro cao ($\text{Risk} \ge 75$) và bộ lọc bằng chứng liên nguồn (truy cập URL giám sát/tìm việc, dịch vụ chia sẻ dữ liệu, thao tác tệp khi cắm USB, gửi email ra ngoài tổ chức sau truy cập tệp).
   * Kiểm chứng khả năng thu gom và truy xuất dữ liệu theo lược đồ ECS qua luồng **Filebeat – Elasticsearch – UEBA**.

---

## Kết quả thực nghiệm tiêu biểu

Đánh giá trên tập kiểm thử độc lập theo thời gian ($151$ ngày, từ `18/12/2010` đến `17/05/2011` với $268$ ca dương đã biết trong CERT Answer Key):

| Chính sách xuất cảnh báo | Số cảnh báo phát ra | Số ca dương truy hồi | Known-Positive Yield | Known-Positive Recall |
| :--- | :---: | :---: | :---: | :---: |
| **Fixed Top-10 mỗi ngày** | $1.510$ | $71 / 268$ | **4,702%** | **26,4925%** |
| **Hàng đợi `hybrid_evidence`** | $724$ | $58 / 268$ | **8,0110%** | **21,6418%** |

* Chỉ số xếp hạng có điều kiện đạt **$\text{ROC-AUC} = 0,9017$**.
* Thí nghiệm kiểm chứng tích hợp thu gom và chuẩn hóa thành công **$5.600 / 5.600$** sự kiện phát lại qua Filebeat và Elasticsearch API.

---

## Cấu trúc thư mục tài liệu

```text
.
├── README.md                                  # Tóm tắt đề tài, phương pháp và kết quả chính
├── report/
│   └── 07-SV-2026-ATTT-Bao-cao-ket-qua.pdf    # Toàn văn báo cáo kết quả đề tài khoa học sinh viên
└── slides/                                    # Slide thuyết trình bảo vệ đề tài
```