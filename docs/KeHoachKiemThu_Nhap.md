# Kế hoạch kiểm thử (bản nháp) – ShipFee

> **Nhóm 7 – COMP1307 Kiểm thử phần mềm cơ bản** · Đề tài 9: Tính phí giao hàng / khuyến mãi (hướng B)
> Phiên bản: 0.1 (nháp, trước Buổi 4) · Cập nhật: 29/10/2026 · Người soạn: Nguyễn Trần Phúc Thiên

## 1. Giới thiệu đối tượng kiểm thử

**ShipFee** là ứng dụng web + REST API do nhóm tự xây dựng, tính phí giao hàng cho một đơn hàng dựa trên khối lượng, khoảng cách, hạng thành viên và mã giảm giá; đồng thời quản lý trạng thái đơn giao hàng.

- Công nghệ: Python 3.12, FastAPI, SQLite; giao diện web đơn giản gọi API.
- Mã nguồn: thư mục `app/` của kho này.

### 1.1. Quy tắc nghiệp vụ sơ bộ (sẽ hoàn thiện thành đặc tả R01–R0x ở Buổi 4–6)

| Mã | Quy tắc |
|---|---|
| R01 | **Phí theo khối lượng** (0 < w ≤ 30 kg, bước 0,1 kg): w ≤ 1 kg: 15.000 đ; 1 < w ≤ 3: 20.000 đ; 3 < w ≤ 5: 25.000 đ; 5 < w ≤ 30: 25.000 đ + 5.000 đ cho mỗi kg vượt 5 kg (phần lẻ tính tròn lên 1 kg). Ngoài khoảng → báo "Khối lượng không hợp lệ". |
| R02 | **Phụ phí khoảng cách** (0 < d ≤ 100 km): d ≤ 5 km: 0 đ; 5 < d ≤ 20: 2.000 đ/km vượt 5 km; 20 < d ≤ 100: 30.000 đ + 3.000 đ/km vượt 20 km (km lẻ tính tròn lên). Ngoài khoảng → "Ngoài phạm vi giao hàng". |
| R03 | **Giảm theo hạng thành viên** trên phí gốc: Đồng 0%, Bạc 5%, Vàng 10%, Kim cương 15%. |
| R04 | **Mã giảm giá**: 6–10 ký tự chữ in hoa và chữ số; mỗi đơn dùng tối đa 1 mã; mã phải còn hạn, còn lượt, giá trị hàng ≥ mức tối thiểu của mã. Loại mã: giảm số tiền cố định (ví dụ FREESHIP giảm tối đa 30.000 đ) hoặc giảm % có trần. |
| R05 | **Kết hợp giảm**: áp giảm theo hạng trước, rồi mới áp mã; tổng giảm không vượt 50% phí gốc; phí cuối làm tròn đến 1.000 đ và không âm. |
| R06 | **Trạng thái đơn**: Mới → Đã xác nhận → Đang giao → Đã giao; được hủy khi ở Mới hoặc Đã xác nhận; hủy thì hoàn lại 1 lượt mã đã dùng. |

### 1.2. Câu hỏi làm rõ (kiểm thử tĩnh)

1. Làm tròn phí cuối lên hay xuống 1.000 đ? (đang giả định: làm tròn đến 1.000 đ gần nhất)
2. Khoảng cách đúng bằng 5,0 km có tính phụ phí không? (giả định: không)
3. Mã hết hạn lúc 23:59:59 hay 00:00:00 ngày ghi trên mã?
4. Mã giảm giá có phân biệt chữ hoa/thường và có tự cắt khoảng trắng không?
5. Hủy đơn ở trạng thái "Đang giao" có được không nếu khách chưa nhận hàng?

## 2. Phạm vi

