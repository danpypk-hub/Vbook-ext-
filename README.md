#  LẬP TRÌNH VBOOK EXTENSION (Gemini viết)

Tài liệu hướng dẫn chi tiết quy trình phát triển tiện ích (Extension) cho ứng dụng Vbook. Kiến thức được hệ thống hóa từ khâu thiết lập môi trường, công cụ, bóc tách dữ liệu (Scraping), đến xuất bản mã nguồn lên GitHub. Case study thực hành thực tế: `truyenmoiyy.com`.

> *Tham khảo thêm mã nguồn các Vbook Extension hoàn thiện tại:* [https://github.com/dat-bi/ext-vbook](https://github.com/dat-bi/ext-vbook)

---

## PHẦN 1: CÔNG CỤ VÀ MÔI TRƯỜNG PHÁT TRIỂN (BUILD & DEBUG)

Extension Vbook đóng vai trò là "cầu nối", tải mã HTML của trang web, trích xuất dữ liệu và định dạng lại để hiển thị trên ứng dụng. Cần chuẩn bị môi trường phát triển như sau:

### 1. Công cụ lập trình
* **Trên Thiết bị di động:** Sử dụng ứng dụng `Vbook Ext Maker` (tải trên Android). Ứng dụng này hỗ trợ tạo khung tệp tin tự động, viết mã, chạy thử (Run/Preview) trực tiếp với URL thực tế và xuất tệp `.zip` tiện lợi.
* **Trên Máy tính (Khuyến nghị):** Sử dụng phần mềm soạn thảo mã nguồn **Visual Studio Code (VSCode)** kết hợp trình duyệt **Google Chrome**.

### 2. Thiết lập kết nối & Gỡ lỗi (Remote Debug)
Phương pháp này giúp mã nguồn đồng bộ trực tiếp từ VSCode (máy tính) lên Vbook (điện thoại) theo thời gian thực mà không cần đóng gói tệp `.zip` nhiều lần.

1.  **Lấy IP:** Đảm bảo máy tính và điện thoại truy cập chung một mạng Wifi. Mở Vbook trên điện thoại -> *Cài đặt* -> Kích hoạt chế độ *Gỡ lỗi (Debug)*. Hệ thống sẽ cung cấp một địa chỉ IP nội bộ (VD: `192.168.1.15:8080`).
2.  **Kết nối:** Trên máy tính, mở trình duyệt Chrome, nhập địa chỉ IP trên vào thanh URL. Giao diện Vbook Extension Server sẽ xuất hiện.
3.  **Kiểm thử (Test):** Viết mã trên VSCode, lưu lại. Kéo thả tệp `plugin.json` (hoặc cả thư mục dự án) vào giao diện web trên Chrome. App Vbook sẽ tự động nạp mã nguồn.
4.  **Gỡ lỗi (Debug):** Nhấn **F12** trên Chrome (máy tính), chuyển sang tab `Console`. Bất kỳ lỗi logic hoặc cú pháp nào khi thao tác trên điện thoại đều được báo đỏ chi tiết tại đây.
5.  **Đóng gói:** Khi hoàn thiện, nén thư mục thành tệp `.zip` (Lưu ý quan trọng: tệp `plugin.json` và `icon.png` phải nằm ở lớp ngoài cùng của tệp nén, không được nằm trong thư mục con).

---

## PHẦN 2: KIẾN TRÚC THƯ MỤC CỦA MỘT EXTENSION

Một extension đạt chuẩn bao gồm cấu trúc tệp tin sau:

```text
Ten_Extension.zip/
├── plugin.json       (Hồ sơ lõi: Định danh tên, tác giả, cấu hình URL)
├── icon.png          (Ảnh đại diện tiện ích, tỷ lệ vuông 1:1)
└── src/              (Thư mục chứa mã nguồn thực thi logic)
    ├── home.js       (Màn hình chính: Chứa các Tab mặc định VD: Truyện mới đăng)
    ├── genre.js      (Danh mục mở rộng: Danh sách Thể loại trong nút "Khám phá")
    ├── gen.js        (Quét danh sách: Trích xuất hàng loạt truyện từ một chuyên mục)
    ├── detail.js     (Quét chi tiết: Thông tin 1 bộ truyện như Tên, Tác giả, Số chương)
    ├── page.js       (Hỗ trợ phân trang: Tính tổng số trang của mục lục nếu quá dài)
    ├── toc.js        (Quét Mục lục: Trích xuất mảng liên kết các chương)
    └── chap.js       (Quét nội dung: Lấy chữ của chương và cắt bỏ quảng cáo)
```

---

## PHẦN 3: KỸ THUẬT F12 & GIẢI PHẪU MÃ NGUỒN (TRUYENMOIYY.COM)

Để trích xuất dữ liệu, bắt buộc phải biết thông tin nằm ở đoạn mã HTML nào. Công cụ **F12 (DevTools)** trên Chrome là công cụ then chốt.

**Cách thao tác:** Mở trang web -> Nhấn `F12` -> Nhấn biểu tượng **Mũi tên chéo** (Select an element) góc trên bên trái -> Rê chuột vào phần tử cần lấy trên giao diện web. Hệ thống sẽ bôi đậm đoạn mã HTML tương ứng.

Dưới đây là giải phẫu chi tiết, tách biệt giữa **Khung chuẩn (Template)** và **Mã thực tế (`truyenmoiyy.com`)**.

### 1. Tệp Hồ sơ `plugin.json`
**Mục đích:** Khai báo cấu hình tiện ích cho hệ thống Vbook.

```json
// ==========================================
// 📌 KHUNG CODE CHUNG (TEMPLATE)
// ==========================================
{
  "metadata": {
    "name": "Tên Website",
    "author": "[Tên_Của_Bạn]",
    "version": 1,
    "source": "[https://domain.com](https://domain.com)",
    "regexp": "domain\\.com", 
    "description": "Mô tả tiện ích",
    "type": "novel", 
    "locale": "vi_VN",
    "language": "javascript"
  },
  "script": { 
    "home": "home.js",
    "genre": "genre.js",
    "detail": "detail.js",
    "toc": "toc.js",
    "chap": "chap.js"
  }
}
```

```json
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY.COM)
// ==========================================
{
  "metadata": {
    "name": "Truyện Mới YY",
    "author": "[Tên_Của_Bạn]",
    "version": 0.1,
    "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
    "regexp": "truyenmoiyy\\.com",
    "description": "Bổ sung chuẩn hiển thị và thuật toán lật trang mục lục.",
    "locale": "vi_VN",
    "language": "javascript",
    "type": "novel"
  },
  "script": {
    "home": "home.js",
    "genre": "genre.js",
    "detail": "detail.js",
    "page": "page.js",
    "toc": "toc.js",
    "chap": "chap.js",
    "search": "search.js"
  }
}
```

---

### 2. Thiết kế Màn hình chính (`home.js`) và Khám phá (`genre.js`)
**Chiến lược thiết kế:** Để tối ưu giao diện, `home.js` chỉ nên chứa 1-3 tab chính hiển thị ngay khi mở tiện ích (Ví dụ: *Truyện mới đăng*). Toàn bộ các chuyên mục hoặc thể loại khác (Tiên hiệp, Ngôn tình...) sẽ được đưa vào `genre.js` để ẩn gọn trong nút menu **Khám phá thêm**.

#### A. Tệp `src/home.js` (Tab Mặc định)
Khai báo tĩnh các chuyên mục chính nhất. Không cần dùng F12, chỉ cần sao chép liên kết từ thanh menu của trang web.

```javascript
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY.COM)
// ==========================================
function execute() {
    return Response.success([
        { title: "Mới Cập Nhật", input: "/danh-sach/truyen-moi", script: "gen.js" },
        { title: "Truyện Hot", input: "/danh-sach/truyen-hot", script: "gen.js" }
    ]);
}
```

#### B. Tệp `src/genre.js` (Trích xuất Thể loại động)
**Thao tác F12:** Mở trang chủ `https://truyenmoiyy.com/`. Rê chuột vào thanh menu thả xuống chứa các thể loại. Bấm F12. Khối danh sách thể loại nằm trong `<ul class="dropdown-menu multi-column">`, mỗi thể loại là một thẻ `<li>` chứa thẻ `<a>`.

```javascript
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY.COM)
// ==========================================
function execute() {
    var res = fetch("[https://truyenmoiyy.com/](https://truyenmoiyy.com/)");
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];
    
    // Quét toàn bộ thẻ 'a' nằm trong menu thể loại
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

---

### 3. Trích xuất Danh sách truyện (`src/gen.js`)
**Mục đích:** Vào trang danh mục, trích xuất Tên, Ảnh, Link của nhiều truyện cùng lúc.
**Thao tác F12:** Mở URL `https://truyenmoiyy.com/danh-sach/truyen-moi`. Bấm F12, trỏ vào một cuốn truyện. Thông tin truyện được bọc trong khối `<div class="row" itemscope>`. Bắt buộc phải dùng khối bọc này làm "Container" để vòng lặp `.forEach()` quét từng truyện mà không bị lẫn lộn dữ liệu.

```javascript
// ==========================================
// 📌 KHUNG CODE CHUNG (TEMPLATE)
// ==========================================
function execute(url, page) {
    var res = fetch(url + "?page=" + (page || '1')); 
    var doc = res.html();
    var data = [];
    
    doc.select("KHỐI_BỌC_TRUYỆN_CONTAINER").forEach(function(e) {
        data.push({
            name: e.select("THẺ_TÊN").text().trim(),
            link: e.select("THẺ_LINK").attr("href"),
            cover: e.select("THẺ_ẢNH").attr("src"),
            host: "[https://domain.com](https://domain.com)"
        });
    });
    return Response.success(data, "TRANG_TIẾP_THEO");
}
```

```javascript
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY)
// ==========================================
function execute(url, page) {
    if (!page) page = '1';
    
    // Xử lý logic URL đặc thù: Web này dùng /trang-1 thay vì ?page=1
    var fetchUrl = url;
    if (fetchUrl.indexOf("http") !== 0) fetchUrl = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + url;
    fetchUrl = fetchUrl + "/trang-" + page;

    var res = fetch(fetchUrl);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];

    // Chọn Container chứa thông tin của từng truyện
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

    // Lật trang: Kiểm tra xem có thẻ a nào chứa thuộc tính rel=next hay không
    var hasNext = doc.select(".pagination li a[rel=next]").length > 0;
    if (hasNext) return Response.success(data, (parseInt(page) + 1).toString());
    
    return Response.success(data);
}
```

---

### 4. Trích xuất Chi tiết truyện (`src/detail.js`)
**Mục đích:** Lấy siêu dữ liệu (Metadata) như Tác giả, Mô tả, Số chương của 1 truyện.
**Thao tác F12:** Mở URL `https://truyenmoiyy.com/no-le-bong-toi-q7-lang-mo-ariel/`.
* **Tên truyện:** Trỏ vào tên truyện -> Thấy nằm trong `<h1 class="story-title">`.
* **Tác giả:** Trỏ vào tên tác giả -> Thấy nằm trong `<a itemprop="name">` thuộc khối cha `.author`.
* **Số chương:** Trỏ vào vùng có ghi số chương. Khối HTML thường bị phân mảnh hoặc không có thẻ bọc riêng. **Giải pháp:** Quét toàn bộ văn bản thuần trên trang bằng `doc.text()` và dùng `Regex` (biểu thức chính quy) để bắt chính xác con số.

```javascript
// ==========================================
// 📌 KHUNG CODE CHUNG (TEMPLATE)
// ==========================================
function execute(url) {
    var res = fetch(url);
    var doc = res.html();
    
    return Response.success({
        name: doc.select("THẺ_TÊN").text().trim(),
        cover: doc.select("THẺ_ẢNH").attr("src"),
        author: doc.select("THẺ_TÁC_GIẢ").text().trim(),
        description: doc.select("THẺ_MÔ_TẢ").text().trim(),
        detail: "Hiển thị thông tin phụ",
        host: "[https://domain.com](https://domain.com)"
    });
}
```

```javascript
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY)
// ==========================================
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    // Ưu tiên class .story-title, dùng thẻ h1 làm lớp dự phòng (||)
    var name = doc.select(".story-title").text().trim() || doc.select("h1").text().trim();
    var cover = doc.select(".book img").first().attr("src");
    if (cover && cover.indexOf("http") !== 0) cover = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + cover;
    
    var authorEl = doc.select("[itemprop=author] [itemprop=name]").first() || doc.select(".author").first();
    // Loại bỏ chuỗi "Tác giả:" thừa bị dính liền trong text
    var author = authorEl ? authorEl.text().replace(/Tác giả\s*:/i, "").trim() : "Đang cập nhật";
    
    // Kỹ thuật Regex: Dò tìm Số chương từ toàn bộ khối văn bản
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
        description: doc.select(".desc-text").text().trim(), // Vị trí khối thẻ mô tả
        detail: detailInfo.join("<br>"),
        host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
    });
}
```

---

### 5. Trích xuất Mục lục (`src/toc.js`)
**Mục đích:** Trích xuất toàn bộ liên kết của các chương vào một mảng.
**Thao tác F12:** Vẫn ở URL chi tiết truyện `.../no-le-bong-toi-q7-lang-mo-ariel/`, cuộn xuống danh sách chương. Trỏ vào "Chương 1". Quan sát cây HTML sẽ thấy toàn bộ các chương được bọc trong danh sách `<ul class="list-chapter">`, mỗi chương là một thẻ `<li>` chứa thẻ `<a>`. **Nguyên tắc:** Bắt buộc lấy từ thẻ cha `ul` để mã không quét nhầm các liên kết bài viết khác nằm rải rác trên trang.

```javascript
// ==========================================
// 📌 KHUNG CODE CHUNG (TEMPLATE)
// ==========================================
function execute(url) {
    var res = fetch(url);
    var doc = res.html();
    var chaps = [];
    
    doc.select("KHỐI_BỌC_MỤC_LỤC THẺ_A").forEach(function(e) {
        chaps.push({ name: e.text().trim(), url: e.attr("href"), host: "[https://domain.com](https://domain.com)" });
    });
    return Response.success(chaps);
}
```

```javascript
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY)
// ==========================================
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var chaps = [];
    
    // Quét chính xác các thẻ a nằm trong khối list-chapter
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

---

### 6. Trích xuất Nội dung Chữ (`src/chap.js`)
**Mục đích:** Lấy đoạn văn bản thuần để đọc, vứt bỏ quảng cáo.
**Thao tác F12:** Mở URL `https://truyenmoiyy.com/no-le-bong-toi-q7-lang-mo-ariel/chuong-1`. Trỏ vào một đoạn văn bản bất kỳ. Cây HTML chỉ ra rằng mọi đoạn `<p>` đều nằm chung trong một thẻ cha `<article class="chapter-content">`. Lấy toàn bộ khối `article` này để thu thập trọn vẹn văn bản.

```javascript
// ==========================================
// 📌 KHUNG CODE CHUNG (TEMPLATE)
// ==========================================
function execute(url) {
    var res = fetch(url);
    var doc = res.html();
    
    var content = doc.select("KHỐI_BỌC_CHỮ").first();
    // Tiêu diệt mã độc, quảng cáo
    content.select("script, iframe, .ads").remove();
    
    return Response.success(content.html());
}
```

```javascript
// ==========================================
// 🎯 CODE THỰC TẾ (TRUYENMOIYY)
// ==========================================
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    // Lấy vùng chứa chữ chính
    var content = doc.select("article.chapter-content").first();
    if (!content) return null;
    
    // LỌC RÁC: Dùng .remove() để triệt tiêu mã độc (script), quảng cáo (iframe, div mang class ads)
    content.select("script, iframe, .ads").remove();
    
    return Response.success(content.html());
}
```

---

## PHẦN 4: ĐÓNG GÓI VÀ XUẤT BẢN LÊN GITHUB

Cần đẩy tiện ích lên kho lưu trữ GitHub và cấu hình tệp Danh bạ tổng (`plugin.json` định dạng Data Array) để cho phép thiết bị di động cài đặt từ xa thông qua một đường dẫn URL.

### 1. Kiến trúc Kho lưu trữ (Repository)
Tại Repository GitHub cá nhân (Ví dụ: `https://github.com/danpypk-hub/Vbook-ext-`), khởi tạo cấu trúc sau:

```text
Vbook-ext-/
├── truyenmoiyy_pro.zip    (File nén chứa toàn bộ mã nguồn extension đã đóng gói)
├── icon_truyenmoiyy.png   (Ảnh đại diện hiển thị ngoài danh sách)
└── plugin.json            (File Danh bạ tổng / Index File)
```

### 2. Viết file Danh bạ `plugin.json` (Nằm ngoài cùng Repository)
File này liệt kê toàn bộ các tiện ích có trong kho lưu trữ của bạn.
*Lưu ý: Mọi đường dẫn trong trường `path` và `icon` bắt buộc phải là định dạng Raw Link. Lấy bằng cách bấm vào tệp trên giao diện GitHub và chọn nút **Raw**.*

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
            "version": 0.1,
            "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
            "icon": "[https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/icon_truyenmoiyy.png](https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/icon_truyenmoiyy.png)",
            "description": "Tiện ích bóc tách tốc độ cao cho Truyện Mới YY.",
            "type": "novel"
        }
        // Tiếp tục thêm cấu hình của các tiện ích khác vào mảng này...
    ]
}
```

### 3. Cài đặt vào ứng dụng Vbook
1.  Mở ứng dụng Vbook -> Mục **Tiện ích (Extensions)** -> Nhấn dấu **+** -> Chọn **Thêm từ URL**.
2.  Dán đường dẫn Raw của file Danh bạ vừa tạo:
    `https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/plugin.json`
3.  Ứng dụng tự động phân tích tệp JSON, hiển thị danh mục tiện ích và cho phép cài đặt với 1 chạm. Hệ thống sẽ tự động thông báo nhận diện bản cập nhật khi trường `version` trong tệp danh bạ thay đổi.

---

## 📌 GHI CHÚ KỸ THUẬT QUAN TRỌNG: VAR vs LET
1.  **Bản chất Engine:** Ứng dụng Vbook sử dụng engine JavaScript nhúng trên thiết bị di động (như Rhino/QuickJS) được thiết kế tối giản nhằm tiết kiệm tài nguyên hệ thống (RAM).
2.  **Chuẩn ES5:** Môi trường này **chỉ hỗ trợ và biên dịch được cú pháp chuẩn ES5**. Tuyệt đối không sử dụng các cú pháp JavaScript hiện đại của ES6+ như `let`, `const`, hoặc hàm mũi tên `() => {}`.
3.  **Hậu quả:** Việc vi phạm chuẩn ES5 sẽ khiến hệ thống không thể đọc hiểu mã nguồn, lập tức văng lỗi `SyntaxError` và từ chối hoạt động.
4.  **Quy tắc:** Mọi khai báo biến bắt buộc phải sử dụng từ khóa `var`. Mọi khai báo hàm bắt buộc sử dụng định dạng truyền thống `function() {}`.
