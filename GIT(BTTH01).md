# Phân tích bố cục trang chủ tlu.edu.vn

> Phân tích theo 3 lớp song song: **Layout Box → CSS Layout Technique → Content + HTML Element**

---

## 0. Phạm vi và độ tin cậy

Bài phân tích dựa trên **nội dung văn bản, thứ tự các khối, link và ảnh** đọc được từ trang. Không có screenshot hay file CSS. Vì vậy:

- **Nhìn thấy trực tiếp (từ HTML):** thứ tự các vùng, nội dung từng vùng, số lượng mục, cấu trúc menu, link và ảnh. Các URL `_next/image` cho thấy trang viết bằng **Next.js**.
- **Suy luận từ layout:** mọi chỗ ghi Flex, Grid, tỷ lệ cột, responsive. Đây là cách **thiết kế lại** theo tư duy bố cục, chưa phải CSS thật của trường. Muốn xác nhận: mở DevTools (F12), chọn phần tử và xem `display`.
- Một số khối xuất hiện **hai lần** trong HTML (menu, danh sách tin). Khả năng cao là một bản cho desktop và một bản cho mobile (drawer, carousel), hoặc phần tử nhân đôi để chạy vòng lặp (marquee). Cần kiểm tra để xác nhận.

---

## 1. Bản đồ bố cục toàn trang

```text
page
├── header                ← logo + tìm kiếm + ngôn ngữ + menu 8 mục
├── hero                  ← slider ảnh/video (4 ảnh, 2 video, 1 ảnh hội nghị)
├── main
│   ├── news-block        ← Tin Tức (carousel) + Thông Báo (list) + Sự Kiện (list)
│   ├── statistics        ← "Những Con Số Ấn Tượng" (5 số liệu) + ảnh lãnh đạo
│   ├── highlights        ← 4 khối: Tuyển Sinh | Đào Tạo | Sinh Viên | Cựu Sinh Viên
│   ├── units             ← "Các đơn vị đào tạo, KHCN" (11 đơn vị + mô tả)
│   ├── international     ← Hợp Tác Quốc Tế
│   ├── research          ← Nghiên Cứu
│   ├── partners          ← Mạng Lưới Đối Tác
│   ├── community         ← Vì Cộng Đồng
│   └── gallery           ← Khoảnh Khắc TLU (~14 ảnh)
└── footer                ← liên hệ + 3 cột link + bản đồ + bản quyền
```

| Vùng                    | Vai trò                     | Quan hệ cha    | Container căn giữa?                    |
| ----------------------- | --------------------------- | -------------- | -------------------------------------- |
| header                  | Nhận diện và điều hướng     | con của `page` | Có (nội dung căn giữa, nền full-width) |
| hero                    | Gây ấn tượng, thông báo lớn | con của `page` | Thường full-width                      |
| từng section trong main | Mỗi section là một chủ đề   | con của `main` | Có, `max-width` khoảng 1200px          |
| footer                  | Liên hệ, link nhanh         | con của `page` | Có                                     |

**Pattern lặp lại:** từ `news-block` đến `community` đều theo dạng *tiêu đề section + danh sách thẻ tin*. Nhận ra pattern này giúp viết một lần, dùng lại nhiều lần.

---

## 2. Phân tích từng component (3 lớp song song)

### 2.1. Header

**A. Layout Box**

```text
page
└── header
    └── header-container
        ├── brand           (logo)
        ├── main-nav        (menu 8 mục)
        └── header-tools
            ├── search-button
            └── language-switch  (VN | EN + cờ)
```

**B. CSS**

```css
.header-container { display: flex; align-items: center; justify-content: space-between; gap: 24px; }
.header-tools     { display: flex; align-items: center; gap: 12px; }
```

- Flex Container: `header-container`, `header-tools`.
- Flex Items: `brand`, `main-nav`, `header-tools`; `search-button`, `language-switch`.
- Vì sao Flex: các hộp con đứng **ngang một hàng**, cần căn giữa theo trục dọc, dãn cách đều hai đầu.
- **Nested Flexbox:** `header-container` (flex) chứa `header-tools` (flex).

**C. Nội dung + HTML**

| Nội dung | Vai trò | HTML |
|---|---|---|
| Logo trường | Thương hiệu, về trang chủ | `a` bọc `img` |
| "Tìm kiếm" | Mở ô tìm kiếm | `button` (hành động, không điều hướng) |
| VN / EN + cờ | Đổi ngôn ngữ | `button` hoặc `a`, kèm `img` cờ có `alt` |

**Chỉ dùng div:**

```html
<div class="header">
  <div class="header-container">
    <div class="brand">Logo</div>
    <div class="main-nav">...</div>
    <div class="header-tools">
      <div class="search-button">Tìm kiếm</div>
      <div class="language-switch">VN | EN</div>
    </div>
  </div>
</div>
```