| Trong phạm vi | Ngoài phạm vi (lý do) |
|---|---|
| Tính phí theo khối lượng, khoảng cách (R01, R02) | Thanh toán trực tuyến – ứng dụng không có chức năng này |
| Giảm theo hạng thành viên (R03) | Kiểm thử hiệu năng, tải – ứng dụng học tập, không có yêu cầu hiệu năng |
| Áp dụng mã giảm giá, kết hợp giảm (R04, R05) | Ứng dụng di động, nhiều trình duyệt – nhóm chỉ kiểm thử API và web trên Chrome |
| Vòng đời đơn giao hàng (R06) | Tích hợp bản đồ thật để đo khoảng cách – khoảng cách nhập tay |
| Kiểm thử đơn vị các hàm tính phí; kiểm thử API tự động | Bảo mật nâng cao (quét lỗ hổng) |

## 3. Mục tiêu (đo được)

- 100% yêu cầu R01–R06 có ≥ 1 ca kiểm thử trong ma trận truy vết.
- ≥ 60 ca kiểm thử mức thấp, áp dụng EP + BVA cho ≥ 2 trường, ≥ 1 bảng quyết định và ≥ 1 sơ đồ chuyển trạng thái.
- 2 hàm hộp trắng đạt ≥ 80% bao phủ nhánh (coverage.py).
- ≥ 15 kiểm thử đơn vị và ≥ 8 kịch bản API tự động chạy trong CI.
- ≥ 5 báo cáo lỗi có minh chứng trên GitHub Issues.

## 4. Cách tiếp cận

| Mức kiểm thử | Nội dung | Kỹ thuật | Công cụ |
|---|---|---|---|
| Đơn vị | `tinh_phi_co_ban`, `ap_dung_giam_gia`, kiểm tra mã | EP, BVA (tham số hóa), hộp trắng (CFG, V(G), đường cơ sở, MC/DC) | pytest, coverage.py |
| Tích hợp / API | Endpoint tính phí, áp mã, cập nhật trạng thái đơn | EP, BVA, bảng quyết định, chuyển trạng thái | Postman, Newman |
| Hệ thống | Luồng tạo đơn → tính phí → áp mã → giao/hủy | Kiểm thử thăm dò có điều lệ (≥ 2 phiên), đoán lỗi | Trình duyệt Chrome |
| Hồi quy | Chạy lại bộ unit + API mỗi lần push/PR | Tự động | GitHub Actions |

Ưu tiên kiểm thử theo rủi ro (mục 7): R01, R02, R05 kiểm thử kỹ nhất.

## 5. Tiêu chí đầu vào / đầu ra (mức kiểm thử hệ thống)

**Đầu vào**
1. Đặc tả R01–R06 đã được cả nhóm review, câu hỏi làm rõ đã có câu trả lời.
2. Bản build chạy được trên máy kiểm thử, smoke test 5 ca cơ bản đạt 100%.
3. Kiểm thử đơn vị hiện có đạt 100% và CI chạy xanh.
4. Dữ liệu thử sẵn sàng: ≥ 4 tài khoản (đủ 4 hạng), ≥ 5 mã giảm giá (còn hạn, hết hạn, hết lượt, chưa đủ giá trị tối thiểu).

**Đầu ra**
1. 100% ca kiểm thử ưu tiên cao đã chạy; ≥ 95% tổng số ca đã chạy.
2. Tỷ lệ Pass ≥ 90%.
3. Không còn lỗi Critical/High đang mở; lỗi Medium có ghi kế hoạch xử lý.
4. Bao phủ nhánh ≥ 80% cho 2 hàm hộp trắng.
5. Báo cáo hoàn tất kiểm thử đã được cả nhóm thống nhất.

## 6. Môi trường

Windows 10/11, Python 3.12, Chrome bản mới nhất, Postman, SQLite; CI trên GitHub Actions (ubuntu-latest).

## 7. Rủi ro (Khả năng × Ảnh hưởng, thang 1–5)

