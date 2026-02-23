 # Gemini soạn

Tài liệu hướng dẫn chi tiết quy trình phát triển tiện ích (Extension) cho ứng dụng Vbook. Kiến thức được hệ thống hóa từ khâu thiết lập môi trường, bộ API lõi, kỹ thuật bóc tách dữ liệu (Scraping), đến xuất bản mã nguồn lên GitHub. Case study thực hành thực tế: `truyenmoiyy.com`.

> *Tham khảo thêm mã nguồn các Vbook Extension hoàn thiện và tutorial gốc tại:* [https://github.com/dat-bi/ext-vbook](https://github.com/dat-bi/ext-vbook)

---

## PHẦN I: TỔNG QUAN KIẾN TRÚC VÀ BỘ TỪ ĐIỂN API LÕI

Một extension Vbook là một gói `.zip` chứa các mã lệnh JavaScript (chuẩn ES5) nhằm chuyển đổi mã HTML của một trang web thành dữ liệu cấu trúc cho ứng dụng.

### 1. Cấu trúc thư mục chuẩn
```text
Ten_Extension.zip/
├── plugin.json       # Metadata và cấu hình lõi (Bắt buộc)
├── icon.png          # Icon đại diện 64x64px, tỷ lệ vuông 1:1 (Bắt buộc)
└── src/              # Thư mục chứa mã nguồn thực thi
    ├── home.js       # Màn hình chính (Chứa Tab mặc định VD: Truyện mới đăng)
    ├── genre.js      # Danh mục mở rộng (Thể loại trong nút Khám phá thêm)
    ├── search.js     # Tìm kiếm truyện
    ├── gen.js        # Quét danh sách truyện từ một chuyên mục
    ├── detail.js     # Chi tiết truyện (Tên, Tác giả, Thể loại, Số chương)
    ├── page.js       # Phân trang mục lục (Tính tổng số trang nếu mục lục quá dài)
    ├── toc.js        # Mục lục (Trích xuất mảng liên kết các chương)
    └── chap.js       # Nội dung chương (Văn bản thuần và bộ lọc rác/quảng cáo)
```

### 2. Bộ API hỗ trợ tích hợp sẵn (Vbook Core)
Vbook cung cấp các hàm nội tại để tối ưu quá trình lấy dữ liệu:
* **Mạng lưới (Network):**
  * `fetch(url, options)`: Gửi request HTTP/HTTPS (GET, POST).
  * `res.ok`: Kiểm tra trạng thái HTTP (true nếu mã 200).
  * `res.html(charset)`: Trả về đối tượng Document (DOM) để truy vấn HTML.
  * `res.json()`: Trả về dữ liệu JSON (Dành cho các trang dùng API).
* **Truy vấn HTML (DOM Parser):**
  * `doc.select(selector)`: Chọn phần tử theo CSS Selector.
  * `element.text()`: Lấy văn bản thuần.
  * `element.html()`: Lấy mã HTML bên trong.
  * `element.attr(name)`: Lấy giá trị thuộc tính (VD: `href`, `src`).
  * `element.remove()`: Xóa phần tử khỏi cây DOM (Thường dùng lọc quảng cáo).
* **Phản hồi (Response):**
  * `Response.success(data, next)`: Trả về dữ liệu thành công, `next` là tham số lật trang tiếp theo.
  * `Response.error(message)`: Trả về thông báo lỗi lên giao diện.

---

## PHẦN II: CÔNG CỤ VÀ MÔI TRƯỜNG PHÁT TRIỂN (BUILD & DEBUG)

Để phát triển tiện ích, bạn có thể lựa chọn một trong hai quy trình làm việc trên **Máy tính (PC)**:

### 1. Sử dụng Vbook Extension Maker (Khuyến nghị cho người mới)
Đây là một phần mềm chuyên dụng trên PC. Ứng dụng này hỗ trợ:
* Tự động tạo khung dự án (Scaffolding).
* Cung cấp môi trường viết mã trực tiếp.
* Chạy thử (Run/Preview) và xem ngay kết quả bóc tách từ URL thực tế.
* Xuất tệp `.zip` hoàn chỉnh chỉ với 1 click.

### 2. Sử dụng VSCode + Gỡ lỗi Từ xa (Remote Debug)
Đây là phương pháp dành cho lập trình viên, giúp đồng bộ mã nguồn trực tiếp từ máy tính lên ứng dụng trên điện thoại theo thời gian thực mà không cần nén tệp `.zip`.
1. **Lấy IP:** Đảm bảo máy tính và điện thoại chung mạng Wifi. Mở Vbook trên điện thoại -> *Cài đặt* -> Bật *Gỡ lỗi (Debug)*. Hệ thống cung cấp IP (VD: `192.168.1.15:8080`).
2. **Kết nối:** Mở trình duyệt Chrome trên máy tính, nhập IP vào thanh URL. Giao diện Vbook Extension Server xuất hiện.
3. **Kiểm thử:** Viết mã trên Visual Studio Code (VSCode), kéo thả tệp `plugin.json` (hoặc cả thư mục dự án) vào giao diện web trên Chrome. App Vbook sẽ nạp mã ngay lập tức.
4. **Gỡ lỗi:** Nhấn **F12** trên Chrome (máy tính), chuyển sang tab `Console`. Bất kỳ lỗi logic hoặc cú pháp nào khi thao tác trên điện thoại đều được báo đỏ tại đây.

---

## PHẦN III: KỸ THUẬT F12 & GIẢI PHẪU MÃ NGUỒN (TRUYENMOIYY.COM)

Công cụ **F12 (DevTools)** trên Chrome giúp xác định dữ liệu nằm ở thẻ HTML nào.
*Cách dùng:* Mở trang web -> Nhấn `F12` -> Nhấn biểu tượng **Mũi tên chéo** (Select an element) góc trái -> Rê chuột vào phần tử cần lấy.

### 1. Tệp Hồ sơ `plugin.json`
Khai báo cấu hình định danh.

```json
{
  "metadata": {
    "name": "Truyện Mới YY",
    "author": "[Tên_Của_Bạn]",
    "version": 1,
    "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
    "regexp": "truyenmoiyy\\.com",
    "description": "Tiện ích tối ưu tốc độ cho Truyện Mới YY",
    "type": "novel",
    "locale": "vi_VN",
    "language": "javascript"
  },
  "script": {
    "home": "home.js", "genre": "genre.js", "detail": "detail.js",
    "page": "page.js", "toc": "toc.js", "chap": "chap.js", "search": "search.js"
  }
}
```

### 2. Thiết kế Màn hình chính (`home.js`) & Khám phá (`genre.js`)
Để tối ưu giao diện, `home.js` chỉ chứa các tab mặc định (VD: Truyện mới đăng, Truyện Hot). Toàn bộ thể loại chi tiết (Tiên hiệp, Kiếm hiệp...) được chuyển vào `genre.js` để ẩn gọn trong nút menu **Khám phá thêm**.

**`src/home.js` (Tab Mặc định):**
```javascript
function execute() {
    return Response.success([
        { title: "Mới Cập Nhật", input: "/danh-sach/truyen-moi", script: "gen.js" },
        { title: "Truyện Hot", input: "/danh-sach/truyen-hot", script: "gen.js" }
    ]);
}
```

**`src/genre.js` (Trích xuất Thể loại động):**
*Thao tác F12:* Rê chuột vào thanh menu Thể loại trên trang chủ. Bấm F12, thấy danh sách nằm trong `<ul class="dropdown-menu multi-column">`.
```javascript
function execute() {
    var res = fetch("[https://truyenmoiyy.com/](https://truyenmoiyy.com/)");
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];
    
    doc.select(".dropdown-menu.multi-column li a").forEach(function(e) {
        data.push({
            title: e.text().trim(),
            input: e.attr("href"),
            script: "gen.js"
        });
    });
    return Response.success(data);
}
```

### 3. Trích xuất Danh sách truyện (`src/gen.js`)
*Thao tác F12:* Mở `https://truyenmoiyy.com/danh-sach/truyen-moi`. Trỏ vào một cuốn truyện. Thông tin truyện bọc trong `<div class="row" itemscope>`. Sử dụng khối này làm "Container" vòng lặp.

```javascript
function execute(url, page) {
    if (!page) page = '1';
    
    // Web dùng định dạng /trang-1 thay vì ?page=1
    var fetchUrl = url;
    if (fetchUrl.indexOf("http") !== 0) fetchUrl = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + url;
    fetchUrl = fetchUrl + "/trang-" + page;

    var res = fetch(fetchUrl);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];

    // Chọn Container chứa thông tin 1 truyện
    doc.select(".list-truyen .row[itemscope]").forEach(function(e) {
        var a = e.select(".truyen-title a").first(); 
        var img = e.select("img[itemprop=image]").first();
        var author = e.select(".author").first();
        
        if (a && img) {
            var link = a.attr("href");
            if (link.indexOf("http") !== 0) link = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + link;
            
            data.push({
                name: a.text().trim(),
                link: link,
                cover: img.attr("src"),
                description: author ? author.text().trim() : "",
                host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
            });
        }
    });

    // Tính toán số trang kế tiếp
    var hasNext = doc.select(".pagination li a[rel=next]").length > 0;
    if (hasNext) return Response.success(data, (parseInt(page) + 1).toString());
    
    return Response.success(data);
}
```

### 4. Trích xuất Chi tiết truyện (`src/detail.js`)
*Thao tác F12:* Mở URL một bộ truyện.
* Tên truyện: Nằm trong `<h1 class="story-title">`.
* Tác giả: Nằm trong `<a itemprop="name">` thuộc `.author`.
* Số chương: HTML thường phân mảnh, do đó quét toàn văn `doc.text()` và dùng `Regex` để bắt số.

```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    var name = doc.select(".story-title").text().trim() || doc.select("h1").text().trim();
    var cover = doc.select(".book img").first().attr("src");
    if (cover && cover.indexOf("http") !== 0) cover = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + cover;
    
    var authorEl = doc.select("[itemprop=author] [itemprop=name]").first() || doc.select(".author").first();
    var author = authorEl ? authorEl.text().replace(/Tác giả\s*:/i, "").trim() : "Đang cập nhật";
    
    // Regex lấy số chương từ văn bản thuần
    var rawText = doc.text(); 
    var chapters = "Đang cập nhật";
    var chapMatch = rawText.match(/Số [Cc]hương[\s:]*([\d,.]+)/);
    if (chapMatch) chapters = chapMatch[1] + " chương";
    
    var detailInfo = [
        "👤 Tác giả: " + author,
        "📊 Số chương: " + chapters
    ];

    return Response.success({
        name: name,
        cover: cover,
        author: author,
        description: doc.select(".desc-text").text().trim(),
        detail: detailInfo.join("<br>"),
        host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
    });
}
```

### 5. Trích xuất Mục lục (`src/toc.js`)
*Thao tác F12:* Quan sát cây HTML thấy các chương được bọc trong danh sách `<ul class="list-chapter">`, mỗi chương là một thẻ `<li>` chứa thẻ `<a>`. Đi từ thẻ cha `ul` để tránh quét nhầm link rác.

```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var chaps = [];
    
    doc.select(".list-chapter li a").forEach(function(e) {
        var link = e.attr("href");
        if (link.indexOf("http") !== 0) link = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + link;
        
        chaps.push({
            name: e.select(".chapter-text").text().trim(),
            url: link,
            host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
        });
    });
    return Response.success(chaps);
}
```

### 6. Trích xuất Nội dung Chữ (`src/chap.js`)
*Thao tác F12:* Nội dung văn bản thuần nằm trong `<article class="chapter-content">`. Tiến hành gỡ bỏ quảng cáo.

```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    var content = doc.select("article.chapter-content").first();
    if (!content) return null;
    
    // LỌC RÁC: Dùng .remove() để cắt bỏ thẻ mã độc (script), quảng cáo (iframe, .ads)
    content.select("script, iframe, .ads").remove();
    
    return Response.success(content.html());
}
```

---

## PHẦN IV: ĐÓNG GÓI VÀ XUẤT BẢN LÊN GITHUB

Để người dùng cài đặt extension thông qua một đường dẫn URL, cần đẩy mã nguồn lên GitHub và tạo tệp Danh bạ tổng (`plugin.json` ở cấp cao nhất).

**1. Kiến trúc Kho lưu trữ (Repository)**
Tại Repo GitHub (VD: `https://github.com/danpypk-hub/Vbook-ext-`):
```text
Vbook-ext-/
├── truyenmoiyy_pro.zip    (File nén chứa mã nguồn extension)
├── icon_truyenmoiyy.png   (Ảnh đại diện)
└── plugin.json            (File Danh bạ Index dùng để load vào Vbook)
```

**2. Khai báo tệp Danh bạ `plugin.json` (Nằm ngoài cùng Repo)**
File này liệt kê toàn bộ các tiện ích có trong kho lưu trữ.
*(Lưu ý: Mọi đường dẫn trong `path` và `icon` bắt buộc phải là dạng Raw Link của GitHub).*

```json
{
    "metadata": {
        "author": "[Tên_Của_Bạn]",
        "description": "Kho tiện ích Vbook cá nhân"
    },
    "data": [
        {
            "name": "Truyện Mới YY",
            "author": "[Tên_Của_Bạn]",
            "path": "[https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/truyenmoiyy_pro.zip](https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/truyenmoiyy_pro.zip)",
            "version": 1,
            "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
            "icon": "[https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/icon_truyenmoiyy.png](https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/icon_truyenmoiyy.png)",
            "description": "Tiện ích tốc độ cao cho Truyện Mới YY",
            "type": "novel"
        }
    ]
}
```

**3. Cài đặt vào ứng dụng Vbook**
1. Mở ứng dụng Vbook -> Mục **Tiện ích** -> Nhấn dấu **+** -> Chọn **Thêm từ URL**.
2. Dán đường dẫn Raw của file Danh bạ: `https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/plugin.json`
3. Ứng dụng sẽ hiển thị danh mục tiện ích và cho phép cài đặt.

---

## PHẦN V: TIÊU CHUẨN KỸ THUẬT VÀ BEST PRACTICES

### 1. Giới hạn ECMAScript 5 (ES5) - VAR vs LET
Ứng dụng Vbook sử dụng engine JavaScript nhúng trên thiết bị di động (Rhino/QuickJS) được tối ưu để tiết kiệm RAM. Môi trường này **chỉ hỗ trợ cú pháp ES5**.
* Tuyệt đối **KHÔNG** sử dụng `let`, `const`, hoặc hàm mũi tên `() => {}`. Hệ thống sẽ văng lỗi `SyntaxError` và từ chối nạp tiện ích.
* **Quy tắc:** Chỉ sử dụng `var` để khai báo biến và `function() {}` để định nghĩa hàm.

### 2. Checklist cho một Extension hoàn chỉnh
- [ ] `plugin.json` có đầy đủ metadata hợp lệ.
- [ ] Ảnh đại diện `icon.png` có kích thước 64x64px, vuông vức.
- [ ] Hàm bắt lỗi linh hoạt (Kiểm tra `if (!res.ok)` để tránh crash).
- [ ] URL luôn được chuẩn hóa (Bổ sung tiền tố `http` hoặc domain gốc nếu thiếu).
- [ ] Đã sử dụng `content.select(rác).remove()` tại `chap.js`.
- [ ] Đã kiểm thử thuật toán lật trang ở cả mục lục (TOC) và danh mục (GEN).
