**QUẢN LÝ Ổ ĐĨA**

**A. Phân vùng ổ đĩa**

*1. Partition*
- Partition (phân vùng) của một đĩa là một đơn vị lưu trữ logic được chia từ một ổ đĩa vật lý. Một đĩa có thể được chia thành nhiều partition, mỗi partition sẽ được sử dụng cho một mục đích khác nhau (ví dụ: cài đặt hệ điều hành, lưu trữ dữ liệu,....). Mỗi partition có thể có một hệ thống tập tin (file system) riêng để quản lý dữ liệu trong đó.
- Mục đích: giúp hệ thống quản lý dữ liệu hiệu quả hơn, tạo không gian làm việc độc lập cho từng ứng dụng.

*2. Thao tác quản lý phân vùng bằng lệnh fdisk*

a. Khởi động fdisk: sudo fdisk <tên ổ đĩa cần thao tác>

b. Hiển thị bảng phân vùng: 
- Sau khi vào giao diện fdisk, sử dụng lệnh p để hiển thị thông tin các partition có trên ổ đĩa này.

c. Tạo phân vùng mới
- Nhập n để vào giao diện tạo phân vùng
- Sau khi vào giao diện:
  - Chọn loại phân vùng (primary cho phân vùng bình thường, extend nếu muốn tạo nhiều phân vùng con trong phân vùng đó)
  - Chọn số thứ tự phân vùng
  - Chọn kích thước phân vùng: Chọn sector đầu tiên và cuối cùng của phân vùng (mỗi sector có kích thước 512 bytes) 
- Lưu thay đổi: w để lưu thay đổi, q để không lưu thay đổi
- Định dạng phân vùng sau khi tạo: (format thành một file system: ext4, xfs,...): sudo mkfs.ext4 <tên phân vùng>

d. Xóa phân vùng: Nhấn d, sau đó chỉ định phân vùng muốn xóa

*3. Phân biệt partition với file system*
- Partition: Là phần không gian vật lí được chia ra từ một ổ đĩa hoặc 1 thiết bị lưu trữ
- File system: Là phương thức lưu trữ và quản lí dữ liệu trên ổ đĩa hay phân vùng
- *Mối quan hệ giữa partition và file system:* Sau khi tạo phân vùng, cần định dạng với 1 file system để có thể lưu trữ và quản lý dữ liệu

**B. RAID và lệnh mdadm**

*1. Cấu trúc và nguyên lý hoạt động của các cấp độ RAID*

*RAID là công nghệ cho phép kết hợp nhiều ổ đĩa thành một hệ thống lưu trữ duy nhất với mục đích cải thiện hiệu suất đọc/ghi hoặc đảm bảo an toàn dữ liệu tùy theo từng cấp độ RAID sử dụng*

a. RAID 0
- Số ổ đĩa tối thiểu: 2
- Nguyên lí hoạt động:
  - Ghi dữ liệu: Dữ liệu được chia thành các khối và ghi xen kẽ các khối đó lên các ổ đĩa trong mảng. Nếu có 1 ổ đĩa gặp sự cố thì tất cả dữ liệu trong mảng sẽ mất
  - Đọc dữ liệu: Khi đọc dữ liệu, các khối dữ liệu nằm trong các ổ đĩa được truy xuất đồng thời.
    
b. RAID 1
- Số ổ đĩa tối thiểu: 2
- Nguyên lý hoạt động: 
  - Ghi dữ liệu: Khi ghi dữ liệu, hệ thống sẽ sao chép dữ liệu đó cùng lúc vào tất cả các ổ. Mỗi ổ đĩa sẽ có một bản sao chính xác của dữ liệu
  - Đọc dữ liệu: Khi đọc dữ liệu, hệ thống có thể truy xuất từ bất kỳ ổ đĩa nào, thậm chí là từ nhiều ổ đĩa cùng lúc.

