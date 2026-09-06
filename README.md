# 🚴 Quy Trình Tích Hợp Doanh Nghiệp SAP S/4HANA - Case Study: Global Bike Inc. (GBI)

Đồ án phân tích và thực thi chu trình kinh doanh tích hợp liên phân hệ (**Procure-to-Pay ➔ Plan-to-Produce ➔ Order-to-Cash**) trên nền tảng hệ thống hoạch định nguồn lực doanh nghiệp **SAP S/4HANA**.

---

## 📌 Tổng Quan Dự Án (Project Overview)
- **Tác giả / Người thực hiện:** Hà Anh Đức
- **Mã sinh viên:** 2321003977
- **Mã định danh thực hành:** `LEARN-876` (Hậu tố dữ liệu nền: `...876`)
- **Vai trò:** Business Process Analyst / ERP Consultant
- **Mục tiêu:** Mô phỏng chu trình kinh doanh khép kín của một doanh nghiệp sản xuất và thương mại xe đạp (GBI), làm sáng tỏ sự luân chuyển dữ liệu và tính toàn vẹn (ACID) giữa các phân hệ cốt lõi.

---

## 🏢 Cấu Trúc Tổ Chức & Dữ Liệu Nền (Enterprise Structure & Master Data)
- **Cấu trúc doanh nghiệp:**
  - Company Code: `US00` (Global Bike US)
  - Plant: `DL00` (Dallas Plant)
  - Sales Organization: `UW00` (US West) | Distribution Channel: `WH` (Wholesale) | Division: `BI` (Bicycles)
  - Storage Locations: `RM00` (Nguyên liệu thô) | `SF00` (Bán thành phẩm) | `FG00` (Thành phẩm)
- **Dữ liệu nền tự khởi tạo (Master Data):**
  - **17 Mã Vật tư (Materials):** 1 Thành phẩm (`DXTR876`), 1 Bán thành phẩm (`TRWA876`), 15 Nguyên vật liệu thô (`TRFR876`, `DGAM876`,...).
  - **Đối tác kinh doanh (Business Partners):** Nhà cung cấp `Global Materials Supply 876` (Mã số `1012784`) và Khách hàng `Summer Bike Shop 876` (Mã số `1012785`).
  - **Dữ liệu sản xuất:** Định mức vật tư (BOM) 2 cấp, Quy trình sản xuất (Routing) 11 công đoạn và Phiên bản sản xuất (Production Version `0001`).

---

## 🔄 Chu Trình Kinh Doanh Tích Hợp 24 Bước Nghiệp Vụ

### 1. Quy trình Mua hàng (Procure-to-Pay - Phân hệ MM & FI)
- **Lập kế hoạch & Mua sắm:** Khởi tạo Planned Order số `39572` cho 100 xe đạp ➔ Kiểm tra tồn kho (Stock Overview) xác nhận lượng thiếu hụt ➔ Lập Yêu cầu mua hàng (Purchase Requisition) số `0010029277` cho 15 loại nguyên vật liệu.
- **Đơn hàng & Nhận hàng:** Tạo Đơn đặt hàng (PO) số `4500004617` trị giá **$95,900.00 USD** ➔ Nhập kho (Goods Receipt) qua giao dịch MIGO, sinh Chứng từ vật tư số `5000005829` ghi tăng kho `RM00`.
- **Thanh toán:** Đối chiếu hóa đơn 3 chiều (Three-way match) qua MIRO số `5105604570` ➔ Thanh toán cho nhà cung cấp qua F-53, ghi nhận Chứng từ thanh toán số `1500002677`.

### 2. Quy trình Sản xuất (Plan-to-Produce - Phân hệ PP, MM & CO)
- **Giai đoạn 1 (Bán thành phẩm):** Phát hành Lệnh sản xuất số `1001346` cho 200 Cụm bánh xe `TRWA876` ➔ Xuất kho NVL (Chứng từ `4900045179`) ➔ Xác nhận hoàn thành (Confirmation) ➔ Nhập kho `SF00` (Chứng từ `5000005857`).
- **Giai đoạn 2 (Thành phẩm):** Chuyển đổi Planned Order thành Lệnh sản xuất số `1001348` cho 100 xe `DXTR876` ➔ Xuất kho linh kiện & BTP (Chứng từ `4900045181`) ➔ Xác nhận hoàn thành ➔ Nhập kho Thành phẩm `FG00` (Chứng từ `5000005862`).

### 3. Quy trình Bán hàng (Order-to-Cash - Phân hệ SD & FI)
- **Tiền bán hàng:** Tiếp nhận Yêu cầu hỏi hàng (Inquiry) số `10003810` (15 xe `DXTR876`) ➔ Lập Báo giá (Quotation) số `20003155` áp dụng chiết khấu vật tư $125/chiếc (K004) và chiết khấu 5% mùa hè (RA00).
- **Đơn hàng & Giao hàng:** Tạo Đơn bán hàng (Sales Order) số `4670` trị giá **$43,818.75 USD** ➔ Lập Lệnh giao hàng (Outbound Delivery) số `80004273` ➔ Lấy hàng & Đóng gói (Picking `15 EA`) ➔ Xuất kho giao hàng (Post Goods Issue), sinh Chứng từ vật tư số `4900045221`.
- **Thanh toán & Thu tiền:** Tạo Hóa đơn (Billing Document) số `90003234` ➔ Ghi nhận thanh toán từ khách hàng (Incoming Payment) số `1400003578` (Nợ TK 100000 / Có TK 1012785), tất toán toàn bộ công nợ phải thu.

---

## 📊 Phân Tích & Đối Soát Chứng Từ Tích Hợp (Audit & Integration Trail)
- **SD Document Flow:** Kiểm tra chuỗi trạng thái khép kín từ Inquiry ➔ Quotation ➔ Sales Order ➔ Delivery ➔ Invoice ➔ Journal Entry (Trạng thái `Completed` & `Cleared`).
- **PO History:** Xác minh sự đồng bộ giữa khâu Kho vận (Goods Receipt - WE) và khâu Kế toán phải trả (Invoice Receipt - RE-L).
- **Production Cost Analysis:** Báo cáo chi phí thực tế Lệnh sản xuất số `1001348` đạt **$73,205.00 USD** (bao gồm $38,895.22 chi phí NVL, $22,600.00 chi phí BTP và $11,000.22 chi phí máy móc).

---

## 📁 Tài Liệu Báo Cáo (Project Deliverables)
Chi tiết từng bước cấu hình, tham số giao dịch và ảnh chụp màn hình minh chứng được lưu trữ tại thư mục:
- [Báo cáo đồ án hoàn chỉnh (PDF)](Reports/8002_2321003977_HaAnhDuc.pdf)
- [Bản thảo chi tiết (DOCX)](Reports/8002_2321003977_HaAnhDuc.docx)
