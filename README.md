# Nhom20 — Bài thực hành dữ liệu đặt phòng khách sạn

## 1. Thành viên và phân công

| Họ tên | MSSV | Phụ trách chính | Có thể giải thích thêm |
|---|---|---|---|
| [Họ và tên Thành viên 1] | [MSSV 1] | Giai đoạn A (Khảo sát dữ liệu), Làm sạch B1 – B5, Phần 2 báo cáo | Quy trình gộp `booking_id` (B8), các kết quả phân tích & biểu đồ (Câu 2 – 5) |
| [Họ và tên Thành viên 2] | [MSSV 2] | Làm sạch B6 – B8, Giai đoạn C (`clean_df`, `monthly_summary`, nhật ký, truy vết, kiểm tra), Câu 1, README | Quy tắc chuẩn hóa nhãn & ngày tháng (B1 – B5), phân tích giá phòng & lưu trú (Câu 6 – 10) |
| [Họ và tên Thành viên 3] | [MSSV 3] | Giai đoạn D (Câu 2 – 10, 8 biểu đồ), tổng hợp 6 Insight có bằng chứng, Phần 3 & 4 báo cáo | Khảo sát chất lượng dữ liệu ban đầu (Giai đoạn A), quy tắc làm sạch chính (B1 – B8) |

## 2. Cách chạy lại

1. Giải nén `NhomXX_KhachSan.zip`, giữ nguyên cấu trúc thư mục.
2. Mở `hotel_analysis.ipynb` tại thư mục gốc; chọn **Restart & Run all**.
3. Kết quả được ghi vào `outputs/` (5 tệp dữ liệu, `tables/`, `figures/`).
4. (Tùy chọn) Chạy công cụ tự kiểm tra tại thư mục gốc:
   ```bash
   python tools/kiem_tra_nop_bai.py .
   ```

Thời gian chạy toàn bộ trên máy nhóm: khoảng **6 – 10** giây.

## 3. Môi trường và thư viện

| Thành phần | Phiên bản |
|---|---|
| Python | 3.14.2 |
| pandas | 3.0.5 |
| matplotlib | 3.11.1 |
| seaborn (nếu dùng) | Không sử dụng (chỉ dùng `matplotlib`) |
| Môi trường | VS Code / Jupyter Notebook (tương thích Google Colab) |

## 4. Nguồn dữ liệu

Antonio, N., de Almeida, A., & Nunes, L. (2019). Hotel booking demand datasets. *Data in Brief*, 22, 41–49. https://doi.org/10.1016/j.dib.2018.11.126 (CC BY 4.0).
Bản biên soạn cho học tập `hotel_bookings_dirty.csv` (10.325 dòng × 24 cột), khảo sát hai khách sạn tại Bồ Đào Nha (`City Hotel` và `Resort Hotel`) trong giai đoạn đến nhận phòng từ `01/07/2015` đến `31/08/2017` (mốc chốt trạng thái `SNAPSHOT_DATE = 2017-09-14`).

## 5. Quy tắc riêng của nhóm (khác hoặc bổ sung so với đề)

