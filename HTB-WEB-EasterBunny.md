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
=> Trong nội dung **base.html**, có nội dung code sau `<base href="{{cdn}}" />`