**Semantic:**

```html
<header class="header">
  <div class="header-container">
    <a class="brand" href="/"><img src="logo.webp" alt="Trường Đại học Thủy lợi"></a>
    <nav class="main-nav">...</nav>
    <div class="header-tools">
      <button type="button">Tìm kiếm</button>
      <div class="language-switch">...</div>
    </div>
  </div>
</header>
```

**Kết luận:** Layout: `brand + nav + tools`. CSS: Flexbox hàng, `space-between`. HTML: `header > a/img, nav, button`.

---

### 2.2. Navigation (menu 8 mục + dropdown)

**Nhìn thấy trực tiếp:** 8 mục cấp 1 (GIỚI THIỆU, TIN TỨC, TUYỂN SINH, ĐÀO TẠO, NGHIÊN CỨU, HỢP TÁC QUỐC TẾ, SINH VIÊN, eTLU). Mỗi mục có danh sách con 3 đến 11 link. Có **bản menu thứ hai** với thêm mục "Tất cả ...", khả năng là menu mobile dạng drawer.

**A. Layout Box**

```text
main-nav
└── nav-list
    ├── nav-item
    │   ├── nav-link            (GIỚI THIỆU)
    │   └── submenu             (ẩn, hiện khi hover/click)
    │       ├── submenu-item
    │       └── submenu-item
    ├── nav-item ...
```

**B. CSS**

```css
.nav-list { display: flex; gap: 8px; }          /* 8 nav-item đứng ngang */
.nav-item { position: relative; }                /* làm mốc cho submenu */
.submenu  { position: absolute; top: 100%; left: 0; }   /* nổi xuống dưới */
.submenu-item { display: block; }                /* link con xếp dọc: Normal Flow */
```

- **Flex Container:** `nav-list`. **Flex Items:** `nav-item`.
- **Position:** `nav-item` là `relative` (mốc), `submenu` là `absolute` (nổi lên trên nội dung phía dưới).
- **KHÔNG cần Flex** cho `submenu`: các link con vốn xếp dọc theo Normal Flow.
- Dropdown ẩn/hiện cần JS hoặc `:hover` → "Có khả năng là dropdown hover/click, cần kiểm tra interaction để xác nhận."

**C. Nội dung + HTML**

| Nội dung | HTML |
|---|---|
| Vùng menu | `nav` |
| Danh sách mục | `ul` > `li` |
| Mục có link (GIỚI THIỆU → /gioi-thieu) | `a` |
| Menu con | `ul` lồng trong `li` |
| Mục eTLU (không thấy link) | có thể là `button` mở dropdown |

**Chỉ dùng div:**

```html
<div class="main-nav">
  <div class="nav-list">
    <div class="nav-item">
      <div class="nav-link">GIỚI THIỆU</div>
      <div class="submenu">
        <div class="submenu-item">Chiến lược phát triển</div>
        <div class="submenu-item">Cơ cấu tổ chức</div>
      </div>
    </div>
  </div>
</div>
```

**Semantic:**

```html
<nav aria-label="Menu chính">
  <ul class="nav-list">
    <li class="nav-item">
      <a href="/gioi-thieu">GIỚI THIỆU</a>
      <ul class="submenu">
        <li><a href="/su-mang-muc-tieu-chien-luoc">Chiến lược phát triển</a></li>
        <li><a href="/co-cau-to-chuc">Cơ cấu tổ chức</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

**Kết luận:** Flex (hàng) + Position (dropdown) + Block (submenu). HTML: `nav > ul > li > a + ul`.

---

### 2.3. Hero (slider)

**Nhìn thấy trực tiếp:** chuỗi 4 ảnh, 2 video `.mp4` và 1 ảnh hội nghị liên tiếp, không có chữ mô tả. **Có khả năng là slider/carousel hoặc video banner**, cần kiểm tra để xác nhận.

**A. Layout Box**

```text
hero
└── hero-slider
    ├── hero-track
    │   ├── hero-slide (ảnh/video)
    │   └── hero-slide ...
    ├── hero-prev / hero-next      (nếu có)
    └── hero-dots                  (nếu có)
```

**B. CSS**

```css
.hero-slider { position: relative; overflow: hidden; }
.hero-track  { display: flex; }                  /* các slide xếp ngang liên tiếp */
.hero-slide  { flex: 0 0 100%; }                 /* mỗi slide chiếm đúng 1 khung nhìn */
.hero-prev, .hero-next { position: absolute; top: 50%; }
```

- Cách phổ biến: `track` là Flex Container, mỗi slide là Flex Item rộng 100%, rồi dịch bằng `transform: translateX(...)`.
- Nút điều hướng nổi lên trên ảnh → Position absolute.

**C. HTML:** ảnh → `img` (có `alt`), video → `video` + `source`, nút → `button`.

**Semantic:**

```html
<section class="hero" aria-label="Banner">
  <div class="hero-track">
    <img class="hero-slide" src="..." alt="...">
    <video class="hero-slide" src="1.mp4" autoplay muted loop></video>
  </div>
  <button class="hero-prev" aria-label="Trước">‹</button>
  <button class="hero-next" aria-label="Sau">›</button>