| Mã | Rủi ro | Loại | K | A | Mức | Hướng xử lý |
|---|---|---|---|---|---|---|
| RR1 | Tính sai phí tại các mốc bậc khối lượng (1 / 3 / 5 kg) | Sản phẩm | 4 | 4 | **16 – Cao** | BVA 3 giá trị quanh mỗi mốc; kiểm thử đơn vị tham số hóa |
| RR2 | Thứ tự áp dụng giảm theo hạng và mã chưa rõ → phí sai | Sản phẩm | 4 | 4 | **16 – Cao** | Chốt trong câu hỏi làm rõ; bảng quyết định cho tổ hợp giảm |
| RR3 | Phí cuối âm hoặc tổng giảm vượt trần 50% | Sản phẩm | 3 | 5 | **15 – Cao** | Ca biên giảm = phí, giảm > phí; MC/DC cho điều kiện trần |
| RR4 | API không kiểm tra dữ liệu sai định dạng (số âm, chữ, "2,5") | Sản phẩm | 4 | 3 | 12 – TB | EP lớp không hợp lệ, mỗi ca một lớp sai |
| RR5 | Thành viên chưa quen FastAPI/pytest → trễ tiến độ | Dự án | 3 | 3 | 9 – TB | Duy dựng khung ứng dụng + 1 test mẫu trước Buổi 5 |
| RR6 | Lịch thi các môn khác trùng giai đoạn Buổi 9–10 | Dự án | 3 | 3 | 9 – TB | Hoàn thành ≥ 50% ca kiểm thử trước Buổi 7; dự phòng 1 tuần |

## 8. Ước lượng (ba điểm, giờ công)

| Công việc | a | m | b | E = (a+4m+b)/6 | SD = (b−a)/6 |
|---|---|---|---|---|---|
| Xây dựng ứng dụng ShipFee | 8 | 12 | 20 | 12,7 | 2,0 |
| Thiết kế ≥ 60 ca kiểm thử + RTM | 10 | 14 | 22 | 14,7 | 2,0 |
| Tự động hóa (unit + API + CI) | 10 | 15 | 26 | 16,0 | 2,7 |
| **Tổng** | | | | **≈ 43** | ≈ 3,9 |

## 9. Phân công (dự kiến)

| Thành viên | MSSV | Vai trò, công việc chính |
|---|---|---|
| Nguyễn Trần Phúc Thiên | 50.01.104.151 | Trưởng nhóm; kế hoạch, đặc tả, thiết kế ca hộp đen (EP, BVA, bảng quyết định), báo cáo hoàn tất, tổng hợp tiểu luận |
| Nguyễn Hữu Duy | 50.01.104.035 | Xây dựng ứng dụng; kiểm thử đơn vị (pytest); hộp trắng 2 hàm; CI GitHub Actions |
| Phạm Duy Kha | 50.01.104.068 | Kiểm thử API (Postman/Newman); chuyển trạng thái đơn; thực thi, báo cáo lỗi trên GitHub Issues; video demo |

## 10. Lịch (dự kiến – giả định học thứ Sáu hằng tuần, Buổi 3 ngày 30/10/2026)

| Mốc | Thời gian | Việc cần xong |
|---|---|---|
| Buổi 4 | 06/11 | Nộp phiếu đăng ký đề tài; kế hoạch nháp trên GitHub |
| Buổi 5–6 | 13/11 – 20/11 | Khung ứng dụng chạy được; đặc tả R01–R06; ca kiểm thử vòng 1 |
| Buổi 7 | 27/11 | **Mốc tiến độ**: kế hoạch, đặc tả, ≥ 30 ca kiểm thử (≥ 50%), RTM sơ bộ (nộp Excel + PDF) |
| Buổi 8 | 04/12 | Hộp trắng 2 hàm; hoàn thiện kỹ thuật hộp đen |
| Buổi 9 | 11/12 | ≥ 15 kiểm thử đơn vị trong CI |
| Buổi 10 | 18/12 | ≥ 8 kịch bản API tự động; hồi quy; báo cáo hoàn tất; slide; video |
| Hạn nộp | 25/12, 23:59 | Nộp tệp `COMP1307_Nhom7_ShipFee.zip` |
