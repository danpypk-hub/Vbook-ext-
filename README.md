# Gemini soạn cách tạoVBOOK EXTENSION

Tài liệu hướng dẫn chi tiết quy trình phát triển tiện ích (Extension) cho ứng dụng Vbook. Kiến thức được hệ thống hóa từ khâu thiết lập môi trường, kỹ thuật cào dữ liệu (Web Scraping), đối chiếu API gốc và xuất bản lên GitHub.

> *Tham khảo mã nguồn các Vbook Extension hoàn thiện tại:* [https://github.com/dat-bi/ext-vbook](https://github.com/dat-bi/ext-vbook)

---

## PHẦN I: TIÊU CHUẨN KIẾN TRÚC VÀ CHECKLIST BẮT BUỘC

Một extension Vbook là một gói `.zip` chứa cấu trúc thư mục phân tầng. Để extension có thể cài đặt được, **bắt buộc tuân thủ Checklist sau:**

- [ ] Tệp `plugin.json` phải nằm ở ngoài cùng (Root).
- [ ] Ảnh đại diện `icon.png` (kích thước 64x64px, tỷ lệ 1:1) phải nằm ở ngoài cùng (Root).
- [ ] Toàn bộ các file `.js` còn lại **BẮT BUỘC** phải nằm trong thư mục `src/`.
- [ ] KHÔNG nén nguyên cả một thư mục cha, mà phải bôi đen `plugin.json`, `icon.png`, và thư mục `src` rồi chọn Add to archive (Zip).
- [ ] Mọi file JavaScript phải tuân thủ chuẩn **ES5** (Luôn dùng `var` và `function()`. Tuyệt đối không dùng `let`, `const`, hoặc `=>`).

**Cấu trúc cây thư mục chuẩn:**
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

## PHẦN II: THỦ THUẬT ĐỌC SOURCE CODE & CHIẾT XUẤT CONTAINER (F12)

Kỹ năng quan trọng nhất khi làm extension là biết cách xác định "Container" (Khối bọc) trên mã nguồn HTML thông qua công cụ DevTools (F12) của Chrome.

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

---

## PHẦN III: GIẢI PHẪU MÃ NGUỒN CÁC SCRIPT CHÍNH

Dưới đây là các tệp kịch bản. **[KHUNG CODE CHUẨN]** là mã gốc copy nguyên bản 100% từ tài liệu API chính thức của Vbook (giữ nguyên mọi chú thích định hướng). **[CODE THỰC TẾ]** là cách áp dụng chuẩn đó để code cho trang mẫu `truyenmoiyy.com`.

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
    "tag": "nsfw"              // Chỉ thêm nếu có nội dung 18+
  },
  "script": {
    "home": "home.js",         // Trang chủ (tùy chọn)
    "genre": "genre.js",       // Thể loại (tùy chọn)
    "detail": "detail.js",     // Chi tiết truyện (bắt buộc)
    "search": "search.js",     // Tìm kiếm (tùy chọn)
    "page": "page.js",         // Phân trang mục lục (tùy chọn)
    "toc": "toc.js",           // Mục lục (bắt buộc)
    "chap": "chap.js"          // Nội dung chương (bắt buộc)
  }
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```json
{
  "metadata": {
    "name": "Truyện Mới YY",
    "author": "Tên_Của_Bạn",
    "version": 0.1,
    "source": "[https://truyenmoiyy.com](https://truyenmoiyy.com)",
    "regexp": "truyenmoiyy\\.com",
    "description": "Tiện ích tối ưu tốc độ và cào dữ liệu cho Truyện Mới YY.",
    "locale": "vi_VN",
    "language": "javascript",
    "type": "novel"
  },
  "script": {
    "home": "home.js", "genre": "genre.js", "detail": "detail.js",
    "page": "page.js", "toc": "toc.js", "chap": "chap.js", "search": "search.js"
  }
}
```

---