</section>
```

**Kết luận:** Layout `slider > track > slide`. CSS: Flex + Position. HTML: `section > img/video + button`.

---

### 2.4. Tin Tức + Thông Báo + Sự Kiện

**Nhìn thấy trực tiếp:**

- **Tin Tức:** 7 thẻ tin (ảnh, nhãn danh mục như "Tin tức chung", tiêu đề, mô tả ngắn, ngày) và hai nút `‹ ›`. Thẻ đầu có nhãn "Hot".
- **Thông Báo:** danh sách 6 dòng, mỗi dòng gồm tiêu đề link và ngày.
- **Sự Kiện:** 8 mục, mỗi mục có ảnh, ngày/tháng (ví dụ "19 Tháng 11") và tiêu đề.

**A. Layout Box**

```text
news-block
├── news-carousel                 (Tin Tức)
│   ├── section-header            (tiêu đề + link "Tin Tức")
│   ├── news-track
│   │   └── news-card × 7
│   │       ├── news-thumb
│   │       └── news-body
│   │           ├── news-category
│   │           ├── news-title
│   │           ├── news-excerpt
│   │           └── news-date
│   └── carousel-controls (‹ ›)
├── notice-list                   (Thông Báo)
│   └── notice-item × 6
│       ├── notice-title
│       └── notice-date
└── event-list                    (Sự Kiện)
    └── event-item × 8
        ├── event-thumb
        │   └── event-date        (ngày + tháng đè lên ảnh)
        └── event-title
```

**B. CSS** (cột ngoài + từng thẻ)

```css
.news-block   { display: flex; gap: 24px; }       /* ngang: carousel | notice | event */
.news-carousel{ flex: 2; }                         /* tỷ lệ gợi ý 2 : 1 : 1, cần kiểm tra */
.notice-list, .event-list { flex: 1; }

.news-track { display: flex; gap: 16px; overflow-x: auto; }
.news-card  { flex: 0 0 280px; display: flex; flex-direction: column; }  /* ảnh trên, chữ dưới */

.notice-item { display: flex; justify-content: space-between; gap: 12px; } /* tiêu đề trái, ngày phải */
.event-item  { display: flex; gap: 12px; align-items: center; }            /* ảnh trái, tên phải */
.event-thumb { position: relative; }               /* làm mốc cho ngày đè lên ảnh */
.event-date  { position: absolute; left: 0; bottom: 0; }
```

- **Nested Flexbox ba cấp:** `news-block` → `news-track` → `news-card`.
- **Flex-wrap:** không dùng cho `news-track` (carousel cuộn ngang), nhưng dùng khi chuyển sang lưới trên mobile.
- **KHÔNG cần Flex:** `notice-list` và `event-list` là danh sách xếp dọc → Normal Flow; mỗi `*-item` mới cần Flex.
- Tỷ lệ cột 2 : 1 : 1 chỉ là đề xuất, cần kiểm tra DevTools.

**C. Nội dung + HTML**

| Nội dung | Vai trò | HTML |
|---|---|---|
| Mỗi thẻ tin | bài viết độc lập | `article` |
| Ảnh đại diện | hình minh họa, bấm vào được | `a` > `img` |
| Nhãn "Tin tức chung", "Hot" | phân loại | `span` |
| Tiêu đề tin | tiêu đề | `h3` > `a` |
| Mô tả | đoạn văn | `p` |
| Ngày 19/10/2026 | thời gian | `time datetime="2026-10-19"` |
| Nút ‹ › | thao tác | `button` |
| Danh sách thông báo | danh sách | `ul` > `li` |
| Ngày + tháng trên ảnh sự kiện | thời gian | `time` |

**Chỉ dùng div:**

```html
<div class="news-card">
  <div class="news-thumb">[ảnh]</div>
  <div class="news-body">
    <div class="news-category">Tin tức chung</div>
    <div class="news-title">Tiêu đề</div>
    <div class="news-excerpt">Căn cứ nhu cầu...</div>
    <div class="news-date">19/10/2026</div>
  </div>
</div>
```

**Semantic:**

```html
<section class="news-carousel">
  <h2><a href="/tin-tuc-thong-bao">Tin Tức</a></h2>
  <article class="news-card">
    <a href="..."><img src="..." alt="..."></a>
    <div class="news-body">
      <span class="news-category">Tin tức chung</span>
      <h3><a href="...">Thông báo Tuyển dụng giảng viên...</a></h3>
      <p>Căn cứ nhu cầu giảng dạy...</p>
      <time datetime="2026-10-19">19/10/2026</time>
    </div>
  </article>
