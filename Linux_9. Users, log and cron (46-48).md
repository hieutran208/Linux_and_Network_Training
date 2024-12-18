**QUẢN LÝ NGƯỜI DÙNG, FILE LOG VÀ CÁC TÁC VỤ TỰ ĐỘNG**

**A. Quản lý người dùng và nhóm người dùng**

*1. Quản lý người dùng*

a. Tạo tài khoản người dùng mới với useradd:
- Cú pháp: useradd [Option] [tên tài khoản]
- Options: 
  - -u: tạo userID, nếu không tạo sẽ lấy ID tiếp theo để gán cho user
  - -g: chỉ định nhóm chính, nếu không sẽ tự động tạo ra nhóm chính giống tên user
  - -s: chỉ định shell của người dùng (/bin/zsh)

b. Sửa thông tin tài khoản với lệnh usermod
- Cú pháp: usermod [Option] [tên tài khoản]
- Options: 
  - -g: đổi lại nhóm chính
  - -G <group1,group2,...>: Thêm vào nhóm khác (chỉ còn thuộc về các nhóm được chỉ định trong lệnh)
  - -aG: Thêm vào nhóm khác (không làm mất các nhóm hiện tại)
  - -s: Thay đổi shell
  - -l <tên_mới> <tên_cũ>: Thay đổi tên user
  - -d /đường_dẫn/thư_mục_mới -m: Thay đổi thư mục home

c. Xóa người dùng: userdel [tên tài khoản] (-r: Xóa cả thư mục home)

d. Các thao tác khác
- Kiểm tra thông tin người dùng: id [tên người dùng]
- Kiểm tra nhóm của người dùng: groups [tên người dùng]
- Hiển thị tên người dùng hiện tại: whoami
- Thay đổi mật khẩu: passwd [tên người dùng]
- Kiểm tra xem người dùng đã được thêm vào nhóm chưa: groups [tên người dùng]

*2. Quản lí nhóm*

a. Tạo nhóm mới: sudo groupadd <tên_group>

b. Thay đổi thông tin nhóm
- Thay đổi tên nhóm: groupmod –n + [tên nhóm mới] +[tên nhóm cũ]
- Thay đổi ID nhóm: groupmod –g + [tên ID] + [tên nhóm]
- Xoá người khỏi nhóm: gpasswd -d [tên người dùng] + [tên nhóm]
- Thêm người vào nhóm: gpasswd -a [tên người dùng] + [tên nhóm] (hoặc usedmod -aG)

c. Xóa nhóm: groupdel <tên nhóm>

d. Hiển thị thông tin nhóm: getent group <tên nhóm>

**B. Sử dụng file log trong hệ thống**

*1. Khái niệm và vị trí của file log trong hệ thống*
- File log của hệ thống là những file văn bản ghi lại các các sự kiện quan trọng của hệ thống và các chương trình trên máy tính (có người dùng đăng nhập, có dịch vụ gặp lỗi, hệ thống khởi động hoặc tắt,...). Những thông tin này giúp người quản trị hệ thống theo dõi và kịp thời khắc phục các sự cố xảy ra.
- Vị trí của file log trong hệ thống: các file log quan trọng của hệ thống và các ứng dụng thường được lưu trong thư mục /var/log/.
  - /var/log/syslog: Lưu trữ các sự kiện chung của hệ thống (thông tin khởi động, thông báo của hệ thống, các lỗi trong quá trình hoạt động,...)
  - /var/log/auth.log: Lưu trữ các sự kiện liên quan đến xác thực và quyền truy cập người dùng. Thông tin trong file này giúp theo dõi quá trình đăng nhập và đăng xuất của người dùng.
  - /var/log/kern.log: Lưu các thông báo từ kernel (thường liên quan đến các sự kiện phần cứng hoặc toàn hệ thống.)
  - /var/log/dmesg: Chứa các thông báo của hệ thống trong quá trình khởi động. 
  - /var/log/nginx: Chứa các sự kiện liên quan đến hoạt động của ứng dụng nginx

*2. Rotate log*
- Rotate log: Xoay log là việc tạo một file log mới và lưu trữ file log cũ dưới dạng có thể nén lại được khi file log cũ có kích thước đủ lớn hoặc đã tồn tại đủ lâu
- Tác dụng:
  - Tiết kiệm dung lượng ổ đĩa
  - Dàng truy xuất thông tin
  - Nén các file log cũ giúp tránh bị lộ dữ liệu hệ thống nếu xảy ra lỗi.
- Cách rotate log: Sử dụng công cụ logrotation. Với công cụ này, ta chủ yếu thực hiện cấu hình trong các file/thư mục:
  - /etc/logrotate.conf: File cấu hình chính của logrotate để thiết lập các tham số chung.
  - /etc/logrotate.d/: Thư mục chứa các file cấu hình riêng biệt cho từng ứng dụng hoặc dịch vụ (ví dụ: nginx, apache, syslog).