c. RAID 5 (*)
- Số ổ đĩa tối thiểu: 3
- Nguyên lí hoạt động:
  - Ghi dữ liệu: Chia dữ liệu thành các khối và ghi theo chu trình (mỗi chu trình sẽ ghi N-1 khối dữ liệu + 1 parity được tính toán từ N-1 khối trên, N khối này được ghi xen kẽ vào N ổ trong mảng theo vòng tròn). Các chu trình này lặp lại đến khi mỗi block dữ liệu được ghi đầy đủ vào một ổ đĩa duy nhất
  - Đọc dữ liệu: Khi đọc một tệp dữ liệu từ mảng RAID 5, các phần dữ liệu sẽ được đọc đồng thời từ các ổ đĩa.
  - Khôi phục dữ liệu: nếu một ổ bị hỏng, có thể khôi phục dữ liệu từ thông tin parity của các ổ còn lại

- VÍ DỤ:

Giả sử cần ghi 6 block dữ liệu: Block 1 đến Block 6. Mỗi chu kỳ ghi sẽ có 2 block dữ liệu và 1 block Parity.
Chu trình 1:

    Block 1 (Dữ liệu) sẽ được ghi đầy đủ vào Ổ A.
    Block 2 (Dữ liệu) sẽ được ghi đầy đủ vào Ổ B.
    Parity của Block 1 và Block 2 sẽ được tính toán và ghi vào Ổ C.

Kết quả sau chu trình 1:

    Ổ A: Block 1 (Dữ liệu)
    Ổ B: Block 2 (Dữ liệu)
    Ổ C: Parity (Block 1 XOR Block 2)

Chu trình 2:

    Block 3 (Dữ liệu) sẽ được ghi đầy đủ vào Ổ B.
    Block 4 (Dữ liệu) sẽ được ghi đầy đủ vào Ổ C.
    Parity của Block 3 và Block 4 sẽ được tính toán và ghi vào Ổ A.

Kết quả sau chu trình 2:

    Ổ A: Parity (Block 3 XOR Block 4)
    Ổ B: Block 3 (Dữ liệu)
    Ổ C: Block 4 (Dữ liệu)

Chu trình 3:

    Block 5 (Dữ liệu) sẽ được ghi đầy đủ vào Ổ C.
    Block 6 (Dữ liệu) sẽ được ghi đầy đủ vào Ổ A.
    Parity của Block 5 và Block 6 sẽ được tính toán và ghi vào Ổ B.

Kết quả sau chu trình 3:

    Ổ A: Block 6 (Dữ liệu)
    Ổ B: Parity (Block 5 XOR Block 6)
    Ổ C: Block 5 (Dữ liệu)

*2.Đánh giá hiệu năng và độ an toàn so với dùng 1 ổ cứng*

a. RAID 0
- Hiệu suất: Hiệu suất cao do dữ liệu được chia thành các phần nhỏ được đọc/ghi song song trên tất cả ổ đĩa. Với RAID 0, tốc độ đọc/ghi có thể gấp 2 lần hoặc hơn so với sử dụng một ổ (tùy vào số lượng ổ đĩa trong mảng).
- An toàn: Độ an toàn của RAID 0 kém hơn nhiều so với một ổ đĩa thông thường vì không có dự phòng hoặc bảo vệ dữ liệu. Nếu có 1 ổ đĩa gặp sự cố thì tất cả dữ liệu trên mảng RAID sẽ mất

b. RAID 1
- Hiệu suất: Vì dữ liệu được sao chép từ nhiều ổ đĩa, tốc độ đọc có thể nhanh hơn so với một ổ đĩa đơn. Tốc độ ghi không cải thiện so với một ổ đĩa đơn vì dữ liệu phải được ghi vào tất cả các ổ đĩa trong mảng.
- An toàn: RAID 1 mang đến sự bảo vệ tốt hơn vì dữ liệu được sao chép sang các ổ đĩa. Nếu một ổ đĩa bị hỏng, vẫn có thể truy cập vào bản sao dữ liệu từ các ổ đĩa còn lại.