| # | Tình huống | Quy tắc nhóm chọn | Lý do | Ảnh hưởng đến kết quả |
|---|---|---|---|---|
| 1 | Xung đột giá trị khi gộp cùng `booking_id` (B8) | Sắp xếp bảng theo `row_id` tăng dần trước khi gọi `.groupby('booking_id', sort=False).first()`. Ưu tiên giữ giá trị không thiếu (`non-null`) đầu tiên ở mỗi cột; nếu hai dòng có giá trị khác nhau (mâu thuẫn), lấy giá trị của dòng có `row_id` nhỏ nhất, sau đó đồng bộ lại `is_canceled` theo `reservation_status`. | Tận dụng tối đa thông tin từ 64 bản ghi bổ sung (`merged`) để bù cho các ô bị khuyết, đồng thời đảm bảo quy tắc phân xử xung đột nhất quán, tái lập được 100%. | Thu gọn 10.325 dòng gốc về đúng 10.000 đơn đặt phòng duy nhất (`10.000 kept`, `261 dropped`, `64 merged` trong `row_lineage.csv`), không làm mất thông tin hợp lệ. |
| 2 | Thời điểm loại bỏ các bản ghi trùng tuyệt đối 23 cột nghiệp vụ (B1 & B8) | Ở B1 chỉ chuẩn hóa `booking_id` (`strip().upper()`) và đếm số dòng trùng tuyệt đối (181 dòng), chưa xóa ngay (`removed_rows = 0`) mà giữ lại để làm sạch đồng bộ qua B2–B7 rồi xử lý gộp/loại một lần ở B8. | Tránh mất dấu `row_id` giữa các bước và giúp nhận diện chính xác hơn các bản ghi trở nên trùng khớp hoàn toàn (261 dòng `dropped`) sau khi đã chuẩn hóa nhãn, ngày và số. | Bảng `row_lineage.csv` phủ kín 100% (`10.325/10.325`) `row_id` gốc với trạng thái `kept` / `merged` / `dropped` rõ ràng. |
| 3 | Xử lý nhãn `Undefined` khác nhau giữa các cột (B3 & B4) | Chuyển `Undefined` ở `market_segment` và `distribution_channel` về `pd.NA` (bước B3), nhưng giữ lại `Undefined` ở cột `meal` để ánh xạ thành `SC` (*Self Catering* — không kèm bữa ăn) ở bước B4. | Tuân thủ đúng quy ước của bộ dữ liệu nguồn Antonio et al. (2019) và Mục 5.2 của đề bài (`Undefined` và `SC` ở cột `meal` cùng mang nghĩa không đặt gói ăn). | Bảo toàn cỡ mẫu hợp lệ của cột `meal` và tránh tạo nhãn rác khi phân tích theo `market_segment` và `distribution_channel`. |
| 4 | Giải quyết mâu thuẫn giữa `is_canceled` và `reservation_status` (B2 & B4) | Ở B2, khi chuyển 26 giá trị ngày nằm nhầm từ `reservation_status` sang `reservation_status_date`, đặt `reservation_status = pd.NA` (không đoán từ `is_canceled`). Ở B4, khi `reservation_status` hợp lệ nhưng mâu thuẫn với `is_canceled` (41 dòng), cập nhật lại `is_canceled` theo `reservation_status` (`Check-Out` → `0`; `Canceled`, `No-Show` → `1`). | Khi `is_canceled = 1` không thể đoán trạng thái là `Canceled` hay `No-Show`; ngược lại khi đã có trạng thái kết thúc hợp lệ thì đó là bằng chứng nghiệp vụ cuối cùng. | Đảm bảo 0 dòng mâu thuẫn trạng thái hủy trên toàn bộ 10.000 đặt phòng mà không vi phạm nguyên tắc "không đoán dữ liệu". |
| 5 | Xử lý giá trị âm, vượt miền và mã quốc gia `CN` (B6 & B7) | Khôi phục các lỗi hình thức (`"3 nights"` → `3`, `"€95,50"` → `95.50`), giữ nguyên `adr = 0` và mã `CN` (2 chữ cái); chuyển toàn bộ số âm hoặc vượt miền cho phép về `pd.NA`/`NaN` (không lấy trị tuyệt đối, không điền trung bình). | Bảo toàn phân phối thực tế của dữ liệu và tuân thủ nguyên tắc không tự suy diễn hoặc gộp `CN` vào `CHN` khi chưa có bằng chứng từ hệ thống nguồn. | Các phép tính thống kê ở Giai đoạn D (Top 10 quốc gia, trung vị/IQR của `adr`, `lead_time`, `revenue_est`) phản ánh trung thực trên mẫu số hợp lệ. |

## 6. Danh sách tệp nộp

- [x] `README.md`
- [x] `hotel_analysis.ipynb`
- [ ] `Bao_cao_Khach_san.pdf` hoặc `.docx`
- [x] `data/hotel_bookings_dirty.csv`
- [x] `outputs/` — 5 tệp dữ liệu (`bookings_clean.csv`, `monthly_summary.csv`, `cleaning_log.csv`, `row_lineage.csv`, `quality_report.csv`)
- [x] `outputs/tables/` — 14 bảng (`a_missing_survey.csv`, `a_issue_catalog.csv`, `q01` → `q10`)
- [x] `outputs/figures/` — 8 hình (`fig01` → `fig08`)
