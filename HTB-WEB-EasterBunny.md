[HTB] Web - EasterBunny Write Up!
## By @ndkhai
***
* Link Challenge: `https://app.hackthebox.com/challenges/easterbunny`
* Challenge Description: `It's that time of the year again! Write a letter to the Easter bunny and make your wish come true! But be careful what you wish for because the Easter bunny's helpers are watching!`
* Necessary files to play the challenge: `Source Code`
***
## Sơ lược tính năng của ứng dụng
1. Giao diện chính của web:

 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/a90bf878-f7ca-4bcc-9078-f447450f0a87)

2. Ứng dụng cho phép submit thư mới lên hệ thống, gọi vào link `/submit`. Nội dung thư mới sẽ được insert vào DB với id tăng dần.

3. Ứng dụng cho phép đọc nội dung của các thư hiện có trên hệ thống (link URL: `/letters?id=x`)

## Source code

Mục tiêu của Challenge là đọc được nội dung của message có `id=3`. Nội dung của các message đã được input sẵn trong Database:
```sql
INSERT INTO messages (id, message, hidden) VALUES
              (1, "Dear Easter Bunny,\nPlease could I have the biggest easter egg you have?\n\nThank you\nGeorge", 0),
              (2, "Dear Easter Bunny,\nCould I have 3 chocolate bars and 2 easter eggs please!\nYours sincerly, Katie", 0),
              (3, "Dear Easter Bunny, Santa's better than you! HTB{f4k3_fl4g_f0r_t3st1ng}", 1),
              (4, "Hello Easter Bunny,\n\nCan I have a PlayStation 5 and a chocolate chick??", 0),
              (5, "Dear Ester Bunny,\nOne chocolate and marshmallow bunny please\n\nLove from Milly", 0),
              (6, "Dear Easter Bunny,\n\nHow are you? Im fine please may I have 31 chocolate bunnies\n\nThank you\nBeth", 0);
```

Message **id=3** có trường **hidden=1**, do vậy, không thể view được nội dung của message này trên giao diện web theo cách thông thường, giống như các message khác. Đoạn code xử lý phần này như sau:

```javascript
router.get("/message/:id", async (req, res) => {
    try {
        const { id } = req.params;
        const { count } = await db.getMessageCount();
        const message = await db.getMessage(id);

        if (!message) return res.status(404).send({
            error: "Can't find this note!",
            count: count
        });

        if (message.hidden && !isAdmin(req))
            return res.status(401).send({
                error: "Sorry, this letter has been hidden by the easter bunny's helpers!",
                count: count
            });
```

=> Như vậy các message có trường **hidden=1** và **!isAdmin(req))** thì sẽ không hiển thị nội dung ra bên ngoài trình duyệt. Để đạt được mục đích, ta phải tìm cách để request đọc message id=3 phải được hiểu là xuất phát từ **admin**.




![image](https://github.com/nguyenkhai98/writeup/assets/51147179/4931f70f-d3b8-4d38-9e3c-96ae64808372)

Chi tiết phần code xử lý request đến URL `/letters` (dòng 17->21) và `/submit` (dòng 23->45) trong file `routes/routes.js` như hình bên trên.

```javascript
router.get("/letters", (req, res) => {
    return res.render("viewletters.html", {
        cdn: `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/`,
    });
});
```

=> Khi `GET /letters` thì ứng dụng sẽ trả về nội dung từ `viewletters.html` với giá trị `cdn` như trên. Okay, vậy giá trị **cdn** có vai trò gì?
Cùng check nội dung file **viewletters.html**:

```html
{% extends "base.html" %}
{% block content %}
  <h1 class="title" style="margin: 0">Viewing letter #<span id="letter-id">1</span></h1>
  <h2 class="title" id="error-message" style="visibility: hidden;">&nbsp;</h2>
  {% include "letter.html" %}
  
  <div class="letter letter-small">
    <div class="letter-inner letter-inner-small">
        <a href="/">Write New Letter</a>
    </div>
  </div>
  
  <div id="previous" class="sign-post">
    <div class="sign-post-text">
      <a href="#">View previous<br><br>letter</a>
    </div>
  </div>

  <div id="next" class="sign-post flipped">
    <div class="sign-post-text">
      <a href="#">View next<br><br>letter</a>
    </div>
  </div>

  <script src="viewletter.js"></script>
{% endblock %}
```

File này extends nội dung từ **base.html**:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <base href="{{cdn}}" />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="icon" type="image/x-icon" href="favicon.ico">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="" />
    <link href="https://fonts.googleapis.com/css2?family=Caveat&amp;family=Secular+One&amp;display=swap" rel="stylesheet" />
    <link href="main.css" rel="stylesheet" />
    <title>Write to the Easter Bunny!</title>
  </head>

  <body>

    {% block content %}{% endblock %}
  </body>
</html>
```
=> Trong nội dung **base.html**, có nội dung code sau `<base href="{{cdn}}" />`. Thẻ `<base>` trong `html` chỉ định đường dẫn cơ sở (base url) cho toàn bộ các liên kết tương đối trong nội dung file html. Do vậy các liên kết tương đối trong **base.html** và **viewletters.html** sẽ trở thành như sau:

`<link href="main.css" rel="stylesheet" />` => `{{cdn}}/main.css` => `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/main.css`

`<script src="viewletter.js"></script>` => `{{cdn}}/viewletter.js` => `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/viewletter.js`