</section>

<section class="notice-list">
  <h2><a href="/thong-bao">Thông Báo</a></h2>
  <ul>
    <li><a href="...">Thông báo thu học phí học kỳ 1...</a> <time datetime="2026-11-11">11/11/2026</time></li>
  </ul>
</section>
```

**Kết luận:** `news-card` = Flex cột (ảnh + body). `news-block` = Flex hàng 3 cột. HTML: `article > a/img, span, h3/a, p, time`.

---

### 2.5. Statistics ("Những Con Số Ấn Tượng")

**Nhìn thấy trực tiếp:** 5 nhãn (Người học, Đối tác trong & ngoài nước, Sinh viên tốt nghiệp có việc làm, Ngành học, Giảng viên Tiến sĩ). **Con số không có trong nội dung đọc được**, có khả năng là bộ đếm chạy bằng JS khi cuộn tới. Kèm một ảnh chân dung (alt: "Trang Chủ", tên file `GS-Nguyen-Trung-Viet`), khả năng là ảnh lãnh đạo nhà trường, cần kiểm tra.

**A. Layout Box**

```text
statistics
├── statistics-header     (h2)
└── statistics-list
    └── statistics-item × 5
        ├── statistics-number
        └── statistics-label
```

**B. CSS**

```css
.statistics-list { display: flex; justify-content: space-between; flex-wrap: wrap; gap: 24px; }
.statistics-item { flex: 1; text-align: center; }
```

- Cần chia đều 5 cột → `flex: 1` cho mỗi item.
- Nếu muốn 5 cột thẳng hàng tuyệt đối, **Grid** (`repeat(5, 1fr)`) cũng phù hợp; Flex đủ dùng vì chỉ có một hàng.

**C. HTML:** tiêu đề → `h2`; danh sách số liệu → `dl` (cặp nhãn–giá trị) hoặc `ul > li`.

**Semantic:**

```html
<section class="statistics">
  <h2>Những Con Số Ấn Tượng</h2>
  <dl class="statistics-list">
    <div class="statistics-item">
      <dt>Người học</dt>
      <dd>...</dd>
    </div>
  </dl>
</section>
```

**Kết luận:** `list > item × 5`; Flex hàng + `flex: 1` (hoặc Grid 5 cột); `h2 + dl/dt/dd`.

---

### 2.6. Highlights (Tuyển Sinh | Đào Tạo | Sinh Viên | Cựu Sinh Viên)

**Nhìn thấy trực tiếp:**

- **Tuyển Sinh:** ảnh nền, đoạn mô tả, 3 mục (Đại học, Thạc sĩ, Tiến sĩ) và nút "Đăng ký ngay".
- **Đào Tạo, Sinh Viên, Cựu Sinh Viên:** mỗi khối một bài nổi bật gồm ảnh, tiêu đề link, đoạn trích.

**A. Layout Box**

```text
highlights
├── highlight-admission        (Tuyển Sinh: lớn hơn)
│   ├── highlight-image
│   └── highlight-body
│       ├── h2, p
│       ├── admission-levels (ul × 3)
│       └── cta-button
├── highlight-card             (Đào Tạo)
├── highlight-card             (Sinh Viên)
└── highlight-card             (Cựu Sinh Viên)
```

**B. CSS**

```css
.highlights { display: flex; flex-wrap: wrap; gap: 24px; }
.highlight-admission { flex: 2; }            /* khối Tuyển Sinh rộng gấp đôi, tỷ lệ gợi ý */
.highlight-card      { flex: 1; display: flex; flex-direction: column; }
```

- **Nếu 4 khối bằng nhau** → Grid `repeat(4, 1fr)` hợp hơn; **nếu Tuyển Sinh lớn hơn** → Flex với tỷ lệ. Cần kiểm tra ảnh thật để chọn.
- `highlight-body` bên trong: Normal Flow (tiêu đề → đoạn → danh sách → nút xếp dọc), **không cần Flex**.

**C. HTML**

| Nội dung | HTML |
|---|---|
| Tên khối "Tuyển Sinh" | `h2` > `a` |
| Mô tả | `p` |
| Đại học / Thạc sĩ / Tiến sĩ | `ul > li > a` |
| "Đăng ký ngay" | `a` (chuyển trang → là link, tạo kiểu giống nút) |
| Bài nổi bật | `article` với `img`, `h3 > a`, `p` |

**Semantic:**

```html
<section class="highlight-admission">
  <h2><a href="https://ts.tlu.edu.vn/">Tuyển Sinh</a></h2>
  <p>Cung cấp thông tin tuyển sinh...</p>
  <ul>
    <li><a href="...">Đại học</a></li>
    <li><a href="...">Thạc sĩ</a></li>
    <li><a href="...">Tiến sĩ</a></li>
  </ul>
  <a class="cta-button" href="https://ts.tlu.edu.vn/">Đăng ký ngay</a>
