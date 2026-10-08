# Báo cáo lab: chọn tracker cho 5 video

Detector cố định: `yolo26n.pt`, ảnh 640 px, lớp người, Re-ID `osnet_x0_25_msmt17.pt`. Không đổi các mục này trong bài nộp chính.

Các video `video_2` đến `video_5` không có nhãn; nhận xét dưới đây dựa trên video mẫu và thống kê kết quả, không phải điểm đánh giá chính xác.

## 1. Cấu hình đã chọn

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---:|---:|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.3 | 0.7 | Người nhìn rõ giữ ID tương đối ổn định; nhiều người ở xa không được phát hiện. | bytetrack 0.3/0.5 (HOTA 26.9); botsort 0.5/0.5 (HOTA 27.2, bỏ sót nhiều hơn) |
| video_2 (phố đêm, tĩnh, rất đông) | botsort | 0.15 | 0.5 | Người ở gần được bám tốt; hạ `conf` giúp bắt thêm người nhỏ ở xa. | ocsort 0.3/0.5 (77 ID, 34% track ngắn); botsort 0.3/0.7 (72 ID) |
| video_3 (camera di động, ảnh nhỏ) | ocsort | 0.3 | 0.5 | Người che khuất nhau, hình mờ và ít khung hình/giây; các tracker đều đổi ID nhiều. | botsort 0.3/0.5 (146–161 ID); strongsort 0.3/0.5 (160 ID); deepocsort 0.3/0.5 (169 ID) |
| video_4 (trong nhà, camera di chuyển) | botsort | 0.15 | 0.5 | Người ở gần có hộp lớn và ID giữ khá lâu; chưa thấy hộp giả rõ trên ảnh phản chiếu ở frame mẫu. | ocsort 0.3/0.5 (79 ID, 38% track ngắn); botsort 0.5/0.5 (ít hộp hơn mỗi frame) |
| video_5 (trên xe bus, giao lộ đông) | botsort | 0.15 | 0.5 | Người nhỏ, ở xa; camera rung và mỗi frame có ít hộp nên nhiều track ngắn. | ocsort 0.3/0.5 (99 ID, 51% track ngắn); botsort 0.3/0.5 (82 ID) |

## 2. Số liệu video_1

Kết quả do `scripts/evaluate_practice.py` in ra:

| Chỉ số | Kết quả |
|---|---:|
| HOTA / DetA / AssA | 29.969 / 18.408 / 49.061 |
| MOTA / MOTP / IDSW | 19.025 / 80.817 / 33 |
| IDF1 / IDR / IDP | 29.703 / 18.680 / 72.463 |
| DetRe / DetPr | 19.176 / 74.385 |
| Detections / GT detections | 4,790 / 18,581 |
| IDs / GT IDs | 55 / 62 |

So sánh các cấu hình đã thử trên `video_1`:

| Tracker | conf | iou | HOTA | MOTA | IDF1 |
|---|---:|---:|---:|---:|---:|
| bytetrack | 0.3 | 0.5 | 26.9 | 17.3 | 25.7 |
| ocsort | 0.3 | 0.5 | 27.5 | 19.8 | 28.7 |
| botsort | 0.3 | 0.5 | 29.5 | 19.8 | 29.4 |
| strongsort | 0.3 | 0.5 | 28.7 | 19.7 | 29.9 |
| deepocsort | 0.3 | 0.5 | 27.4 | 19.8 | 27.8 |
| botsort | 0.5 | 0.5 | 27.2 | 15.3 | 24.6 |
| **botsort (nộp)** | **0.3** | **0.7** | **30.0** | **19.0** | **29.7** |

`video_2` đến `video_5` không có nhãn nên không báo cáo điểm đánh giá cho các video này. Trên `video_1`, cấu hình nộp có HOTA cao hơn 0.5 điểm so với botsort 0.3/0.5.

## 3. Phân tích

Ở `video_1`, botsort đạt HOTA 30.0, cao hơn bytetrack (26.9). AssA 49.1 cho thấy tracker giữ danh tính tương đối tốt với các đối tượng đã phát hiện; ngược lại, DetA chỉ 18.4 và DetRe 19.2%, nên bỏ sót người là hạn chế chính.

Ở `video_2`, hạ `conf` từ 0.3 xuống 0.15 làm số hộp trung bình mỗi frame tăng từ 11.7 lên 13.2 và tỷ lệ track ngắn giảm từ 24% xuống 6%. Điều này phù hợp với cảnh đông, tối, nơi người ở xa dễ bị bỏ sót; tuy nhiên, chưa có nhãn để xác nhận mức cải thiện.

Ở `video_3`, cảnh mờ, độ phân giải thấp và người che khuất nhau khiến cả botsort, strongsort và deepocsort đều tạo nhiều ID (146–169 ở cấu hình được ghi nhận). Vì vậy chọn ocsort, nhưng kết quả vẫn còn nhiều track ngắn (49%); với detector cố định, chưa tracker nào xử lý tốt cảnh này.

## 4. Nếu có thêm thời gian

Tôi sẽ quét `conf` mịn hơn trong khoảng 0.1–0.3 cho `video_2` và `video_5`, đồng thời kiểm tra các frame tạo ID mới ở `video_3` để phân biệt lỗi phát hiện với lỗi gán ID.