### 2. Trang chủ & Khám phá (`home.js` & `genre.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
// HOME.JS - Trang chủ:
function execute() {
    // Trả về danh sách các tab cho trang khám phá
    return Response.success([
        {
            title: "Truyện mới",              // Tên hiển thị
            input: "[https://website.com/new](https://website.com/new)", // URL đầu vào cho script
            script: "gen.js"                  // Script xử lý
        },
        {
            title: "Truyện hot", 
            input: "[https://website.com/hot](https://website.com/hot)", 
            script: "gen.js"
        },
        {
            title: "Hoàn thành",
            input: "[https://website.com/completed](https://website.com/completed)",
            script: "gen.js"
        }
    ]);
}

// GENRE.JS - Thể loại:
function execute() {
    let response = fetch("[https://website.com/genres](https://website.com/genres)");
    if (!response.ok) return Response.error("Không thể lấy danh sách thể loại");
    
    let doc = response.html();
    const genres = [];
    
    doc.select(".genre-list a").forEach(element => {
        genres.push({
            title: element.text(),            // Tên thể loại
            input: element.attr("href"),      // URL thể loại
            script: "gen.js"                  // Script xử lý
        });
    });
    
    return Response.success(genres);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]** *(Lưu ý: Thay thế `let/const` bằng `var`)*
