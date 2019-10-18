## NAC support
Đây là tài liệu ghi lại các sự cố và cách khắc phục sự cố thường gặp trong quá trình vận hành hệ thống Kiểm soát truy cập mạng. Một dự án của ngân hàng MSB

 [1. Quy trình khi gặp lỗi](#Quy_trinh)
 
 [2. Các lỗi thường gặp](#Loi)
 
 [3. Các lệnh cần lưu ý](#Lenh)
 
<a name="Quy_trinh"></a>
### 1. Quy trình khi gặp lỗi
Khi nhận được lỗi từ người dùng
B1: Kiểm tra trên ISE
B2: Kiểm tra trên switch và router
B3: Kiểm tra lỗi phía người dùng (Gọi điện thoại)
#### 1.1 Kiểm tra trên ISE [https://10.1.4.140](https://10.1.4.140/) trong phần Log xem trạng thái của người dùng đó như thế nào
* Nếu lỗi báo đỏ, chủ yếu trường hợp này là do người dùng đổi mật khẩu nhưng không restart máy để xác nhận mật khẩu mới, dẫn đến không xác thực được người dùng trên hệ thống. Làm cho cổng mạng đó bị chuyển sang vlan 150.
	* Khắc phục: bảo người dùng restart lại máy để nhập mật khẩu mới. Nếu không được thì Bypass cổng người dùng để người dùng có mạng để có thể login bằng mật khẩu mới. Sau khi người dùng login được vào máy bằng mật khẩu mới thì cấu hình lại dot1x trên cổng đó bằng lệnh `dot1x port-control auto`
* Nếu lỗi báo `NotApplicate` người dùng bị chuyển sang vlan cách ly. Trường hợp này xảy ra khi có lỗi trong quá trình xác thực người dùng hoặc lỗi do anyConnect.
	* Khắc phục: shut/no shut cổng mạng người dùng (Có thể lặp lại 2-3 lần nếu không được). Nếu không được có thể do lỗi file xml hoặc do người dùng chưa cài AnyConnect. Nếu cách trên không được, tạm thời ByPass công để người dùng có mạng, sau đó sử dụng [lệnh check](#comman1) để kiểm tra. Nếu máy đó đã cài anyconnect, đẩy lại file xml bằng [lệnh đẩy file xml](#comman2). Nếu máy đó chưa cài anyconnect, sử dụng [lệnh cài AnyConnect](#comman3) để cài AnyConnect.
* Nếu báo `Complient` Nhưng khi kiểm tra trên switch vẫn đang là Vlan 100 hoặc 150. Lỗi này xảy ra khi con switch lỗi không nhận được lệnh chuyển vlan từ ISE đẩy xuống.

	* Khắc phục shut/no shut cổng mạng người dùng. Bypass công và cấu hình lại. Bảo người dùng restart lại máy tính.

#### 1.2 Kiểm tra trên switch, router
* Có thể do cổng mạng đó đang cấu hình sai hoặc do switch bị treo chuyển được vlan cho người dùng.

#### 1.3 Kiểm tra phía người dùng
* Gọi người dùng xem lỗi chấm than đỏ hay vàng. 
* Xem máy có join đúng domain hay không, 
* Máy có đang bắt wifi không, 
* Sử dụng IP tĩnh không, 
* Máy có sử dụng win xp không
* Máy đang cắm trực tiếp vào port switch hay cắm qua IP Phone.
Cố gắng nắm được rõ tình trạng của user sẽ khiến cho việc tìm kiếm nguyên nhân lỗi và khắc phục được dễ hơn.
### 2. Các lỗi thường gặp <a name="Loi"></a>
* Người dùng đổi mật khẩu nhưng không restart
* Máy người dùng chưa cài any Connect
* Máy người dùng đăng nhập bằng tài khoản không phải tài khoản domain
* Máy người dùng chưa join domain
* Máy người dùng Chưa bật dịch vụ dot1x (wire any connect)
* Máy người dùng đang sử dụng win xp
* Máy người dùng không nhận DHCP
#### 2.1 Người dùng đổi mật khẩu nhưng không restart
Lỗi này đã nói đến cách khắc phục bên trên
#### 2.2 Chưa cài any connect
Kiểm tra bằng lệnh <a name="comman1"></a>
```
Net Use \\ip_may_nguoi_dung mat_khau /User:ten_tai_khoan
SC  \\ip_may_nguoi_dung Query aciseagent
```
```
Matkhau123456 /User:MSB\NAC2
m1t@2017 /User:administrator
htcnmsb@2018 /User:administrator
```
Với ip_may_nguoi_dung là ip có thể ping được đến (sau khi Bypass lên router kiểm tra IP, hoặc bỏ acl trên router và sử dụng luôn ip ở vlan người dùng đang mất mạng để kiểm tra)
Sau đó cài bằng lệnh: <a name="comman3"></a>
```
psexec -e \\ip_may_nguoi_dung   -u user -p mat_khau -s cmd /c (^msiexec /i \\10.2.142.99\antt\NAC_Project\anyconnect-win-4.7.02036-core-vpn-predeploy-k9.msi ^& msiexec /i \\10.2.142.99\antt\NAC_Project\anyconnect-win-4.7.02036-iseposture-predeploy-k9.msi ^& md "C:\ProgramData\Cisco\Cisco AnyConnect Secure Mobility Client\ISE Posture" ^& copy /Y \\10.2.142.99\antt\NAC_Project\ISEPostureCFG.xml "C:\ProgramData\Cisco\Cisco AnyConnect Secure Mobility Client\ISE Posture\" ^) >tempfullcommand.txt 2>fullcommand.log
```
Với các user
```
-u MSB\NAC2 -p Matkhau123456
-u administrator -p m1t@2017
-u administrator -p htcnmsb@2018
```
#### 2.3 Máy người dùng bị lỗi file xml
* Đẩy lại file xml bằng lệnh. <a name="comman2"></a>
```
PsExec.exe \\ip_may_nguoi_dung -u ten_tai_khoan -p mat_khau -s cmd /c "copy /Y \\10.2.142.99\antt\NAC_Project\ISEPostureCFG.xml "C:\ProgramData\Cisco\Cisco AnyConnect Secure Mobility Client\ISE Posture\""
```
Với các user
```
-u MSB\NAC2 -p Matkhau123456
-u administrator -p m1t@2017
-u administrator -p htcnmsb@2018
```
#### 2.4 Máy người dùng chưa bật dịch vụ dot1x và wire any connect
* Kiểm tra bằng lệnh `query`, restart dịch vụ bằng lệnh `stop/start` Bật dịch vụ luôn chạy bằng lệnh `config ten_dich_vu start=auto`
```
Net Use \\ip_may_nguoi_dung  Msb@2018 /User:MSB\admin-sccm
SC  \\ip_may_nguoi_dung Query aciseagent
SC  \\ip_may_nguoi_dung Query dot3svc
SC  \\ip_may_nguoi_dung  Start aciseagent
SC  \\ip_may_nguoi_dung  Stop aciseagent
SC \\ip_may_nguoi_dung Config aciseagent start= Auto
SC \\ip_may_nguoi_dung Config dot3svc start= Auto
```
#### 2.5 Máy người dùng đang dùng win xp hoặc đang join vào một domain khác
Lệnh kiểm tra
```
systeminfo /s ip_may_nguoi_dung  /u tai_khoan -p mat_khau
```
Với các tài khoản
```
/u administrator -p m1t@2017
/u administrator -p htcnmsb@2018
```
#### 2.6 Máy người dùng không nhận DHCP
* Gọi cho người dùng nếu người dùng đang có IP là 169.xxx.xxx.xxx có nghĩa là máy người dùng đang không nhận được DHCP từ router đẩy xuống. 
** Khắc phục: bảo người dùng vào cửa sổ cmd gõ lệnh 
```
ipconfig /release
ipconfig /renew
```
Lỗi này xảy ra chủ yếu tại HO Hà Nội trên switch HP

### 3 Các lệnh cần lưu ý <a name="Lenh"></a>
* Trên switch cisco
	* Kiểm tra mac vlan `show mac add`
	* Kiểm tra mac vlan trên port `show mac add int fa1/2/1`
	* Kiểm tra user đã xác thưc dot1x `show dot1x users`
	* Kiểm tra cấu hình trên port mạng `show run int fa1/2/1`
	* Kiểm tra switch đang kết nối đến những switch nào
	`show cdp nei`
* Trên router cisco
	* Kiểm tra ip, mac, vlan `show arp | include xxx` (xxx có thể là 4 số cuối địa chỉ MAC, địa chỉ IP, vlan)
	* Bỏ acl trên vlan 100
	`no ip acces-group QUARANTINE_NAC in`
	* Bỏ acl trên vlan 150
	`no ip access-group GUEST_NAC in`
* Trên switch HP
	* Kiểm tra cấu hình trên cổng
	`display cur int g1/0/5`
	* Vào cấu hình một cổng
	```
	system-view
	int g1/0/14
	```
