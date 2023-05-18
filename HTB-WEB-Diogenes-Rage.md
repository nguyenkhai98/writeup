## By @ndkhai
***
* Link Challenge: https://app.hackthebox.com/challenges/diogenes-rage
* Challenge Description: `Having missed the flight as you walk down the street, a wild vending machine appears in your way. You check your pocket and there it is, yet another half torn voucher coupon to feed to the consumerism. You start wondering why should you buy things that you don't like with the money you don't have for the people you don't like. You're Jack's raging bile duct.`
* Necessary files to play the challenge: `Source Code`
***
## Sơ lược tính năng của ứng dụng

Okay, đầu tiên chúng ta nên lướt và thao tác qua một vài tính năng của ứng dụng. Điều này giúp ta có cái nhìn tổng quát về ứng dụng. Ví dụ ứng dụng này là gì, nó làm được gì, các tính năng chính...

1. Giao diện chính của web:

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/cd62cd14-097d-45e6-b4b9-c6ef699437a6)


2. Ứng dụng cho phép tài khoản nhận **coupon** (`1.00$` sẽ được cộng vào số dư của tài khoản)

3. Người dùng truyền vào giá trị tên của item để thực hiện mua item

## Source code
Okay, sau khi nắm được một vài thông tin, tính năng chính của ứng dụng, chúng ta tiến hành đọc source của Challenge

Dòng code số 31-34 trong file `routes/index.js` chỉ ra đoạn logic chương trình chúng ta cần reach đến để đạt được nội dung file `/app/flag` => Khi thực hiện mua thành công item **C8** thì sẽ get được flag.
![image](https://github.com/nguyenkhai98/writeup/assets/51147179/bbba9892-afc4-4bd9-869e-18f80d2dcd20)


Để mua được thành công item **C8** thì số dư tài khoản của chúng ta cần phải nhiều hơn price của item này (`13.37$`). Nghiên cứu thêm 1 chút về vấn đề số dư tài khoản 

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/1ca3a5c7-46a1-452d-b852-a317ec95b816)

=> Dòng code số 18-20 sẽ lấy giá trị user từ hàm `db.getUser(req.data.username)` và thực hiện kiểm tra giá trị user có tồn tại hay không, nếu chưa tồn tại thì sẽ đăng ký user mới và set balance = `0.00$` với hàm `db.registerUser(req.data.username)`
Giá trị đầu vào của req.data.username được nhận vào qua xử lý check trong file `AuthMiddleware.js`:
 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/c2ba3d99-efa7-412d-a64e-8d0ec4dc7c0e)

=> Nếu tồn tại giá trị Cookie name = session thì sẽ thực hiện verify giá trị này bằng cách gọi hàm `JWTHelper.verify()` để lấy ra giá trị **username**. Trường hợp nếu chưa tồn tại giá trị Cookie name = session thì sẽ tạo mới giá trị username theo cấu trúc như đoạn code từ dòng số 7->10.

![image](https://github.com/nguyenkhai98/writeup/assets/51147179/545f7c0b-fc94-4c93-829c-c230d157aba3)

Ảnh bên trên là nội dung của hàm `getUser()`, nhận tham số đầu vào và đưa vào câu truy vấn SQL và trả về kết quả là **user data**.

=> Okay, vậy muốn lấy được nội dung flag thì chúng ta phải tìm cách để số dư tài khoản (balance) đạt được giá trị lớn hơn giá của item **C8** (`13.37$`).

Đào sâu hơn source code của chương trình để tìm kiếm thêm thông tin các hàm có thể tương tác, thay đổi được số dư tài khoản. => Ta phát hiện ra Challenge cho phép apply coupon vào tài khoản người dùng ở chứcnăng `/api/coupons/apply` bằng cách gọi hàm `db.addBalance()` (dòng code số 61). 
 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/457b104a-ef8a-45aa-bb07-90ec24d624f2)

Tuy nhiên, hệ thống chỉ tồn tại duy nhất 1 mã coupon tên là `HTB_100` với giá trị tiền tương ứng là `1.00$`. Với số tiền này thì không đủ `13.37$` để mua **C8**.

 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/c3727281-9ef4-4cab-8e99-76324b16bf4f)


=> Brain storming:  Thử apply coupon **HTB_100** nhiều lần trên 1 tài khoản xem liệu số dư tài khoản có được cộng thêm **n** lần `1.00$` tương ứng với **n** lần apply coupon không?

<img width="627" alt="image" src="https://github.com/nguyenkhai98/writeup/assets/51147179/d440a930-52f8-4bba-8063-c0162fa591e9">

Kết quả là không thành công, ứng dụng không cho phép 1 tài khoản được apply coupon **HTB_100** nhiều hơn 1 lần và trả lại message `"This coupon is already redeemed!"`.
Logic xử lý của phần này ở dòng code số 54->57 và dòng code số 63.

 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/0072a7b8-a517-4f44-bbb0-4fabe0551caf)

Sau khi user apply được coupon lần đầu bằng hàm `db.addBalance()` thì sẽ tiếp tục gọi hàm `db.setCoupon()` để update giá trị coupon của user tương ứng trong Database
 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/66fec470-e714-4432-a9bc-7ee0d96ee756)

Đến lượt apply coupon từ lần thứ 2 trở đi, ứng dụng check giá trị coupon trong Database tương ứng của User hiện tại, nếu đã tồn tại coupon rồi thì không thực hiện hàm `db.addBalance()` nữa mà trả về message như thử nghiệm ta thực hiện bên trên.

**STUCK!** Đến đây mình đang bị tắc, không nghĩ ra cách nào để xử lý tiếp... Tiếp tục Brain Storming...
...
**BOOM!** Thử đến **Race Condition** xem sao, do dòng code số 61 gọi `db.addBalance()` được thực hiện trước dòng code số 63 `db.setCoupon()`
 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/b2362b8b-cf59-46ce-84e4-b30032d8a165)

=> Nếu ta thực hiện liên tục nhiều request add coupon cùng 1 thời điểm nhằm trigger hàm `db.addBalance()` thực hiện n lần trước khi hàm `db.setCoupon()` được thực hiện thì sẽ có số dư tài khoản là `n.00$`
Đoạn code xử lý của chương trình cũng không có cơ chế sử dụng khóa locks hay mutex để hạn chế Race Condition

***
## Exploit

Tiến hành code một đoạn script để exploit:
 ![image](https://github.com/nguyenkhai98/writeup/assets/51147179/e1172788-4bb3-4143-91fd-53abd3c5aea0)

Sau khi run fail vài lần script trên, thì có thời điểm ta cũng reach được kết quả mong muốn. Kết quả thu được:


 <img width="800" alt="image" src="https://github.com/nguyenkhai98/writeup/assets/51147179/4e65ae80-ba20-4b1c-aa96-b0c78fb59075">


