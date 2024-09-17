# dự án: triển khai trang web bán thiết bị đảm bảo an ninh thông tin (10/2023 - 12/2023)

1. Phòng chống SQL Injection
SQL Injection là một trong những kiểu tấn công phổ biến nhất, thường xảy ra khi ứng dụng không kiểm tra kỹ lưỡng đầu vào từ người dùng trước khi chèn vào câu truy vấn SQL. Để phòng chống tấn công này, bạn có thể áp dụng các biện pháp sau:

a. Sử dụng Prepared Statements (Câu truy vấn có tham số)
Prepared statements là phương pháp tốt nhất để bảo vệ ứng dụng khỏi SQL Injection. Nó tách biệt dữ liệu đầu vào của người dùng khỏi câu truy vấn SQL, tránh việc dữ liệu bị tiêm nhiễm mã độc.

Ví dụ với PHP và MySQL:
php

Copy
$stmt = $mysqli->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->bind_param("ss", $username, $password);
$stmt->execute();
b. Kiểm tra và xác thực đầu vào
Xác thực và làm sạch tất cả dữ liệu đầu vào từ người dùng trước khi sử dụng.
Sử dụng các hàm như mysqli_real_escape_string() nếu không dùng Prepared Statements.
Hạn chế quyền truy cập của tài khoản cơ sở dữ liệu, chỉ cho phép thực hiện các hành động cần thiết (SELECT, INSERT, UPDATE).
2. Phòng chống Cross-Site Scripting (XSS)
XSS (Cross-Site Scripting) cho phép kẻ tấn công chèn mã JavaScript độc hại vào trang web mà người dùng khác có thể truy cập. Dưới đây là các biện pháp phòng chống XSS:

a. Escape các ký tự đặc biệt
Khi hiển thị dữ liệu đầu vào từ người dùng trên trang HTML, cần escape các ký tự đặc biệt như <, >, ', ", & để tránh việc mã JavaScript được thực thi.

Ví dụ với PHP:
php

Copy
echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8');
b. Sử dụng CSP (Content Security Policy)
Content Security Policy (CSP) là một cơ chế bảo mật giúp giảm thiểu nguy cơ từ các cuộc tấn công XSS bằng cách chỉ định các nguồn tài nguyên đáng tin cậy mà trình duyệt có thể tải và thực thi.

Ví dụ về CSP Header:
http

Copy
Content-Security-Policy: script-src 'self'; object-src 'none';
Đoạn mã trên chỉ cho phép các script từ domain của chính ứng dụng được thực thi, ngăn chặn việc tải các script từ nguồn không đáng tin.

c. Làm sạch dữ liệu đầu vào
Sử dụng các thư viện như DOMPurify để làm sạch dữ liệu đầu vào trước khi hiển thị nó trong nội dung HTML.

3. Phòng chống Cross-Site Request Forgery (CSRF)
CSRF là cuộc tấn công mà kẻ tấn công đánh lừa người dùng thực hiện các hành động không mong muốn trên một ứng dụng web mà người dùng đã đăng nhập.

a. Sử dụng Token CSRF
Mỗi khi thực hiện các hành động nhạy cảm (như thay đổi mật khẩu, gửi form), bạn nên sử dụng CSRF token để xác minh rằng yêu cầu được thực hiện từ người dùng hợp lệ.

Ví dụ với PHP:
Tạo CSRF Token:
php

Copy
session_start();
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));
Chèn CSRF Token vào form:
html

Copy
<form method="POST" action="/submit">
    <input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">
    <input type="submit" value="Submit">
</form>
Xác thực CSRF Token:
php

Copy
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die("Invalid CSRF token");
}
b. Sử dụng SameSite Cookies
Cài đặt thuộc tính SameSite cho cookie phiên (session cookie) để ngăn chặn các yêu cầu CSRF từ các trang web khác.

Ví dụ:
php

Copy
setcookie('session', $session_id, ['samesite' => 'Strict', 'secure' => true, 'httponly' => true]);
SameSite=Strict: Chỉ cho phép sử dụng cookie khi người dùng tương tác trực tiếp với trang web của bạn, ngăn không cho cookie bị gửi từ các trang web khác.
4. Phòng chống Command Injection
Command Injection xảy ra khi dữ liệu đầu vào của người dùng được chèn trực tiếp vào các lệnh của hệ thống mà không được kiểm tra hoặc làm sạch đúng cách.

a. Sử dụng các hàm hệ thống an toàn
Không bao giờ sử dụng hàm exec(), system(), shell_exec() trực tiếp với dữ liệu từ người dùng. Nếu bắt buộc phải sử dụng, hãy sử dụng các phương thức an toàn hơn như escapeshellarg() hoặc escapeshellcmd() để làm sạch dữ liệu đầu vào.

Ví dụ với PHP:
php

Copy
$filename = escapeshellarg($user_input);
system("cat " . $filename);
b. Sử dụng API nội bộ
Tốt nhất, nên tránh việc gọi lệnh hệ thống. Thay vào đó, sử dụng các API hoặc hàm nội bộ của PHP hoặc ngôn ngữ lập trình khác để xử lý các tác vụ.

