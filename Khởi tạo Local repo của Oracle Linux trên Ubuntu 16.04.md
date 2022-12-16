# Các bước khởi tạo local Oracle Linux repo trên Ubuntu 16.04.md

## 1. Các bước thực hiện tạo local repo cho OL8

### Bước 1: Khởi tạo các thư mục tương ứng với Repo của NSX trên Ubuntu 16.04

VD ở đây ta sẽ khởi tạo local repo của OL8 (Oracle Linux 8):

![image](https://user-images.githubusercontent.com/75653012/208012722-d673dd3b-0d67-41d4-b703-d85c70f48044.png)

Ở đây các folder bao gồm:
- ol8_addons
- ol8_appstream_latest
- ol8_baseos_latest
- ol8_UEKR7

Tương ứng với các Package có trên thư viện của OL8 trên Internet:

![image](https://user-images.githubusercontent.com/75653012/208013178-cafc9641-424b-4d2a-80f8-2d0aef7a9097.png)

### Bước 2: Tiến hành synchronize OL8 repositories vào local repo trên Ubuntu 16.04

Sử dụng reposync tool để tiến hành đồng bộ hóa OL8 repositories vào local repo trên Ubuntu 16.04

```
reposync -d -g -m --download-metadata -a x86_64 -r ol8_UEKR7 -r ol8_baseos_latest -r ol8_appstream_latest -r ol8_addons -c /etc/yum/yum.conf -p /repo/ol/8/
```

Giải thích command:

- –g (--gpgcheck)       : Cho phép chúng ta xóa hoặc gỡ cài đặt các gói trên OL8 không kiểm tra được GPG
- –d (--delete)         : Xóa các gói cục bộ (local packages) không còn trong repository.
- –m (--download compx) : lets you download comps.xml files, hữu ích để đóng gói các nhóm gói theo chức năng
- -n (--newest-only)    : Download only newest packages per-repo.
- -l (--plugins)        : Enable yum plugin support.
- -a (ARCH --arch=ARCH) : Act as if running the specified (chỉ định) arch (default: current arch, note: does not override $releasever. x86_64 is a superset for i*86.).
- -r (REPOID --repoid=REPOID)         : Chỉ định ID repo để truy vấn, có thể được chỉ định nhiều lần (default is all enabled)
- -c (CONFIG --config=CONFIG)         : Config file to use (defaults to /etc/yum.conf).
- -p (DESTDIR --download_path=DESTDIR): Path to download packages to: defaults to current directory.
- ––download-metadata: download non-default metadata

Bổ sung kiến thức:

- Đồng bộ hóa all packages from the 'updates' repo to the current directory                          : reposync --repoid=updates
- Chỉ đồng bộ hóa các gói mới nhất từ kho lưu trữ 'cập nhật' sang thư mục hiện tại                   : reposync -n --repoid=updates
- Đồng bộ hóa các gói từ kho lưu trữ 'updates' và 'extras' với thư mục hiện tại                      : reposync --repoid=updates --repoid=extras
- Sync all packages from the 'updates' repo to the repos directory                                   : reposync -p repos --repoid=updates
- Sync all packages from the 'updates' repo to the repos directory excluding (loại trừ) x86_64 arch  : reposync -p repos --repoid=updates
  Edit /etc/yum.conf adding option exclude=*.x86_64
  
Lưu ý: 
- reposync uses the yum libraries for retrieving information and packages. If no configuration file is specified, the default yum configuration will be used.
  - /etc/yum.conf
  - /etc/yum/repos.d/
-  Nếu chúng ta muốn tiếp tục chạy các process thậm chí cả sau khi đã logout hoặc ngắt kết nối khỏi shell hiện tại, khi đó chúng ta có thể sử dụng lệnh nohup:
   ```
   nohup reposync -d -g -m --download-metadata -a x86_64 -r ol8_UEKR7 -r ol8_baseos_latest -r ol8_appstream_latest -r ol8_addons -c /etc/yum/yum.conf -p /repo/ol/8/ &
   ```

## 2. Bugging đang lỗi 

Sau khi chạy lệnh reposync thành công, công việc bị dừng lại và không chạy tiếp tại đây:

![image](https://user-images.githubusercontent.com/75653012/208014849-3d0a91b8-94cf-4c34-ae86-aa3add0e3cd6.png)

Sau khi pull về 41056 file rpm từ Internet, khi ls các file có trong repo của OL8 vẫn trắng tinh!

![image](https://user-images.githubusercontent.com/75653012/208015060-1184d43e-aeef-4409-82cd-1f1314dd73f3.png)
