# ShipFee – Kiểm thử hệ thống tính phí giao hàng và khuyến mãi

Tiểu luận cuối kỳ **COMP1307 – Kiểm thử phần mềm cơ bản** (A.2.6), Trường ĐH Sư phạm TP.HCM, học kỳ I 2026–2027.
Giảng viên: ThS. Lê Thanh Thoại · Đề tài 9 – hướng B (tự xây dựng rồi kiểm thử).

## Nhóm 7

| Họ tên | MSSV | Vai trò dự kiến |
|---|---|---|
| Nguyễn Hữu Duy | 50.01.104.035 | Trưởng nhóm – kế hoạch, đặc tả, thiết kế ca kiểm thử hộp đen, báo cáo |
| Nguyễn Trần Phúc Thiên | 50.01.104.151 | Xây dựng ứng dụng, kiểm thử đơn vị, hộp trắng, CI |
| Phạm Duy Kha | 50.01.104.068 | Kiểm thử API tự động, thực thi, quản lý lỗi |

## Ứng dụng

ShipFee tính phí giao hàng theo khối lượng, khoảng cách, hạng thành viên và mã giảm giá; quản lý trạng thái đơn giao hàng.
Chi tiết phạm vi, quy tắc nghiệp vụ và kế hoạch: [`docs/KeHoachKiemThu_Nhap.md`](docs/KeHoachKiemThu_Nhap.md).

## Cấu trúc kho

```
app/                 mã ứng dụng (Python 3.12, FastAPI)
tests/unit/          kiểm thử đơn vị (pytest)
tests/api/           collection Postman / Newman
tests/data/          dữ liệu kiểm thử CSV
docs/                kế hoạch, tệp Excel, báo cáo
.github/workflows/   CI chạy kiểm thử mỗi push / pull request
```

## Cài đặt và chạy kiểm thử

> Sẽ cập nhật khi có mã ứng dụng (dự kiến Buổi 5).

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows  (Linux/macOS: source .venv/bin/activate)
pip install -r requirements.txt
uvicorn app.main:app --reload   # chạy ứng dụng tại http://127.0.0.1:8000
pytest --cov=app --cov-branch   # chạy kiểm thử đơn vị + đo bao phủ nhánh
```

## Quy ước

- Báo lỗi trên **Issues** với nhãn `bug`, `severity:high` / `severity:low`, `priority:*`; commit sửa lỗi ghi `fixes #<số>`.
- Theo dõi công việc trên **Projects** (To do / In progress / Review / Done).
- Không đưa mật khẩu, khóa API, dữ liệu cá nhân lên kho.
