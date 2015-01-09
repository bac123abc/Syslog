Syslog
==============
Nhóm thực hiện:

|Họ và tên | Nick skype | Email |
|----------|------------|-------|
| Cao Ngọc Uy | uy.cao | caongocuy@gmail.com |
| Nguyễn Hoài Nam | namptit307 | namptit307@gmail.com |

Xin chào các bạn. Bài viết này chúng tôi xin giới thiệu về syslog.. Đầu tiên syslog là một ứng dung để ghi log của hệ thống, ta có thể hiểu đơn giản quá trình chúng ta ghi sổ nhật kí các hoạt động trong ngày vậy. Biết cách xem và sử dụng các bản tin log là một yêu cầu tối quan trọng đối với người quản trị hệ thống cần phải nắm được. 

Mục lục:

[1. Giới thiệu](#gioithieu)
	
[2. Các câu lệnh hỗ trợ xem syslog](#lenh)
				
[3. Chi tiết file cấu hình của syslog](#chitiet)
				
[4. Rotating Log](#rotating)
				
[5. Log-server](#log)
				
[6. Nâng cao với syslog](#nangcao)

[7. Lab mô hình log với dịch vụ WEB](#web)
	
- [a. Mô hình từ Ubuntu sang Ubuntu](#ubuntu)
	
- [b. Mô hình từ Centos sang Ubuntu](#centos)
				
[8. Lời kết](#lk)

=====================
<a name="gioithieu"></a>
#### 1. Giới thiệu

Syslog là một gói phần mềm trong hệ thống Linux nhằm để ghi bản tin log của hệ thống trong quá trình hoạt động như của kernel, deamon, cron, auth, hoặc các ứng dụng chạy trên hệ thống như http, dns, dhcp, ntp,..

Ứng dụng của log:
- Phân tích nguyên nhân gốc rễ của một vấn đề
- Giúp cho việc khắc phục sự cố nhanh hơn khi hệ thống gặp vấn đề
- Giúp cho việc phát hiện, dự đoán một vấn đề có thể xảy ra đối với hệ thống
- ...

Theo măc định các bản tin log của hệ thống được syslog lưu vào trong thư mục /var/log, và được lưu riêng rẽ đối với từng tác vụ trong hệ thông nhưng đối với tiến trình cron thì sẽ lưu trong file cron.log. Như hình dưới đây

<img class="image__pic js-image-pic" src="http://i.imgur.com/dFZUtfY.png" alt="" id="screenshot-image">

<a name="lenh"></a>
#### 2. Các câu lệnh hỗ trợ xem syslog

Đối với các file ghi log các bạn có thể dùng một số lệnh sau để giúp cho việc xem log

|Câu lệnh | Cú pháp |Ý nghĩa | Ghi chú thêm |
|---------|---------|--------|--------------|
|more | more [file] | Dùng xem toàn bộ nội dung của thư muc | Đối với câu lênh này nôi dung được xem theo từng trang. Bạn dung dấu "cách" để chuyển  trang |
|tail | tail [file] | In ra 10 dòng cuối cùng nội dung của file | thêm tùy chọn -n [số dòng] sẽ in ra số dòng theo yêu cầu |
|head | head [file] | In ra 10 dòng đầu tiên của nôi dụng file |
|tail -f | tail -f [file] | Dùng để xem ngay lâp tức khi có log đến | Đây là câu lệnh dùng phổ biến nhất nó giúp ta có thể xem ngay lập tức log mới đến, và nó sẽ in ra 10 dong cuối cùng trong nội dung file đó |

<a name="chitiet"></a>
#### 3. Chi tiết file cấu hình của syslog
 
##### File cấu hình của syslog
 
 - Trong CENTOS, file cấu hình là `/etc/rsyslog.conf` . File này chứa cả các rule về log
 - Trong UBUUNTU file cấu hình là `/etc/rsyslog.conf` nhưng các rule được định nghĩa riêng trong `/etc/rsyslog.d/50-defaul.conf` . File rule này được khai báo include từ file cấu hình `/etc/rsyslog.conf`

##### Dưới đây là file cấu hình và khai báo rule trong CENTOS
 ```
 # rsyslog v5 configuration file

# For more information see /usr/share/doc/rsyslog-*/rsyslog_conf.html
# If you experience problems, see http://www.rsyslog.com/doc/troubleshoot.html

#### MODULES ####

$ModLoad imuxsock # provides support for local system logging (e.g. via logger command)
$ModLoad imklog   # provides kernel logging support (previously done by rklogd)
#$ModLoad immark  # provides --MARK-- message capability

# Provides UDP syslog reception
#$ModLoad imudp
#$UDPServerRun 514

# Provides TCP syslog reception
#$ModLoad imtcp
#$InputTCPServerRun 514


#### GLOBAL DIRECTIVES ####

# Use default timestamp format
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# File syncing capability is disabled by default. This feature is usually not required,
# not useful and an extreme performance hit
#$ActionFileEnableSync on

# Include all config files in /etc/rsyslog.d/
$IncludeConfig /etc/rsyslog.d/*.conf

##@ THEM TU DAY VAO####
#### RULES ####
# Log all kernel messages to the console.
# Logging much else clutters up the screen.
kern.*                                                 /dev/console
############
*.*			@10.0.30.131
##########
# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none;local6.none             /var/log/messages
*.info							@10.0.30.131	
# The authpriv file has restricted access.
authpriv.*                                             -/var/log/secure
authpriv.*						@10.0.30.131
# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 *

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
######LOg http state
local4.*					/var/log/log_http.log
local4.*					@10.0.30.131


# ### begin forwarding rule ###
# The statement between the begin ... end define a SINGLE forwarding
# rule. They belong together, do NOT split them. If you create multiple
# forwarding rules, duplicate the whole block!
# Remote Logging (we use TCP for reliable delivery)
#
# An on-disk queue is created for this action. If the remote host is
# down, messages are spooled to disk and sent when it is up again.
$WorkDirectory /var/lib/rsyslog # where to place spool files
$ActionQueueFileName fwdRule1 # unique name prefix for spool files
$ActionQueueMaxDiskSpace 1g   # 1gb space limit (use as much as possible)
$ActionQueueSaveOnShutdown on # save messages to disk on shutdown
$ActionQueueType LinkedList   # run asynchronously
$ActionResumeRetryCount -1    # infinite retries if host is down
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
#*.* @@remote-host:514
# ### end of the forwarding rule ###

# A template to for higher precision timestamps + severity logging
$template SpiceTmpl,"%TIMESTAMP%.%TIMESTAMP:::date-subseconds% %syslogtag% %syslogseverity-text%:%msg:::sp-if-no-1st-sp%%msg:::drop-last-lf%\n"

:programname, startswith, "spice-vdagent"	/var/log/spice-vdagent.log;SpiceTmpl
```

Cơ bản trong file cấu hình của syslog cho chúng ta thấy được nơi lưu trữ của đối với các tiến trình của hệ thống. như hình sau:

<img class="image__pic js-image-pic" src="http://i.imgur.com/CEgKLZQ.png" alt="" id="screenshot-image">

Trong syslog nhưng rule như hình trên được chia là 2 trường:

##### Trường 1: Trường Seletor ( như ô vuông 1 trong hình )
- Trường Seletor  : Chỉ ra nguồn tạo ra log và mức cảnh bảo của log đó.
- Trong trường seletor có 2 thành phần và được tách nhau bằng dấu "."

Nguồn tao ra log có thể liệt kê một số nguồn sau

|Nguồn tạo log | Ý nghĩa |
|--------------|---------|
|kernel | Những log mà do kernel sinh ra |
|auth hoặc authpriv | Log do quá trình đăng nhập hoặc xác thực tài khoản |
|mail | Log của mail |
|cron | Log do tiến trình cron trong hệ thống |
|user | Log của đến từ ứng dụng của người dùng |
|lpr | Log từ hệ thống in ấn |
|deamon | Log từ các tiến trình chạy trên nền của hệ thống |
|ftp | Log từ tiến trình ftp | 
|local 0 -> local 7 | Log dự trữ cho sử dụng nội bộ |

  - Mức cảnh báo:

| Mức cảnh báo | Ý nghĩa |
|--------------|---------|
|emerg | Thông báo tình trạng khẩn cấp |
|alert | Hệ thống cần can thiệp ngay |
|crit | Tình trạng nguy kịch |
|error | Thông báo lỗi đối với hệ thống |
|warn | Mức cảnh báo đối với hệ thống |
|notice | Chú ý đối với hệ thống |
|info | Thông tin của hệ thống |
|debug | Quá trình kiểm tra hệ thống |

##### Trường 2: Trường action ( như ô vuông thứ 2 trong hình )
- Trường action:là trường để chỉ ra nơi lưu log của tiến trình đó.Có 2 loại là lưu tại file trong localhost hoặc gửi đến IP của Máy chủ Log ( chi tiết phần máy chủ log )

**Note:**
Đối với các dòng lệnh như sau:
```
mail.info         /var/log/mail
```
Khi đó lúc này bản tin log sẽ mail lại với mức cảnh báo từ info trở lên. Cụ thể là mức notice,warn,... nếu bạn chỉ muốn nó log lại mail với mức là info bạn phải sử dụng như sau: 
`mail.=info    /var/log/mail`
```
mail.* 
```
Lúc này kí tự * đại diên cho các mực đăng ý. Lúc này nó sẽ lưu hêt các mức của mail vào trong thư mục. tượng tự khi t đặt *. thì lúc này nó sẽ log lại tất cả các tiến trình của hệ thống vào một file
Nếu bạn muốn log lại tiến trình của mail ngoại trừ mức info bạn có thể dùng kí tự ! `mail.!info`
```
*.info;auth.none;mail.none        /var/log/message
```
Lúc này tất các log từ info của tiến trình hệ thống sẽ được lưu vào trong thư mục message nhưng đối với các log của mail và auth sẽ không lưu vào trong message. Đó là ý nghĩa của dòng auth.none;mail.none
<a name="rotating"></a>
#### 4. Rotating Log
Phần lớn các distro sẽ cài đặt một cấu hình syslog mặc định cho bạn, bao gồm logging to messages và các log files 
khác trong /var/log. Để ngăn cản những files này ngày càng trở nên cồng kềnh và khó kiểm soát, một hệ thống quay 
vòng log file (a log file rotation scheme) nên được cài đặt. Hệ thống cron đưa ra các lệnh để thiết lập những 
log files mới, những file cũ được đổi tên bằng cách thay một con số ở hậu tố. Với loại quay vòng này, /var/log/messages 
của ngày hôm qua sẽ trở thành messages.1 của ngày hôm nay và một messages mới được tạo. 
Sự luân phiên này được cấu hình cho một số lượng lớn các file, và các log files cũ nhất sẽ được
xoá khi sự luân phiên bắt đầu chạy. Ví dụ trong /var/log có các messages sau: messages, messages.1, messages-20141111, messages-20141118, ... 

Tiện ích thi hành rotation là logrotate. Lệnh này được cấu hình sử dụng cho một hoặc nhiều files - được xác định bởi các tham số đi cùng. File cấu hình mặc định là /etc/logrotate.conf. 
```
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    create 0664 root utmp
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```
Trong ví dụ này, bạn sẽ thấy: 
- Hệ thống sẽ quay vòng log files hàng tuần
- Lưu lại những thông tin logs đáng giá trong 4 tuần
- Sử dụng định dạng Ngày tháng thêm vào để làm hậu tố của log files (20141111, 20141118, ...)
- Thông tin về sự quay vòng log của các gói RPM nằm trong /etc/logrotate.d
- rotation được thiết lập cho 2 files: /var/log/wtmp và /var/log/btmp

<a name="log"></a>
#### 5. Log-server

Bài toán thực tế đặt ra khi áp dụng vào trong một hệ thống là phải cần một con Log-server đối với hê thống để lưu log của tất cả các client gửi về. Vì vậy trong syslog có hỗ trợ chức năng đó:
Mặc định trong syslog sử dụng port 514 để gửi và nhận thông tin log
- Đối với Log-server
Để chuyển một máy thành máy chủ log thì đâu tiền bạn phải mở cổng 514 và cấu hình trong file `rsyslog.conf` như sau:

<img class="image__pic js-image-pic" src="http://i.imgur.com/DBaVj01.png" alt="" id="screenshot-image">

+ Bỏ 2 dòng comment của ô vuông 1: Lúc này trên serverlog sẽ nhận các gói tin log với cổng 514 và truyền trên giao thức UDP
+ Bỏ 2 dòng comment của ô vuông 2: Lúc này trên serverlog sẽ nhận các gói tin log với cổng 514 và truyền trên giao thức TCP

- Đối với Client server
Trong file cấu hình syslog bạn làm như sau:

<img class="image__pic js-image-pic" src="http://i.imgur.com/CmURwV0.png" alt="" id="screenshot-image">

*Chú ý 1: @ IP máy chủ log* khi đó các log của mail cũng sẽ gửi đến ip của máy chủ log với port 514 và dùng giao thức UDP. Các bạn nhớ máy chủ log mở port 514 với kiểu truyền vận UPD hay TCP thì trên client cũng phải truyền đúng với giao thức như trên server.

*@IPserver:514* : Đối với giao thức UDP

*@@IPserver:514* : Đối với giao thức TCP

<a name="iptables"></a>
*Chú ý 2:* Ở đây khi cấu hình máy chủ syslog chúng tôi LAB với trường hợp tắt chắc năng firewall của máy chủ và máy client. Đề cập đến vấn đề firewall, bạn hay chắc chắn rẳng đã ở cổng đầu vào và ra với port 514 UDP hoăc TCP trên máy chủ và máy client. Khi bật chức năng iptables lên bạn phải nắm rõ được nguyên lý và cơ chế hoạt động của iptables để có thể thêm các rule đối với máy chủ và máy client. tìm hiểu iptables bạn có thể tham khảo tại [đây](https://github.com/hocchudong/IptablestrongLinux)

Đối với trên máy chủ và client bạn có thể dụng lệnh sau để mở port 514 UDP (TCP).
```
iptables -A INPUT -p udp --dport 514 -j ACCEPT
iptables -A OUTPUT -p udp --sport 514 -j ACCEPT
```
<a name="nangcao"></a>
#### 6. Nâng cao với syslog

Qua tìm hiểu chúng tôi nhận thức được kiến thức nâng cao hơn về phần syslog này như:
- Chỉnh sửa bản tin log đầu ra để đáp ứng yêu cầu công việc
- Cấu hình trên máy chủ log sao nó nó ghi thông tin đối với từng máy client một các riêng biệt và khoa học phục nhằm phục vụ cho việc kiểm tra log sau này
- Dùng chức năng rotating log tốt
- Ghi log lại đối với các dịch vụ như http, mysql, ntp,..
- Có thể ghi log vào hăn một cơ sở dữ liệu để tiện cho quá trình xem log
- Cơ chế mã hóa ssl đối với bạn tin log khi truyền trên mạng internet tránh trường hợp bị bắt gói tin
-...

Nhưng trong phạm vi bài viết này, chúng tôi chỉ giới hạn ở mức độ sơ khai nhất, các vấn đề nêu trên chúng tôi sẽ sớm làm rõ và đưa ra trong loạt các bài viết sau này về syslog.

<a name="web"></a>
#### 7. Lab mô hình log với dịch vụ WEB

Mô hình lab

<img class="image__pic js-image-pic" src="http://i.imgur.com/ailqyX1.png" alt="" id="screenshot-image">

<a name="ubuntu"></a>
###### a. Mô hình Ubuntu sang Ubuntu

Lúc này máy Ubuntu chạy dịch vụ http và là máy client gửi bản log về cho máy Ubuntun là log server

- Bước 1: Chỉnh sửa trong file cấu hình `/etc/rsyslog.conf` của máy chủ Log-server để nó có thể nhận các bản tin log từ các client gửi về.

<img class="image__pic js-image-pic" src="http://i.imgur.com/667Q082.png" alt="" id="screenshot-image">

Nếu bạn muốn trên máy chủ log tạo thành các thư mục lưu riêng log đối với từng máy Client gửi về thêm dòng này vào file cấu hình

<img class="image__pic js-image-pic" src="http://i.imgur.com/jNpIFEw.png" alt="" id="screenshot-image">

Và chuyển chủ sở hưu tập tin /log/var cho syslog để nó có thể tạo các file và thư mục trong /var/log
```
chown syslog.syslog /var/log
```


- Bước 2: Thêm  dòng này trong file cấu hình `/etc/rsyslog.conf` của máy Client 
```
*.*			@ [Địa chỉ IP của máy log-server]
```

- Bước 3: 

Thêm dòng sau đây vào file cấu hình của apache trong máy client: `/etc/apache2/apache2.conf`
```
ErrorLog syslog:local1
```
Thêm dòng sau vào file `/etc/apache2/sites-enabled/000-default.conf`
```
ErrorLog syslog:local1
CustomLog "| /usr/bin/logger -taccess -plocal1.info" combined
```
*Note: Dòng lệnh trên có ý nghĩa chuyển tất cả các Log của phần CustomLog vào đầu vào lệnh logger và lệnh logger cho ra đầu ra của log với nguồn là httpacces và với selector là local1.info. Bạn có thể tìm hiểu thêm lệnh logger tại [đây](http://linux.about.com/library/cmd/blcmdl1_logger.htm)*

Lúc này trên máy chủ log nó sẽ tạo ra một thư mục có tên của máy client và trong đó sẽ chứa tất cả các log của máy Client đó.

<img class="image__pic js-image-pic" src="http://i.imgur.com/EXhzms9.png" alt="" id="screenshot-image">

<a name="centos"></a>
###### b. Mô hình Centos sang Ubuntu

Lúc này máy Centos chạy dịch vụ http và là máy client gửi bản log về cho máy Ubuntun là log server

Phần cấu hình trên máy log server hoàn toàn như cấu hình đối với mô hình Ubuntu sang Ubuntu

Phần cấu hình trên máy Client là Centos

- Bước 1: Chỉnh sửa file `/etc/ryslog.conf`:thêm dòng sau đây vào file cấu hình syslog
```
*.* 		@ <địa chỉ ip của log server>
```

- Bước 2: Chỉnh sửa file cấu hình của http `/etc/httpd/conf/httpd.conf`

Ban thêm 2 dòng sau đây vào file cấu hình
```
ErrorLog syslog:local2
CustomLog "| /usr/bin/logger -thttp_acces -plocal2.info" combined
```
*Chú ý 1:* Bạn hay chắc chắn rằng trong file cấu hình của httpd chỉ có duy nhất dòng ErrorLog bạn vừa nhâp, những ErrorLog khác bạn hãy chuyển thành comment.

*Chú ý 2:* Chúng tôi đang lab với trường hợp tắt iptables. CÒn nếu bạn muốn lab trong trường hợp bạn bật iptables lên, hãy chắc chắn bạn hiểu về iptables nhé. mở port iptables bạn có thể xem lại tại [mục chú ý của phần 5](#iptables)


<a name="lk"></a>
#### 8. Lời kết

Bài viết trên đây của chúng tôi giới thiệu về syslog và những thứ cần thiết nhất khi tìm hiểu về syslog.Nó cũng là kiến thức mà khi tìm hiểu chúng tôi nhận được. Nắm được syslog là điều thực sự cần thiết đối với một người quản trị hệ thống. Nó là công cụ đắc lực nhất cho việc quan trị và sửa chữa hệ thống.

Tài liệu tham khảo:

[Link trang chủ của syslog](http://www.rsyslog.com/doc/master/index.html)

[Link 1](https://www.digitalocean.com/community/tutorials/how-to-view-and-configure-linux-logs-on-ubuntu-and-centos)

[Link 2](http://xmodulo.com/configure-syslog-server-linux.html)

