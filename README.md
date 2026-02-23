## Viết Vbook ext (Gemini)
# Kiến thức được hệ thống hóa từ nền tảng HTML/CSS/JS, kỹ thuật cào dữ liệu (Web Scraping), đối chiếu API gốc và xuất bản lên GitHub.

> *Tham khảo mã nguồn các Vbook Extension hoàn thiện tại:* [https://github.com/dat-bi/ext-vbook](https://github.com/dat-bi/ext-vbook)

---

## PHẦN I: KIẾN THỨC NỀN TẢNG VÀ CHECKLIST BẮT BUỘC

Để làm extension, cần hiểu cơ bản về cấu trúc web:
* **HTML:** Là bộ khung xương của trang web. Dữ liệu (chữ, ảnh) luôn được bọc trong các "thẻ" (tags) như `<div>`, `<a>` (chứa link), `<img>` (chứa ảnh), `<p>` (đoạn văn).
* **CSS:** Là lớp áo trang trí. Các thẻ HTML thường được gắn tên để dễ nhận diện thông qua thuộc tính `class` hoặc `id` (Ví dụ: `<div class="truyen-title">`).
* **JavaScript (JS):** Kịch bản dùng để ra lệnh cho máy tính "đi tìm thẻ HTML có class là `truyen-title` và lấy chữ bên trong ra". (Lưu ý: Vbook chỉ hỗ trợ JS chuẩn **ES5** - bắt buộc dùng `var` và `function`).

### Checklist Cấu trúc Thư mục (Bắt buộc)
Extension là một tệp `.zip` nén các file lại. Để cài đặt thành công, phải tuân thủ vị trí file:

- [ ] Tệp `plugin.json` (Hồ sơ cấu hình) phải nằm ngoài cùng (Root).
- [ ] Ảnh `icon.png` (Kích thước 64x64px, tỷ lệ 1:1) phải nằm ngoài cùng (Root).
- [ ] Toàn bộ các file `.js` còn lại **BẮT BUỘC** nằm trong thư mục `src/`.
- [ ] Khi nén, chỉ bôi đen `plugin.json`, `icon.png` và thư mục `src` rồi chọn *Add to archive (.zip)*.

```text
Ten_Extension.zip/
├── plugin.json       # Metadata và cấu hình (Bắt buộc)
├── icon.png          # Icon đại diện (Bắt buộc)
└── src/              # Thư mục chứa mã nguồn (Bắt buộc)
    ├── home.js       # Trang chủ - Các Tab mặc định
    ├── genre.js      # Danh mục Thể loại
    ├── search.js     # Chức năng tìm kiếm
    ├── gen.js        # Script quét danh sách truyện
    ├── detail.js     # Script lấy chi tiết truyện
    ├── page.js       # Phân trang mục lục
    ├── toc.js        # Script lấy mục lục
    └── chap.js       # Script nội dung chương
```

---

## PHẦN II: THỦ THUẬT ĐỌC SOURCE CODE (F12 DEVTOOLS)

Kỹ năng quan trọng nhất là xác định "Container" (Khối bọc) trên mã HTML thông qua DevTools (F12) của Chrome.

1.  **Mở F12:** Nhấn phím `F12` trên trình duyệt Chrome. Chọn biểu tượng **Mũi tên** ở góc trái trên cùng.
2.  **Xác định Container:** Rê chuột vào toàn bộ khung của 1 cuốn truyện. Đừng chỉ vào riêng cái tên hay tấm ảnh. Hãy tìm thẻ `<div>` hoặc `<li>` bọc trọn vẹn cả ảnh, tên, tác giả của cuốn truyện đó. 
3.  **Tại sao phải tìm Container?** Để tạo vòng lặp. Nếu trang web có 20 truyện, ta ra lệnh cho JS "tìm 20 cái hộp (container) này, sau đó mở từng hộp ra để lấy tên và ảnh bên trong", như vậy dữ liệu sẽ không bao giờ bị râu ông nọ cắm cằm bà kia.

### 1. Tại sao phải chiết xuất Container?
Nếu bạn dùng `doc.select("a").attr("href")` để lấy link truyện, bạn sẽ lấy nhầm cả link trang chủ, link quảng cáo, link tác giả. Do đó, bạn phải tìm khối HTML bọc trọn vẹn 1 đối tượng, gọi là Container.

### 2. Cách thực hiện (Ví dụ trên `gen.js` - Trang danh sách)
1. Mở trang web (VD: `https://truyenmoiyy.com/danh-sach/truyen-moi`). Nhấn **F12** -> Chọn biểu tượng Mũi tên góc trái trên.
2. Rê chuột vào **toàn bộ khung của 1 cuốn truyện**. Quan sát cây HTML, bạn sẽ thấy nó nằm trong một thẻ `<div class="row" itemscope>`.
3. **Cách viết code:**
   ```javascript
   // 1. Dùng Container làm vòng lặp:
   doc.select(".list-truyen .row[itemscope]").forEach(function(e) {
// Biến 'e' lúc này đại diện cho duy nhất 1 truyện.
       // 2. Chỉ truy xuất giá trị bên trong 'e', không dùng 'doc' nữa:
       var name = e.select(".truyen-title a").text();
       var cover = e.select("img[itemprop=image]").attr("src");
   });
   ```
### 3. Cách lấy giá trị cho `detail.js` (Trang chi tiết)
* F12 vào Tên truyện -> Copy class của thẻ tiêu đề (VD: `h1.story-title`).
* F12 vào phần chứa Số chương -> Nếu web không bọc số chương bằng thẻ nào, hãy lấy toàn văn trang bằng `doc.text()` và sử dụng kỹ thuật Regex (Xem Phần V) để dò tìm cụm chữ "Số chương: XXX".

## PHẦN III: GIẢI PHẪU MÃ NGUỒN CÁC SCRIPT CHÍNH

Dưới đây là các tệp kịch bản. **[KHUNG CODE CHUẨN]** là mã gốc từ tài liệu Vbook API (giữ nguyên chú thích gốc). **[CODE THỰC TẾ]** minh họa cách ứng dụng trên trang `truyenmoiyy.com`, với các dòng code được dãn cách và giải thích nội tuyến.

### 1. Tệp Hồ sơ `plugin.json`

**[KHUNG CODE CHUẨN - VBOOK API]**
```json
{
    "metadata": {
        "name": "Tên Extension",
        "author": "Tên tác giả", 
        "version": 1,
        "source": "[https://website.com](https://website.com)",
        "regexp": "website\\.com/truyen/\\d+",
        "description": "Mô tả extension",
        "locale": "vi_VN",          // vi_VN, zh_CN, en_US, ja_JP
        "language": "javascript",
        "type": "novel",            // novel, comic, chinese_novel
        "tag": "nsfw"               // Chỉ thêm nếu có nội dung 18+
    },
    "script": {
        "home": "home.js",          // Trang chủ (tùy chọn)
        "genre": "genre.js",        // Thể loại (tùy chọn)
        "detail": "detail.js",      // Chi tiết truyện (bắt buộc)
        "search": "search.js",      // Tìm kiếm (tùy chọn)
        "page": "page.js",          // Phân trang mục lục (tùy chọn)
        "toc": "toc.js",            // Mục lục (bắt buộc)
        "chap": "chap.js"           // Nội dung chương (bắt buộc)
    }
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```json
{
    "metadata": {
        "name": "Truyện Mới YY",
        "author": "Danpypk",
        "version": 1,
        "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
        "regexp": "truyenmoiyy\\.com",
        "description": "Tiện ích cào dữ liệu tối ưu cho Truyện Mới YY.",
        "locale": "vi_VN",
        "language": "javascript",
        "type": "novel"
    },
    "script": {
        "home": "home.js",
        "genre": "genre.js",
        "gen": "gen.js",
        "detail": "detail.js",
        "page": "page.js",
        "toc": "toc.js",
        "chap": "chap.js",
        "search": "search.js"
    }
}
```

---

### 2. Màn hình chính (`home.js`) & Khám phá (`genre.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
// HOME.JS - Trang chủ:
function execute() {
    return Response.success([
        {
            title: "Truyện mới",              // Tên hiển thị trên app
            input: "[https://website.com/new](https://website.com/new)", // URL đầu vào truyền cho script xử lý
            script: "gen.js"                  // Tên file script sẽ nhận URL này
        }
    ]);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
// --- src/home.js ---
// Không cần F12, chỉ cần copy các link muốn hiển thị ngay trên Tab trang chủ
function execute() {
    return Response.success([
        { 
            title: "Mới Cập Nhật", 
            input: "/danh-sach/truyen-moi", 
            script: "gen.js" 
        },
        { 
            title: "Truyện Hot", 
            input: "/danh-sach/truyen-hot", 
            script: "gen.js" 
        }
    ]);
}

// --- src/genre.js ---
// Dùng F12 soi vào thanh Menu của web để tự động cào toàn bộ danh sách Thể loại
function execute() {
    var res = fetch("[https://truyenmoiyy.com/](https://truyenmoiyy.com/)");
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];
    
    // CSS Selector: Tìm khối ul mang class 'dropdown-menu multi-column', lấy tất cả thẻ 'li a' bên trong
    doc.select(".dropdown-menu.multi-column li a").forEach(function(e) {
        data.push({
            title: e.text().trim(),      // text() lấy chữ hiển thị (VD: Tiên Hiệp)
            input: e.attr("href"),       // attr("href") lấy đường link ẩn bên trong thẻ <a>
            script: "gen.js"             // Ném link này cho file gen.js xử lý
        });
    });
    return Response.success(data);
}
```

---

### 3. Danh sách truyện (`gen.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
function execute(url, page) {
    if (!page) page = "1";
    let pageUrl = `${url}?page=${page}`; // Cách phân trang phổ biến
    let response = fetch(pageUrl);
    if (!response.ok) return Response.error(`Không thể tải trang ${page}`);
    
    let doc = response.html();
    const data = [];
    
    // Vòng lặp lấy danh sách truyện từ Container
    doc.select(".book-list .item").forEach(element => {
        let name = element.select(".title a, h3 a").text();
        let link = element.select(".title a, h3 a").attr("href");
        let cover = element.select("img").attr("src");
        let description = element.select(".description, .summary").text();
        
        if (!name || !link) return; // Bỏ qua nếu lỗi
        
        data.push({
            name: name,                       // Tên truyện (bắt buộc)
            link: link,                       // URL truyện (bắt buộc)
            cover: cover,                     // URL ảnh bìa
            description: description,         // Mô tả ngắn
            host: "[https://website.com](https://website.com)"       // Domain (tùy chọn)
        });
    });
    
    let nextPage = null;
    // Tìm nút Next để lật trang
    let nextLink = doc.select("a.next, a:contains(下一页), a:contains(Next)").attr("href");
    if (nextLink) {
        let pageNum = nextLink.match(/(\d+)/);
        if (pageNum) nextPage = pageNum[1];
    }
    
    return Response.success(data, nextPage);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(url, page) {
    if (!page) page = '1';
    
    // Chuẩn hóa URL: Web này dùng định dạng "/trang-1" thay vì "?page=1"
    var fetchUrl = url;
    if (fetchUrl.indexOf("http") !== 0) {
        fetchUrl = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + url;
    }
    fetchUrl = fetchUrl + "/trang-" + page;

    var res = fetch(fetchUrl);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];

    // TÌM CONTAINER: F12 chỉ ra mỗi truyện được bọc trong <div class="row" itemscope> nằm trong <div class="list-truyen">
    doc.select(".list-truyen .row[itemscope]").forEach(function(e) {
        
        // Từ Container 'e', tìm xuống các thẻ con
        var a = e.select(".truyen-title a").first();      // Thẻ <a> chứa Tên và Link
        var img = e.select("img[itemprop=image]").first();// Thẻ <img> chứa Ảnh bìa
        var author = e.select(".author").first();         // Thẻ chứa Tên tác giả
        
        if (a && img) {
            var link = a.attr("href");
            // Đảm bảo link luôn là link tuyệt đối (có http)
            if (link.indexOf("http") !== 0) link = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + link;
            
            data.push({
                name: a.text().trim(),                     // Lấy tên truyện
                link: link,                                // Lấy link truyện
                cover: img.attr("src"),                    // Lấy link ảnh
                description: author ? author.text().trim() : "", 
                host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
            });
        }
    });

    // LẬT TRANG: Tìm thẻ <a> có thuộc tính rel="next"
    var hasNext = doc.select(".pagination li a[rel=next]").length > 0;
    if (hasNext) {
        // Trả về dữ liệu và báo cho app biết trang tiếp theo là page + 1
        return Response.success(data, (parseInt(page) + 1).toString());
    }
    
    return Response.success(data);
}
```

---

### 4. Chi tiết truyện (`detail.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
function execute(url) {
    url = url.replace("m.website.com", "[www.website.com](https://www.website.com)");
    let response = fetch(url);
    let doc = response.html();
    
    // Lấy thông tin cơ bản bằng các selector thông dụng
    let name = doc.select("h1.title, .book-title").text();
    let cover = doc.select(".cover img, .book-cover img").attr("src");
    let author = doc.select(".author, .book-author").text().replace(/Tác giả:\s*/g, "");
    let description = doc.select(".description, .book-desc").html();
    let status = doc.select(".status, .book-status").text();
    
    let ongoing = true;
    if (status.includes("Hoàn thành") || status.includes("Completed")) ongoing = false;
    
    let detail = `Tác giả: ${author}<br>Trạng thái: ${status}<br>`;
    
    return Response.success({
        name: name,                           // Tên truyện (bắt buộc)
        cover: cover,                         // URL ảnh bìa
        host: "[https://website.com](https://website.com)",          // Domain
        author: author,                       // Tác giả
        description: description,             // Mô tả HTML
        detail: detail,                       // Thông tin chi tiết
        ongoing: ongoing                      // true = đang ra, false = hoàn thành
    });
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    // Lấy Tên: Thử lấy class .story-title, nếu web đổi thì lấy thẻ h1 làm dự phòng (||)
    var name = doc.select(".story-title").text().trim() || doc.select("h1").text().trim();
    
    // Lấy Ảnh: Ưu tiên lấy thẻ meta og:image trên Header của HTML vì đây là ảnh nét nhất
    var cover = doc.select(".book img").first().attr("src") || doc.select("meta[property='og:image']").attr("content");
    if (cover && cover.indexOf("http") !== 0) cover = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + cover;
    
    // Lấy Tác giả:
    var authorEl = doc.select("[itemprop=author] [itemprop=name]").first() || doc.select(".author").first();
    // Dùng Regex xóa cụm từ "Tác giả:" bị dính liền trong text
    var author = authorEl ? authorEl.text().replace(/Tác giả\s*:/i, "").trim() : "Đang cập nhật";
    
    // Lấy Số chương: Do HTML thường bị vỡ, lấy TOÀN BỘ CHỮ trên web và dùng Regex quét
    var rawText = doc.text(); 
    var chapters = "Đang cập nhật";
    // Biểu thức Regex tìm cụm "Số chương: 100" hoặc "Số chương 100"
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

### 5. Mục lục (`toc.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
function execute(url) {
    let response = fetch(url);
    let doc = response.html();
    const chapters = [];
    
    // Lấy danh sách chương - thử các selector khác nhau
    let chapterElements = doc.select(".chapter-list a");
    if (chapterElements.size() === 0) {
        chapterElements = doc.select("#list a, .volume-chapters a, .chapter-item a");
    }
    
    chapterElements.forEach(element => {
        let name = element.text().trim();
        let chapterUrl = element.attr("href");
        if (!name || !chapterUrl) return;
        
        chapters.push({
            name: name,                       // Tên chương (bắt buộc)
            url: chapterUrl,                  // URL chương (bắt buộc) 
            host: "[https://website.com](https://website.com)"       
        });
    });
    return Response.success(chapters);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var chaps = [];
    
    // F12: Các chương nằm trong danh sách <ul> có class 'list-chapter'
    // Đi từ thẻ cha để tuyệt đối không quét nhầm link quảng cáo bên ngoài
    doc.select(".list-chapter li a").forEach(function(e) {
        var link = e.attr("href");
        if (link.indexOf("http") !== 0) link = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + link;
        
        chaps.push({
            name: e.select(".chapter-text").text().trim(), // Tên chương
            url: link,                                     // Link chương
            host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
        });
    });
    
    return Response.success(chaps);
}
```

---

### 6. Nội dung chương (`chap.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
function execute(url) {
    let response = fetch(url);
    let doc = response.html();
    
    // LỌC RÁC QUAN TRỌNG: Xóa các phần không cần thiết
    doc.select(".ads, .advertisement, .banner, script, style, .comment").remove();
    
    let content = doc.select("#content").html();
    if (!content) content = doc.select(".chapter-content").html();
    
    // Clean nội dung bằng Regex
    content = content.replace(/&nbsp;/g, " ");
    content = content.replace(/\s+/g, " ");
    content = content.trim();
    
    return Response.success(content);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    // F12: Xác định khối HTML bọc duy nhất phần chữ để đọc
    var content = doc.select("article.chapter-content").first();
    if (!content) return null;
    
    // BỘ LỌC RÁC: Dùng .remove() để cắt bỏ thẻ mã độc (script), quảng cáo (iframe), và div chứa QC (.ads)
    content.select("script, iframe, .ads").remove();
    
    // Trả về HTML đã sạch sẽ, Vbook sẽ tự động hiển thị thành văn bản đọc
    return Response.success(content.html());
}
```

---

## PHẦN IV: THƯ VIỆN SNIPPETS CỰC KỲ HỮU ÍCH (JSOUP & REGEX)
*(Trích xuất từ tài liệu `vbook-snippet-Mol.txt`)*

**1. Kỹ thuật Jsoup (Bộ chọn HTML)**
```javascript
// Xóa một phần tử theo ID
doc.select("#contentdp").remove();

// Lấy nội dung từ thẻ meta (Thường dùng lấy ảnh bìa HD, hoặc lấy mô tả)
doc.select('meta[property="og:novel:author"]').attr("content");

// Lấy phần tử theo vị trí (Rất hay dùng khi tên class bị trùng lặp)
doc.select(".item").first(); // Lấy cái đầu tiên
doc.select(".item").last();  // Lấy cái cuối cùng
doc.select(".item").get(0);  // Lấy cái ở vị trí số 0 (index)

// Tìm thẻ <a> chứa đoạn chữ cụ thể (Cực hữu ích để tìm nút Next phân trang)
doc.select("a:contains(下一页)").first().attr("href");
doc.select("a:contains(Trang tiếp)").first().attr("href");
```

**2. Kỹ thuật Xử lý Chuỗi (String)**
```javascript
// Thêm dấu "/" vào cuối URL nếu chưa có
if(url.slice(-1) !== "/") url = url + "/";

// Kiểm tra xem chuỗi có chứa từ khóa không
if (url.indexOf("string") !== -1) { /* Có chứa */ }

// Lấy ID cuối cùng của một URL (VD: [domain.com/truyen-a/000124](https://domain.com/truyen-a/000124) => lấy 000124)
let id = url.split(/[\/ ]+/).pop();

// Chuyển đối tượng JSON thành chuỗi (Stringify) và ngược lại (Parse)
var stringData = JSON.stringify(obj);
var jsonObject = JSON.parse(stringData);
```

**3. Hàm Slugify (Tạo link chuẩn từ Tiếng Việt)**
```javascript
// Biến "tôi là ai" thành "toi-la-ai" để nhét vào URL
function slugify(e) {
    var a="àáäâãåăæąçćčđďèéěėëêęğǵḧìíïîįłḿǹńňñòóöôœøṕŕřßşśšșťțùúüûǘůűūųẃẍÿýźžż·/_,:;";
    var r=new RegExp(a.split("").join("|"),"g");
    return e.toString().toLowerCase()
        .replace(/á|à|ả|ạ|ã|ă|ắ|ằ|ẳ|ẵ|ặ|â|ấ|ầ|ẩ|ẫ|ậ/gi,"a")
        .replace(/é|è|ẻ|ẽ|ẹ|ê|ế|ề|ể|ễ|ệ/gi,"e")
        .replace(/i|í|ì|ỉ|ĩ|ị/gi,"i")
        .replace(/ó|ò|ỏ|õ|ọ|ô|ố|ồ|ổ|ỗ|ộ|ơ|ớ|ờ|ở|ở|ỡ|ợ/gi,"o")
        .replace(/ú|ù|ủ|ũ|ụ|ư|ứ|ừ|ử|ữ|ự/gi,"u")
        .replace(/ý|ỳ|ỷ|ỹ|ỵ/gi,"y").replace(/đ/gi,"d")
        .replace(/\s+/g,"-").replace(r,e=>"aaaaaaaaacccddeeeeeeegghiiiiilmnnnnooooooprrsssssttuuuuuuuuuwxyyzzz------".charAt(a.indexOf(e)))
        .replace(/&/g,"-and-").replace(/[^\w\-]+/g,"")
        .replace(/\-\-+/g,"-").replace(/^-+/,"").replace(/-+$/,"");
}
```

---

## PHẦN V: ĐÓNG GÓI VÀ XUẤT BẢN LÊN GITHUB

Để chia sẻ extension, bạn cần đẩy mã nguồn lên GitHub và tạo một tệp `plugin.json` tổng đóng vai trò như một kho (Store).

### 1. Chuẩn bị kho lưu trữ bằng GitHub Desktop & VSCode
1. Truy cập GitHub, tạo Repository (VD: `Vbook-ext-`).
2. Mở phần mềm **GitHub Desktop** trên máy tính, chọn *Clone a repository* để tải repo đó về thư mục nội bộ.
3. Mở thư mục nội bộ đó bằng **VSCode**. Chép tệp `truyenmoiyy_pro.zip` và `icon_truyenmoiyy.png` vào.

### 2. Khai báo tệp Danh bạ tổng (`plugin.json` ở Root Repo)
Tại thư mục gốc, tạo tệp `plugin.json`. Tệp này khai báo mảng `data` chứa tất cả các tiện ích của bạn.
*Lưu ý quan trọng: Thuộc tính `path` và `icon` BẮT BUỘC phải dùng định dạng Raw Link của GitHub (có tiền tố `raw.githubusercontent.com`). Lấy link này bằng cách bấm vào tệp trên web GitHub -> Nhấn nút **Raw**.*

```json
{
    "metadata": {
        "author": "Danpypk",
        "description": "Kho tiện ích Vbook cá nhân"
    },
    "data": [
        {
            "name": "Truyện Mới YY",
            "author": "Danpypk",
            "path": "[https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/truyenmoiyy_pro.zip](https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/truyenmoiyy_pro.zip)",
            "version": 1,
            "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
 "icon": "[https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/icon_truyenmoiyy.png](https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/icon_truyenmoiyy.png)",
            "description": "Tiện ích tốc độ cao cho Truyện Mới YY.",
            "type": "novel"
        }
        // Có thể phẩy (,) và dán khối tương tự để thêm extension thứ 2, 3...
    ]
}
```

### 3. Đẩy lên GitHub (Push)
* **Trên VSCode:** Bấm icon *Source Control* (cột trái) -> Bấm dấu `+` -> Nhập message commit -> Bấm `Commit` -> Bấm `Sync Changes`.
* **Trên GitHub Desktop:** Xem thay đổi ở cột trái -> Nhập Summary ở góc trái dưới -> Bấm `Commit to main` -> Bấm `Push origin` ở thanh trên cùng.
 ### 4. Link Import cho ứng dụng Vbook
Để người dùng tự động cài đặt tất cả tiện ích trong kho của bạn:
1. Mở app Vbook -> **Tiện ích** -> Nhấn dấu **+** -> **Thêm từ URL**.
2. Dán đường dẫn Raw của tệp Danh bạ tổng vừa tạo. Cú pháp bắt buộc:
   `https://raw.githubusercontent.com/danpypk-hub/Vbook-ext-/main/plugin.json`
3. Ứng dụng sẽ hiển thị danh mục và tự động thông báo cập nhật khi `version` thay đổi.