c. RAID 5
- Hiệu suất: RAID 5 có hiệu suất cao hơn ổ đĩa đơn (vì có striping), nhưng thấp hơn RAID 0 vì cần tính toán parity.
- An toàn: RAID 5 cung cấp bảo vệ dữ liệu tốt hơn so với ổ đĩa đơn nhờ vào parity, và có thể chịu được một ổ đĩa bị hỏng mà không mất dữ liệu. Tuy nhiên, độ an toàn thấp hơn RAID 1 (nếu có 2 ổ bị hỏng sẽ bị mất dữ liệu)

*3. Lệnh mdadm để quản lý RAID*

*mdadm là công cụ chính để quản lý RAID trong Linux. Nó cho phép người dùng tạo, cấu hình và duy trì các mảng RAID trên các đĩa cứng*

a. Cài đặt mdadm
- sudo apt update
- sudo apt install mdadm

b. Tạo mảng RAID mới
- Để tạo mảng RAID mới, ta dùng lệnh mdadm --create. Đây là ví dụ lệnh tạo mảng RAID 1 với hai ổ đĩa /dev/sdb và /dev/sdc:

  sudo mdadm --create /dev/md0 --level=1 --raid-device=2 /dev/sdb /dev/sdc
  - /dev/md0 : tên mảng RAID mới
  - --level=1 : Cấp độ RAID
  - --raid-device=2 : Số ổ đĩa trong mảng RAID
  - /dev/sdb /dev/sdc : Các ổ đĩa trong mảng
 
c. Xem thông tin mảng RAID: sudo mdadm --detail /dev/md0
![image](https://github.com/user-attachments/assets/6f3a728b-170a-41bb-8537-1d956f5a1e5f)

Các thông tin cần lưu ý khi muốn kiểm tra mảng RAID: 
- State (của mảng RAID): Nếu mảng RAID ở trạng thái bình thường sẽ là clean. Nếu mảng bị lỗi, có thể thấy các trạng thái như degraded, resyncing, repairing, hoặc rebuilding.
- State (của ổ đĩa): Nếu ở trạng thái bình thường sẽ hiển thị là active sync. Nếu thấy trạng thái của ổ đĩa là removed (bị loại bỏ khỏi mảng), faulty (gặp sự cố) hoặc degraded (mảng RAID không hoàn thiện), đó là dấu hiệu cho thấy ổ đĩa gặp sự cố.
- Failed Devices: Nếu con số lớn hơn 0, thì có một ổ đĩa hoặc nhiều ổ đĩa đã gặp sự cố.

Ngoài ra, ta cũng có thể xem thông tin tổng quan tất cả các mảng RAID trong hệ thống bằng lệnh: cat /proc/mdstat

d. Thay thế ổ đĩa khi gặp lỗi 
- Xóa ổ đĩa: 
  - sudo mdadm --manage /dev/md0 --fail /dev/sdb1
  - sudo mdadm --manage /dev/md0 --remove /dev/sdb1
- Thêm ổ đĩa mới: sudo mdadm --manage /dev/md0 --add /dev/sdd
- Cập nhật trong file cấu hình: sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf

e. Xóa mảng: 
- Dừng mảng: sudo mdadm --stop /dev/md0
- Loại bỏ mảng: sudo mdadm --remove /dev/md0
- Xóa các khối của các ổ đĩa, giúp tái sử dụng các ổ này cho mảng khác: sudo mdadm --zero-superblock /dev/sdb /dev/sdc

f. Cấu hình RAID để tự mount khi hệ thống khởi động
- Tạo file system cho mảng RAID: sudo mkfs.ext4 /dev/md0
- Gắn mảng vào hệ thống:
  - sudo mkdir /mnt/raid
  - sudo mount /dev/md0 /mnt/raid
- Cấu hình để RAID tự động mount khi khởi động:
  - Mở tệp /etc/fstab (tệp cấu hình cách gắn các thiết bị lưu trữ vào hệ thống)
    - vim /etc/fstab
  - Thêm dòng sau vào cuối tệp
    - /dev/md0    /mnt/raid    ext4    defaults    0    0
- Lưu cấu hình RAID:
  - Cập nhật file cấu hình: sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf
  - Cập nhật initramfs (để đảm bảo RAID được nhận diện trong quá trình khởi động)
    - sudo update-initramfs -u