</section>
```

> **Lưu ý link vs button:** "Đăng ký ngay" dẫn sang trang `ts.tlu.edu.vn` nên là `a`, không phải `button`. `button` dành cho hành động trên trang (mở menu, gửi form, đổi slide).

---

### 2.7. Units ("Các đơn vị đào tạo, KHCN")

**Nhìn thấy trực tiếp:** 11 đơn vị (Khoa Kinh tế và Quản lý, Hóa và Môi trường, Kế toán và Kinh doanh, Phân hiệu, CNTT, Cơ khí, Luật và Lý luận chính trị, Điện - Điện tử, Công trình, Kỹ thuật Tài nguyên nước, Trung tâm Đào tạo Quốc tế). Mỗi đơn vị có link riêng. Chỉ **một** đơn vị (Khoa Kinh tế và Quản lý) hiện kèm đoạn giới thiệu và link "Khám phá". **Có khả năng là tab/accordion**: bấm đơn vị nào thì panel mô tả đổi theo, cần kiểm tra interaction.

**A. Layout Box**

```text
units
├── units-header (h2)
└── units-layout
    ├── units-tabs
    │   └── unit-tab × 11
    └── unit-panel
        ├── unit-description
        └── unit-link
```

**B. CSS**

```css
.units-layout { display: flex; gap: 32px; }
.units-tabs   { flex: 1; }          /* danh sách tab, xếp dọc: Normal Flow */
.unit-panel   { flex: 2; }          /* panel mô tả; tỷ lệ 1 : 2 là gợi ý */
```

- Tỷ lệ 1 : 2: cột trái chiếm một phần ba, cột phải hai phần ba.
- Nếu tab nằm ngang và tự xuống dòng → `display: flex; flex-wrap: wrap`.

**C. HTML:** tab → `button role="tab"` (đổi panel tại chỗ, không chuyển trang); tên khoa kèm link riêng → `a`; mô tả → `p`.

**Kết luận:** `tabs + panel`; Flex hàng, `flex: 1` và `flex: 2`; `h2, ul/li, button/a, p`.

---

### 2.8. International / Research / Community (3 khối cùng pattern)

**Nhìn thấy trực tiếp:** mỗi khối gồm tiêu đề link ("Hợp Tác Quốc Tế", "Nghiên Cứu", "Vì Cộng Đồng") và danh sách bài, bài đầu có ảnh, các bài sau chỉ có chữ. Nội dung bị lặp lại hai lần trong HTML (khả năng là hai kiểu hiển thị desktop/mobile hoặc carousel).

**A. Layout Box**

```text
topic-section (international / research / community)
├── section-header (h2 + link)
└── topic-layout
    ├── topic-featured            (bài đầu: ảnh + tiêu đề + ngày)
    └── topic-list
        └── topic-item × n        (nhãn + tiêu đề + ngày)
```

**B. CSS**

```css
.topic-layout   { display: flex; gap: 24px; }
.topic-featured { flex: 1; }
.topic-list     { flex: 1; }               /* hoặc 2 : 1, cần kiểm tra */
.topic-item     { display: block; }        /* danh sách xếp dọc: không cần Flex */
```

**C. HTML:** `section` > `h2`; bài nổi bật là `article` (`img`, `h3 > a`, `time`); danh sách là `ul > li`; nhãn "Hợp tác quốc tế" là `span`.

**Semantic:**

```html
<section class="international">
  <h2><a href="/doi-ngoai">Hợp Tác Quốc Tế</a></h2>
  <article class="topic-featured">
    <a href="..."><img src="..." alt="..."></a>
    <h3><a href="...">Thứ trưởng Bộ Thủy lợi Trung Quốc...</a></h3>
    <time datetime="2026-11-19">19/11/2026</time>
  </article>
  <ul class="topic-list">
    <li><a href="...">...</a> <time datetime="2026-11-18">18/11/2026</time></li>
  </ul>
