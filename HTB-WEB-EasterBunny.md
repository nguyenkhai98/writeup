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
