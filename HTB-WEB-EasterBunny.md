[HTB] Web - EasterBunny Write Up!
## By @ndkhai
***
* Link Challenge: `https://app.hackthebox.com/challenges/easterbunny`
* Challenge Description: `It's that time of the year again! Write a letter to the Easter bunny and make your wish come true! But be careful what you wish for because the Easter bunny's helpers are watching!`
* Necessary files to play the challenge: `Source Code`
***
## Sơ lược tính năng của ứng dụng
1. Giao diện chính của web:

2. Ứng dụng cho phép submit thư mới lên hệ thống, gọi vào link /submit. Nội dung thư mới sẽ được insert vào DB với id tăng dần.

3. Ứng dụng cho phép đọc nội dung của các thư hiện có trên hệ thống (link URL: /letters?id=x)