</section>
```

**Kết luận:** Một component dùng lại 3 lần (`topic-section`). Đặt tên theo vai trò, không theo vị trí.

---

### 2.9. Partners (Mạng Lưới Đối Tác)

**Nhìn thấy trực tiếp:** chỉ có tiêu đề và một đoạn giới thiệu; **phần logo đối tác không có trong nội dung đọc được**, nhiều khả năng là băng logo chạy ngang (marquee) hoặc dựng bằng canvas/hình, cần kiểm tra.

**B. CSS đề xuất:**

```css
.partner-logos { display: flex; gap: 32px; align-items: center; overflow: hidden; }
```

Mỗi logo là một Flex Item.

**C. HTML:** `section > h2 + p + ul > li > img` (mỗi logo có `alt` tên đối tác).

---

### 2.10. Gallery (Khoảnh Khắc TLU)

**Nhìn thấy trực tiếp:** khoảng 14 ảnh liên tiếp, alt là tiêu đề bài viết, không thấy chú thích riêng.

**Đánh giá Flex vs Grid:**

- Nếu các ảnh **cùng kích thước** xếp 3 × N hoặc 4 × N → **Grid phù hợp hơn**:

```css
.gallery-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; }
```

- Nếu ảnh **cao thấp khác nhau** (masonry) hoặc là một băng chạy ngang → Flex hoặc `columns`.
- Chưa thấy bố cục thật nên đây là phương án đề xuất.

**C. HTML:** `section > h2 + ul/div > figure > img` (thêm `figcaption` nếu có chú thích).

---

### 2.11. Footer

**Nhìn thấy trực tiếp:**

- 4 link mạng xã hội (Facebook, YouTube, Instagram, TikTok).
- Logo, **địa chỉ** (175 Tây Sơn, P. Kim Liên, Hà Nội), email, điện thoại.
- 3 cột link: *Giới thiệu* (4 link), *Truy cập nhanh* (5 link), *Theo dõi TLU* (4 link).
- "Trường Đại học Thủy lợi" + "Tải bản đồ tương tác" (khả năng là bản đồ nhúng tải lười, cần kiểm tra).
- Bản quyền và đơn vị phát triển.

**A. Layout Box**

```text
footer
└── footer-container
    ├── footer-top
    │   ├── social-links
    │   └── footer-brand (logo)
    ├── footer-columns
    │   ├── footer-contact     (địa chỉ, email, điện thoại)
    │   ├── footer-col (Giới thiệu)
    │   ├── footer-col (Truy cập nhanh)
    │   ├── footer-col (Theo dõi TLU)
    │   └── footer-map
    └── footer-bottom (bản quyền)
```

**B. CSS**

```css
.footer-columns { display: flex; flex-wrap: wrap; justify-content: space-between; gap: 32px; }
.footer-col     { flex: 1 1 200px; }       /* mỗi cột tối thiểu 200px rồi tự xuống dòng */
.footer-contact { flex: 2 1 300px; }       /* cột liên hệ rộng hơn, tỷ lệ gợi ý */
.social-links   { display: flex; gap: 12px; }
```

- `footer-bottom`: một dòng chữ, Normal Flow, **không cần Flex**.

**C. HTML**

| Nội dung | HTML |
|---|---|
| Vùng chân trang | `footer` |
| Địa chỉ trường | `address` |
| Email, điện thoại | `a href="mailto:..."`, `a href="tel:..."` |
| Tiêu đề cột "Giới thiệu" | `h3` |
| Danh sách link | `ul > li > a` |
| Icon mạng xã hội | `a` > `img/svg` (có `aria-label`) |
| Bản quyền | `small` hoặc `p` |

**Semantic:**

```html
<footer class="footer">
  <div class="footer-columns">
    <address class="footer-contact">
      175 Tây Sơn - P. Kim Liên - Hà Nội<br>
      <a href="tel:02438522201">(024) 38522201</a>
    </address>
    <nav class="footer-col" aria-label="Giới thiệu">
      <h3>Giới thiệu</h3>
      <ul><li><a href="/gioi-thieu">Tổng quan</a></li></ul>
    </nav>
  </div>
  <p class="footer-bottom"><small>Bản quyền © 2026 Trường Đại học Thủy lợi</small></p>
</footer>
```

---

## 3. Bản đồ Flexbox toàn trang

```text
PAGE
│
├── HEADER
│   └── header-container          [FLEX row, space-between]
│       └── header-tools          [FLEX row]            ← nested
│
├── NAV
│   └── nav-list                  [FLEX row]
│       └── nav-item              [position: relative] + submenu [absolute]
│
├── HERO
│   └── hero-track                [FLEX row, slide flex: 0 0 100%]
│
├── MAIN
│   ├── news-block                [FLEX row, 2 : 1 : 1]
│   │   ├── news-track            [FLEX row, cuộn ngang]
│   │   │   └── news-card         [FLEX column]          ← nested 3 cấp
│   │   ├── notice-item           [FLEX row, space-between]
│   │   └── event-item            [FLEX row]
│   ├── statistics-list           [FLEX row + WRAP, item flex: 1]
│   ├── highlights                [FLEX row + WRAP, 2 : 1 : 1 : 1]
│   ├── units-layout              [FLEX row, 1 : 2]
│   ├── topic-layout (×3)         [FLEX row, 1 : 1]
│   ├── partner-logos             [FLEX row, cuộn/marquee]
│   └── gallery                   [GRID — xem mục 4]
│
└── FOOTER
    ├── social-links              [FLEX row]
    └── footer-columns            [FLEX row + WRAP]