```javascript
// --- src/home.js ---
function execute() {
    return Response.success([
        { title: "Truyện Hot", input: "/danh-sach/truyen-hot", script: "gen.js" },
        { title: "Mới Cập Nhật", input: "/danh-sach/truyen-moi", script: "gen.js" },
        { title: "Truyện Full", input: "/danh-sach/truyen-full", script: "gen.js" }
    ]);
}

// --- src/genre.js ---
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

---

### 3. Tìm kiếm (`search.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
// SEARCH.JS - Tìm kiếm:
function execute(key, page) {
    // NOTE: page phải là string, không phải number
    if (!page) page = "1";
    
    // Encode search key
    let encodedKey = encodeURIComponent(key);
    
    // Cho trang GBK encoding (tiếng Trung)
    var gbkEncode = function(s) {
        load('gbk.js');  // Tải từ [https://moleys.4everland.store/cdn/gbk.js](https://moleys.4everland.store/cdn/gbk.js)
        return GBK.encode(s);
    };
    
    let url = `https://website.com/search?q=${encodedKey}&page=${page}`;
    let response = fetch(url);
    
    if (!response.ok) return Response.error("Tìm kiếm thất bại");
    
    let doc = response.html();
    const data = [];
    
    doc.select(".search-result .item").forEach(element => {
        let name = element.select(".title").text();
        let link = element.select("a").attr("href");
        let cover = element.select("img").attr("src");
        
        // Xử lý URL
        if (cover && cover.startsWith("//")) cover = "https:" + cover;
        if (link && !link.startsWith("http")) link = "[https://website.com](https://website.com)" + link;
        
        data.push({
            name: name,
            link: link,
            cover: cover,
            description: element.select(".description").text(),
            host: "[https://website.com](https://website.com)"
        });
    });
    
    // Tìm trang tiếp theo
    let nextPage = null;
    let nextLink = doc.select("a.next").attr("href");
    if (nextLink || data.length > 0) {
        nextPage = (parseInt(page) + 1).toString();  // NOTE: next phải là string
    }
    
    return Response.success(data, nextPage);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(key, page) {
    if (!page) page = '1';
    var fetchUrl = "[https://truyenmoiyy.com/tim-kiem?tu-khoa=](https://truyenmoiyy.com/tim-kiem?tu-khoa=)" + key + "&trang=" + page;
    var res = fetch(fetchUrl);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];
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
    var hasNext = doc.select(".pagination li a[rel=next]").length > 0;
    if (hasNext) return Response.success(data, (parseInt(page) + 1).toString());
    return Response.success(data);
}
```

---

### 4. Danh sách truyện (`gen.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
// GEN.JS - Script tổng quát:
function execute(url, page) {
    // NOTE: page phải là string
    if (!page) page = "1";
    
    // Xử lý URL
    if (url.slice(-1) !== "/") {
        url = url + "/";
    }
    url = url.replace("m.website.com", "[www.website.com](https://www.website.com)");
    
    // Build URL với page
    let pageUrl = `${url}${page}.html`;
    // Hoặc: let pageUrl = `${url}?page=${page}`;
    
    let response = fetch(pageUrl);
    if (!response.ok) return Response.error(`Không thể tải trang ${page}`);
    
    let doc = response.html();
    const data = [];
    
    // Lấy danh sách truyện
    doc.select(".book-list .item").forEach(element => {
        let name = element.select(".title a, h3 a").text();
        let link = element.select(".title a, h3 a").attr("href");
        let cover = element.select("img").attr("src");
        let description = element.select(".description, .summary").text();
        
        // Skip nếu không có tên hoặc link
        if (!name || !link) return;
        
        // Xử lý URLs
        if (cover && cover.startsWith("//")) {
            cover = "https:" + cover;
        } else if (cover && cover.startsWith("/")) {
            cover = "[https://website.com](https://website.com)" + cover;
        }
        
        if (link && !link.startsWith("http")) {
            if (link.startsWith("/")) {
                link = "[https://website.com](https://website.com)" + link;
            } else {
                link = "[https://website.com/](https://website.com/)" + link;
            }
        }
        
        data.push({
            name: name,                       // Tên truyện (bắt buộc)
            link: link,                       // URL truyện (bắt buộc)
            cover: cover,                     // URL ảnh bìa
            description: description,         // Mô tả ngắn
            host: "[https://website.com](https://website.com)"       // Domain (tùy chọn)
        });
    });
    
    // Tìm trang tiếp theo
    let nextPage = null;
    
    // Cách 1: Từ link next
    let nextLink = doc.select("a.next, a:contains(下一页), a:contains(Next)").attr("href");
    if (nextLink) {
        let pageNum = nextLink.match(/(\d+)/);
        if (pageNum) {
            nextPage = pageNum[1];
        }
    }
    
    // Cách 2: Tự tăng page number
    if (!nextPage && data.length > 0) {
        let currentPageNum = parseInt(page);
        let maxPageElement = doc.select(".pagination .last, .last-page");
        
        if (maxPageElement.size() > 0) {
            let maxPageText = maxPageElement.text();
            let maxPageNum = parseInt(maxPageText);
            if (currentPageNum < maxPageNum) {
                nextPage = (currentPageNum + 1).toString();
            }
        } else {
            // Giả sử có trang tiếp theo nếu có data
            nextPage = (currentPageNum + 1).toString();
        }
    }
    
    return Response.success(data, nextPage);
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(url, page) {
    if (!page) page = '1';
    var fetchUrl = url;
    if (fetchUrl.indexOf("http") !== 0) fetchUrl = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + url;
    fetchUrl = fetchUrl + "/trang-" + page; // Định dạng URL riêng của trang web

    var res = fetch(fetchUrl);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    var data = [];

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

    var hasNext = doc.select(".pagination li a[rel=next]").length > 0;
    if (hasNext) return Response.success(data, (parseInt(page) + 1).toString());
    
    return Response.success(data);
}
```

---

### 5. Chi tiết truyện (`detail.js`)

**[KHUNG CODE CHUẨN - VBOOK API]**
```javascript
// DETAIL.JS - Chi tiết truyện:
function execute(url) {
    // NOTE: App tự động bỏ / cuối URL
    url = url.replace("m.website.com", "[www.website.com](https://www.website.com)");
    
    let response = fetch(url);
    if (!response.ok) return Response.error("Không thể tải trang chi tiết");
    
    let doc = response.html();
    
    // Lấy thông tin cơ bản
    let name = doc.select("h1.title, .book-title").text();
    let cover = doc.select(".cover img, .book-cover img").attr("src");
    let author = doc.select(".author, .book-author").text()
                    .replace(/Tác giả:\s*/g, "")
                    .replace(/作者:\s*/g, "");
    let description = doc.select(".description, .book-desc").html();
    let status = doc.select(".status, .book-status").text();
    
    // Xử lý cover URL
    if (cover) {
        if (cover.startsWith("//")) {
            cover = "https:" + cover;
        } else if (cover.startsWith("/")) {
            cover = "[https://website.com](https://website.com)" + cover;
        } else if (!cover.startsWith("http")) {
            cover = "[https://website.com/](https://website.com/)" + cover;
        }
    }
    
    // Xử lý trạng thái ongoing
    let ongoing = true;
    if (status.includes("Hoàn thành") || status.includes("Completed") || 
        status.includes("完结") || status.includes("End")) {
        ongoing = false;
    }
    
    // Thông tin chi tiết
    let detail = `Tác giả: ${author}<br>`;
    detail += `Trạng thái: ${status}<br>`;
    detail += doc.select(".book-info, .novel-info").text();
    
    // Lấy book ID cho các script khác
    let bookId = url.match(/\/(\d+)(?:\/|\.html|$)/);
    bookId = bookId ? bookId[1] : "";
    
    return Response.success({
        name: name,                           // Tên truyện (bắt buộc)
        cover: cover,                         // URL ảnh bìa
        host: "[https://website.com](https://website.com)",          // Domain
        author: author,                       // Tác giả
        description: description,             // Mô tả HTML
        detail: detail,                       // Thông tin chi tiết
        ongoing: ongoing,                     // true = đang ra, false = hoàn thành
        
        // Các phần tùy chọn
        genres: [{                           // Thể loại liên quan
            title: "Cùng thể loại",
            input: "[https://website.com/genre/action](https://website.com/genre/action)",
            script: "gen.js"
        }],
        suggests: [{                         // Truyện đề xuất
            title: "Truyện liên quan",
            input: `https://website.com/related/${bookId}`,
            script: "gen.js"  
        }],
        comments: [{                         // Comments
            title: "Bình luận",
            input: url,
            script: "comment.js"
        }]
    });
}
```

**[CODE THỰC TẾ - TRUYENMOIYY.COM]**
```javascript
function execute(url) {
    var res = fetch(url);
    if (!res.ok) return null;
    var doc = res.html('utf-8');
    
    var name = doc.select(".story-title").text().trim() || doc.select("h1").text().trim();
    var cover = doc.select(".book img").first().attr("src") || doc.select("meta[property='og:image']").attr("content");
    if (cover && cover.indexOf("http") !== 0) cover = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + cover;
    
    var authorEl = doc.select("[itemprop=author] [itemprop=name]").first() || doc.select(".author").first();
    var author = authorEl ? authorEl.text().replace(/Tác giả\s*:/i, "").trim() : "Đang cập nhật";
    
    var genres = [];
    var genreNames = [];
    doc.select(".info a[itemprop=genre], .info a[href*='the-loai']").forEach(function(e) {
        var gName = e.text().trim();
        var gLink = e.attr("href");
        if (gLink.indexOf("http") !== 0) gLink = "[https://truyenmoiyy.com](https://truyenmoiyy.com)" + gLink;
        genres.push({ title: gName, input: gLink, script: "gen.js" });
        genreNames.push(gName);
    });
    
    var rawText = doc.text();
    var status = rawText.indexOf("Hoàn thành") !== -1 ? "Hoàn thành" : "Đang ra";
    
    var chapters = "Đang cập nhật";
    var chapMatch = rawText.match(/Số [Cc]hương[\s:]*([\d,.]+)/);
    if (chapMatch) chapters = chapMatch[1] + " chương";
    else {
        var latestChap = doc.select("ul.l-chapters li a, .latest-chapter a").first().text();
        if (latestChap) {
            var m = latestChap.match(/Chương\s*(\d+)/i);
            if (m) chapters = m[1] + " chương";
        }
    }
    
    var detailInfo = [
        "👤 Tác giả: " + author,
        "ℹ️ Tình trạng: " + status,
        "📊 Số chương: " + chapters,
        "🏷️ Thể loại: " + (genreNames.length > 0 ? genreNames.join(", ") : "Chưa rõ")
    ];
    
    return Response.success({
        name: name,
        cover: cover,
        author: author,
        description: doc.select(".desc-text").html() || "Đang cập nhật mô tả...",
        detail: detailInfo.join("<br>"),
        genres: genres,
        host: "[https://truyenmoiyy.com](https://truyenmoiyy.com)"
    });
}
```

-
