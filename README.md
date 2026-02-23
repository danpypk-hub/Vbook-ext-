## LẬP TRÌNH VBOOK EXTENSION (TỪ ZERO ĐẾN HERO)

Tài liệu hướng dẫn chi tiết quy trình phát triển tiện ích (Extension) cho ứng dụng Vbook. Kiến thức được hệ thống hóa từ khâu thiết lập môi trường, bóc tách dữ liệu (Scraping), đến xuất bản mã nguồn lên GitHub. Case study thực hành thực tế: `truyenmoiyy.com`.

##*Tham khảo thêm mã nguồn và hướng dẫn viết các Vbook Extension hoàn thiện tại:* [https://github.com/dat-bi/ext-vbook](https://github.com/dat-bi/ext-vbook)

---

## PHẦN 1: CÔNG CỤ VÀ MÔI TRƯỜNG PHÁT TRIỂN (BUILD & DEBUG)

Extension Vbook đóng vai trò là "cầu nối", tải mã HTML của trang web, trích xuất dữ liệu và định dạng lại để hiển thị trên ứng dụng. Cần chuẩn bị môi trường phát triển như sau:

### 1. Công cụ lập trình
* **Trực tiếp trên thiết bị di động:** Tải ứng dụng `Vbook Ext Maker`. Hỗ trợ khởi tạo khung tệp tin, viết mã, chạy thử (Run/Preview) trực tiếp với URL thực tế và xuất tệp `.zip`.
* **Trên Máy tính (Khuyến nghị):** Sử dụng phần mềm soạn thảo mã nguồn **Visual Studio Code (VSCode)** kết hợp trình duyệt Google Chrome.

### 2. Thiết lập kết nối & Gỡ lỗi (Remote Debug)
Phương pháp này giúp mã nguồn đồng bộ từ VSCode (máy tính) lên Vbook (điện thoại) ngay lập tức mà không cần nén tệp `.zip` nhiều lần.

1.  **Lấy IP:** Máy tính và điện thoại truy cập chung mạng Wifi. Mở Vbook trên điện thoại -> *Cài đặt* -> Kích hoạt chế độ *Gỡ lỗi (Debug)*. Hệ thống cung cấp một địa chỉ IP nội bộ (VD: `192.168.1.15:8080`).
2.  **Kết nối:** Trên máy tính, mở Chrome, nhập địa chỉ IP vào thanh URL. Giao diện Vbook Extension Server xuất hiện.
3.  **Kiểm thử (Test):** Viết mã trên VSCode, lưu lại. Kéo thả tệp `plugin.json` (hoặc cả thư mục dự án) vào giao diện web. App Vbook sẽ tự động nạp mã nguồn.
4.  **Gỡ lỗi (Debug):** Nhấn **F12** trên Chrome (máy tính), chuyển sang tab `Console`. Bất kỳ lỗi logic hoặc cú pháp nào khi thao tác trên điện thoại đều được báo đỏ chi tiết tại đây.
5.  **Đóng gói:** Khi hoàn thiện, nén thư mục thành tệp `.zip` (Lưu ý: `plugin.json` và `icon.png` phải nằm ở lớp ngoài cùng của tệp nén).

---

## PHẦN 2: KIẾN TRÚC THƯ MỤC CỦA MỘT EXTENSION

Một extension đạt chuẩn bao gồm cấu trúc tệp tin sau:

```text
Ten_Extension.zip/
├── plugin.json       (Hồ sơ lõi: Định danh tên, tác giả, cấu hình URL)
├── icon.png          (Ảnh đại diện tiện ích, tỷ lệ vuông 1:1)
└── src/              (Thư mục chứa mã nguồn thực thi logic)
    ├── home.js       (Màn hình chính: Chứa Tab mặc định VD: Truyện mới đăng)
    ├── genre.js      (Danh sách Thể loại trong nút Khám phá thêm)
    ├── gen.js        (Quét danh sách hàng loạt truyện từ một chuyên mục)
    ├── detail.js     (Quét thông tin chi tiết 1 bộ truyện: Tên, Tác giả...)
    ├── page.js       (Hỗ trợ tìm số trang của mục lục nếu mục lục phân trang)
    ├── toc.js        (Quét Mục lục: Trích xuất mảng liên kết các chương)
    └── chap.js       (Quét nội dung chữ của chương và cắt bỏ quảng cáo)

            