```

**Nơi KHÔNG cần Flexbox (Normal Flow):** `submenu` (link con xếp dọc), `notice-list`, `topic-list`, `footer-bottom`, `highlight-body` (tiêu đề → đoạn → danh sách → nút), `news-body`.

---

## 4. Bản đồ Grid

| Vùng | Flex hay Grid? | Lý do |
|---|---|---|
| Gallery (14 ảnh đều nhau) | **Grid** | Có cấu trúc hàng × cột rõ ràng |
| Highlights (4 khối bằng nhau) | Grid hợp hơn | `repeat(4, 1fr)`, hàng/cột thẳng nhau |
| Statistics (5 số liệu) | Flex hoặc Grid | Chỉ một hàng nên Flex đủ dùng |
| News carousel | Flex | Cuộn ngang một chiều |
| Nav, header | Flex | Một chiều, căn giữa, dãn cách |

**Quy tắc nhớ:** *Flexbox = bố cục một chiều (hàng hoặc cột). Grid = bố cục hai chiều (hàng và cột cùng lúc).*

---

## 5. Responsive dự kiến (suy luận)

| Component | Desktop | Tablet | Mobile |
|---|---|---|---|
| Header + nav | logo \| menu \| tools | logo \| ☰ | logo \| ☰ (menu thành drawer) |
| news-block | 3 cột (carousel \| thông báo \| sự kiện) | 2 cột | 1 cột xếp dọc |
| Highlights | 4 khối / hàng | 2 × 2 | 1 khối / hàng |
| Statistics | 5 / hàng | 3 + 2 | 2 / hàng |
| Units | tab trái \| panel phải | giữ | tab thành dropdown/accordion |
| Footer | 4 cột | 2 × 2 | 1 cột |

```css
.highlights { display: flex; flex-wrap: wrap; }
.highlight-card { flex: 1 1 240px; }      /* không đủ chỗ thì tự xuống dòng */

@media (max-width: 768px) {
  .news-block, .units-layout, .topic-layout { flex-direction: column; }
}
```

Việc trang có **hai bản menu và hai bản danh sách tin** (đọc được trong HTML) ủng hộ giả thuyết này: một bản cho desktop, một bản cho mobile.

---

## 6. Cây DOM toàn trang

### 6.1. Chỉ dùng div

```text
div.page
├── div.header
│   └── div.header-container
│       ├── div.brand
│       ├── div.main-nav > div.nav-list > div.nav-item × 8 > div.submenu
│       └── div.header-tools > div.search-button + div.language-switch
├── div.hero > div.hero-track > div.hero-slide × 7
├── div.main
│   ├── div.news-block
│   │   ├── div.news-carousel > div.news-track > div.news-card × 7
│   │   ├── div.notice-list > div.notice-item × 6
│   │   └── div.event-list > div.event-item × 8
│   ├── div.statistics > div.statistics-list > div.statistics-item × 5
│   ├── div.highlights > div.highlight-admission + div.highlight-card × 3
│   ├── div.units > div.units-layout > div.units-tabs + div.unit-panel
│   ├── div.international > div.topic-layout > div.topic-featured + div.topic-list
│   ├── div.research      (cùng cấu trúc)
│   ├── div.partners > div.partner-logos
│   ├── div.community     (cùng cấu trúc)
│   └── div.gallery > div.gallery-grid > div.gallery-item × 14
└── div.footer
    └── div.footer-container
        ├── div.social-links
        ├── div.footer-columns > div.footer-contact + div.footer-col × 3 + div.footer-map
        └── div.footer-bottom