*3. Thử cấu hình rotate log cho /var/log/syslog*
- Mở file cấu hình chính /etc/logrotate.conf hoặc tạo mới file cấu hình /etc/logrotate.d/syslog để thêm vào cấu hình syslog
- Thêm phần cấu hình mẫu như sau:

  ![image](https://github.com/user-attachments/assets/d73e2ec2-c517-4ad1-b3cc-b4af92cbe5bc)
  - daily: Xoay log mỗi ngày. Có thể sử dụng các tùy chọn khác:
      - weekly: Xoay log hàng tuần.
      - monthly: Xoay log hàng tháng.
  - missingok: Nếu file log /var/log/syslog không tồn tại (ví dụ, dịch vụ không ghi log), không báo lỗi và tiếp tục thực hiện các bước tiếp theo.
  - rotate 7: Sau khi xoay log, logrotate sẽ giữ lại 7 file log cũ và xóa các file cũ hơn
  - compress: Nén các file log cũ sau khi xoay
  - delaycompress: Chỉ nén các file log cũ sau khi chúng đã được xoay ít nhất một lần.
  - notifempty: Nếu file log rỗng (không có bất kỳ dữ liệu nào), logrotate sẽ không xoay file đó
  - create 0640 root adm: Khi một file log mới được tạo sau khi xoay, tham số này quy định quyền truy cập cho file log đó. 0640 có nghĩa là chủ sở hữu (root) có quyền đọc và ghi, nhóm (adm) có quyền đọc, còn những người khác không có quyền gì.
  - sharedscripts: Nếu có nhiều file log được định nghĩa trong một khối cấu hình, các script trong phần postrotate chỉ được thực thi một lần thay vì thực thi cho từng file log
  - postrotate và endscript: Các lệnh trong phần này sẽ được thực thi sau khi các file log đã được xoay. Lệnh này thường được sử dụng để khởi động lại dịch vụ hoặc thực hiện các công việc cần thiết sau khi xoay log.
  - /usr/bin/systemctl reload rsyslog > /dev/null 2>&1 || true: Lệnh này thực hiện reload lại dịch vụ rsyslog sau khi log được xoay. Điều này giúp dịch vụ rsyslog tiếp tục ghi vào file log mới. Kết quả của lệnh được chuyển hướng về /dev/null (không hiển thị thông báo), và nếu có lỗi, lệnh sẽ bỏ qua lỗi đó (nhờ || true).
- Kiểm tra và áp dụng cấu hình Logrotate
  - Test cấu hình: logrotate -d /etc/logrotate.conf
  - Áp dụng cấu hình: logrotate -f /etc/logrotate.conf

**C. Cron và cách thiết lập cron**

*1. Cron và cú pháp cơ bản*

a. Khái niệm
- Cron là một tiện ích trong các hệ điều hành Unix/Linux dùng để lên lịch và tự động thực thi các tác vụ vào một thời gian nhất định hoặc theo chu kỳ lặp lại định kỳ
- Mục đích sử dụng: dùng để tự động hóa các công việc trong hệ thống (sao lưu dữ liệu, xoá các file cũ, kiểm tra log,....)

b. Cú pháp cơ bản
- Để lên lịch một tác vụ tự động, cần thêm một dòng cron job vào file cấu hình cron. Đây là cú pháp cơ bản của 1 dòng cron job:
  
  ![image](https://github.com/user-attachments/assets/ddd7f9d6-6097-4e06-98e2-7bd36abcfb2f)
  - Phút (0 - 59): Thời gian trong phút (0 đến 59).
  - Giờ (0 - 23): Thời gian trong giờ (0 đến 23, theo định dạng 24 giờ).
  - Ngày trong tháng (1 - 31): Ngày trong tháng (1 đến 31).
  - Tháng (1 - 12): Tháng trong năm (1 đến 12).
  - Ngày trong tuần (0 - 6): Ngày trong tuần (0 = Chủ nhật, 1 = Thứ hai,....). Dùng * để chỉ tất cả các ngày trong tuần.
- VÍ DỤ: 0 12 * * 1 /path/to/script.sh => Cron job sẽ chạy vào 12h AM mỗi thứ 2 trong tuần

*2.Thiết lập cron*

a. Các thao tác thường dùng với cron
- Xem các cron job được thiết lập cho user hiện tại: crontab -l
- Chỉnh sửa cronjob của user hiện tại: crontab -e
- Xem cron jobs của người dùng khác (Cần quyền root): sudo crontab -u [tên người dung] -l

b. Thiết lập 2 job: 1 job chạy vào 2 giờ sáng mỗi ngày và 1 job chạy vào 3 giờ, 2 ngày 1 lần.
- Chỉnh sửa file crontab: crontab -e
- Thêm vào các dòng sau:
  - job chạy hằng ngày lúc 2h: 0 2 * * * path/to/script.sh
  - job chạy 2 ngày/lần lúc 3h: 0 3 */2 * * path/to/script.sh
- Lưu file crontab và thoát