5. Phòng chống Local File Inclusion (LFI) và Remote File Inclusion (RFI)
LFI và RFI là các lỗ hổng cho phép kẻ tấn công bao gồm các tệp tin cục bộ hoặc từ xa vào ứng dụng, thường thông qua các tham số URL hoặc đầu vào từ người dùng.

a. Kiểm tra và xác thực đầu vào
Không bao giờ cho phép người dùng nhập trực tiếp đường dẫn tệp. Nếu cần thiết, luôn xác thực đầu vào bằng cách giới hạn tệp mà người dùng có thể truy cập.

Ví dụ:
php

Copy
$allowed_files = ['home.php', 'about.php', 'contact.php'];
if (in_array($requested_file, $allowed_files)) {
    include $requested_file;
} else {
    die("Truy cập không hợp lệ.");
}
b. Vô hiệu hóa remote file inclusion
Nếu không cần sử dụng Remote File Inclusion, hãy đảm bảo rằng tùy chọn allow_url_include bị vô hiệu hóa trong tệp php.ini.

ini

Copy
allow_url_include = Off
6. Phòng chống Brute Force Attack
Brute Force Attack là kỹ thuật thử từng mật khẩu cho đến khi tìm ra mật khẩu đúng. Để phòng chống dạng tấn công này, có thể áp dụng các biện pháp sau:

a. Giới hạn số lần thử đăng nhập
Giới hạn số lần thử đăng nhập trong một khoảng thời gian nhất định bằng cách sử dụng cơ chế rate-limiting. Ví dụ, sau 5 lần thử đăng nhập sai, khóa tài khoản hoặc yêu cầu người dùng chờ vài phút.

Ví dụ:
php

Copy
if ($failed_attempts >= 5) {
    die("Bạn đã thử đăng nhập quá nhiều lần. Vui lòng thử lại sau 15 phút.");
}
b. Sử dụng CAPTCHA
Sử dụng CAPTCHA để ngăn chặn bot tự động thử nhiều lần đăng nhập.

Ví dụ về Google reCAPTCHA:
html

Copy
<form action="/submit" method="POST">
    <div class="g-recaptcha" data-sitekey="your-site-key"></div>
    <input type="submit" value="Submit">
</form>
c. Sử dụng mật khẩu mạnh và mã hóa
Yêu cầu người dùng tạo mật khẩu mạnh, bao gồm các ký tự đặc biệt, số, và chữ hoa.
Mã hóa mật khẩu bằng các thuật toán mã hóa mạnh như bcrypt hoặc argon2.
Ví dụ với PHP:
php

Copy
$hashed_password = password_hash($password, PASSWORD_BCRYPT);
7. Phòng chống Clickjacking
Clickjacking là kỹ thuật tấn công trong đó kẻ tấn công "đánh lừa" người dùng nhấp vào một nút hoặc liên kết ẩn bằng cách bao bọc trang web hợp pháp trong một iframe.

a. Sử dụng X-Frame-Options Header
Cài đặt header X-Frame-Options không cho phép trang web của bạn được nhúng vào các trang khác bằng iframe.

Ví dụ:
http

Copy
X-Frame-Options: DENY
DENY: Không cho phép nhúng trang web của bạn vào bất kỳ trang nào.
SAMEORIGIN: Chỉ cho phép nhúng trang web vào cùng một domain.
b. Sử dụng CSP (Content Security Policy)
Ngoài ra, bạn có thể dùng Content Security Policy để ngăn chặn nhúng iframe.

Ví dụ:
http

Copy
Content-Security-Policy: frame-ancestors 'none';
8. Phòng chống Directory Traversal
Directory Traversal là tấn công cho phép kẻ tấn công truy cập vào các tệp hoặc thư mục trên hệ thống mà ứng dụng không dự định cho phép.

a. Làm sạch đường dẫn đầu vào
Khi xử lý các đường dẫn tệp từ đầu vào, luôn làm sạch đường dẫn để loại bỏ các ký tự đặc biệt như ../ nhằm ngăn chặn tấn công Directory Traversal.

Ví dụ:
php

Copy
$file = basename($user_input);  // Loại bỏ các ký tự ../ khỏi đường dẫn
include "/var/www/html/" . $file;
b. Giới hạn quyền truy cập tệp
Giới hạn quyền truy cập tệp và thư mục của người dùng web. Cấu hình máy chủ để ngăn chặn truy cập vào các thư mục nhạy cảm như /etc/ hoặc /var/.

9. Phòng chống Man-in-the-Middle (MITM)
Man-in-the-Middle (MITM) là loại tấn công khi kẻ tấn công can thiệp vào quá trình giao tiếp giữa hai bên (người dùng và server) để đánh cắp hoặc thay đổi thông tin.

a. Sử dụng HTTPS
Bắt buộc sử dụng HTTPS trên toàn bộ ứng dụng để mã hóa thông tin trao đổi giữa người dùng và server, ngăn chặn việc thông tin bị đọc trộm.

b. Cài đặt HSTS (HTTP Strict Transport Security)
Cài đặt HSTS để đảm bảo trình duyệt chỉ giao tiếp với server qua HTTPS và ngăn chặn tấn công SSL Stripping.

Ví dụ:
http

Copy
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
10. Kết luận