```

### 6.2. Semantic HTML

```text
body
├── header.header
│   ├── a.brand > img
│   ├── nav.main-nav > ul > li > a + ul
│   └── div.header-tools > button + div.language-switch
├── main
│   ├── section.hero > img/video + button
│   ├── section.news-carousel > h2 + article × 7
│   ├── section.notice-list > h2 + ul > li
│   ├── section.event-list > h2 + ul > li > a > img + time
│   ├── section.statistics > h2 + dl > div > dt + dd
│   ├── section.highlights > section/article × 4
│   ├── section.units > h2 + ul + article
│   ├── section.international > h2 + article + ul
│   ├── section.research
│   ├── section.partners > h2 + p + ul > li > img
│   ├── section.community
│   └── section.gallery > h2 + figure > img × 14
└── footer > address + nav × 3 + p > small
```

---

## 7. Bảng tổng hợp

| Component | Layout Box | Layout Technique | Flex Container? | Flex Items | Nội dung | HTML semantic |
|---|---|---|---|---|---|---|
| Header | header-container | Flexbox | Có | brand, main-nav, header-tools | Logo, menu, tìm kiếm, ngôn ngữ | `header`, `a`, `img`, `button` |
| Navigation | nav-list | Flex + Position | Có | nav-item | Menu 8 mục + dropdown | `nav`, `ul`, `li`, `a` |
| Hero | hero-track | Flex + Position | Có | hero-slide | Ảnh/video banner | `section`, `img`, `video`, `button` |
| News block | news-block | Flexbox (2:1:1) | Có | carousel, notice, event | Tin, thông báo, sự kiện | `section`, `article`, `ul`, `time` |
| News card | news-card | Flexbox column | Có | thumb, body | Bài viết | `article`, `img`, `h3`, `p`, `time` |
| Notice item | notice-item | Flexbox row | Có | title, date | Thông báo | `li`, `a`, `time` |
| Statistics | statistics-list | Flex (hoặc Grid) | Có | statistics-item | Số liệu | `h2`, `dl`, `dt`, `dd` |
| Highlights | highlights | Flex+wrap (hoặc Grid) | Có | highlight-card | 4 khối chủ đề | `section`, `article`, `ul`, `a` |
| Units | units-layout | Flexbox (1:2) | Có | tabs, panel | Đơn vị đào tạo | `ul`, `button`/`a`, `p` |
| Topic section ×3 | topic-layout | Flexbox | Có | featured, list | Bài theo chủ đề | `section`, `article`, `ul`, `time` |
| Partners | partner-logos | Flexbox / marquee | Có | logo | Logo đối tác | `ul`, `li`, `img` |
| Gallery | gallery-grid | **Grid** | Không | – | Ảnh | `figure`, `img`, `figcaption` |
| Footer columns | footer-columns | Flex + wrap | Có | footer-col | Liên hệ, link | `footer`, `address`, `nav`, `h3`, `ul` |

---

## 8. Checklist tư duy cho sinh viên

```text
Nhìn thấy một vùng giao diện
        ↓
Khoanh hộp  →  Đặt tên theo VAI TRÒ  →  Xác định cha – con
        ↓
Các con có đứng ngang hàng không?
        ├── Có → Flexbox
        └── Không
             ↓
        Có cấu trúc hàng × cột không?
        ├── Có → Grid
        └── Không → Normal Flow
             ↓
        Có phần tử nổi chồng lên phần tử khác?
        └── Có → Position (cha relative, con absolute)
        ↓
Xác định kiểu nội dung  →  Chọn HTML semantic
        ↓
Click đi nơi khác? → <a>   |   Làm gì đó tại chỗ? → <button>
```

**Nguyên tắc đặt tên (đã dùng trong bài):** `news-card`, `nav-item`, `footer-col`, `topic-layout`; dùng lại khi pattern lặp (`*-list`, `*-item`, `*-header`). Tránh: `box1`, `left-area`, `big-box`.

---

## 9. Các lỗi tư duy phổ biến

1. **Dùng Flex cho mọi thứ.** Danh sách xếp dọc (submenu, notice-list) chỉ cần Normal Flow.
2. **Flex Container nhầm cấp.** `display: flex` đặt trên cha thì **con trực tiếp** mới thành Flex Item, cháu thì không. Cần Flex thêm một cấp (nested).
3. **Chọn Flex cho lưới hàng × cột.** Gallery và 4 khối đều nhau nên dùng Grid.
4. **Dùng `div` thay mọi thẻ.** Thẻ tin là `article`, menu là `nav`, ngày là `time`, địa chỉ là `address`.
5. **Nhầm `a` và `button`.** "Đăng ký ngay" dẫn sang trang khác nên là `a`; nút ‹ › của carousel là `button`.
6. **Quên `alt` cho `img`.** Ảnh tin cần `alt` mô tả, ảnh trang trí có thể để `alt=""`.
7. **Đặt tên theo hình dạng.** `left-area`, `blue-box` hỏng khi đổi giao diện; dùng tên theo vai trò.
8. **Kết luận từ ảnh quá sớm.** Slider, tab, marquee chỉ nhìn tĩnh thì khó chắc; phải kiểm tra HTML và interaction.

---

## Việc nên làm tiếp để xác nhận

Mở https://tlu.edu.vn, nhấn **F12**, dùng công cụ chọn phần tử, rồi xem tab *Computed* hoặc *Styles* để kiểm tra `display`, `flex`, `grid-template-columns` ở các khối: `news-block`, `statistics`, `units`, `footer`, `gallery`. Nếu có screenshot full-page hoặc file HTML đã render, các phần "gợi ý" (tỷ lệ cột, Flex hay Grid) có thể được sửa thành kết luận chắc chắn.
