Syslog
==============

Xin chào các bạn. Bài viết này chúng tôi xin giới thiệu về syslog.. Đầu tiên syslog là một ứng dung để ghi log của hệ thống, ta có thể hiểu đơn giản quá trình chúng ta ghi sổ nhật kí các hoạt động trong ngày vậy. Biết cách xem và sử dụng các bản tin log là một yêu cầu tối quan trọng đối với người quản trị hệ thống cần phải nắm được. 

Mục lục:

1. Giới thiệu
2. Các câu lệnh để hỗ trợ xem log
3. Chi tiết file cấu hình của log
4. Rotating Log
5. Log-server
6. Nâng cao với Syslog
7. Lời kết


##### 1. Giới thiệu

Syslog là một gói phần mềm trong hệ thống Linux nhằm để ghi bản tin log của hệ thống trong quá trình hoạt động như của kernel, deamon, cron, auth, hoặc các ứng dụng chạy trên hệ thống như http, dns, dhcp, ntp,..

Ứng dụng của log:
- Phân tích nguyên nhân gốc rễ của một vấn đề
- Giúp cho việc khắc phục sự cố nhanh hơn khi hệ thống gặp vấn đề
- Giúp cho việc phát hiện, dự đoán một vấn đề có thể xảy ra đối với hệ thống
- ...

Theo măc định các bản tin log của hệ thống được syslog lưu vào trong thư mục /var/log, và được lưu riêng rẽ đối với từng tác vụ trong hệ thông nhưng đối với tiến trình cron thì sẽ lưu trong file cron.log. Như hình dưới đây

<img class="image__pic js-image-pic" src="http://i.imgur.com/dFZUtfY.png" alt="" id="screenshot-image">

##### 2. Các câu lệnh hỗ trợ xem syslog

Đối với các file ghi log các bạn có thể dùng một số lệnh sau để giúp cho việc xem log

|Câu lệnh | Cú pháp |Ý nghĩa | Ghi chú thêm |
|---------|---------|--------|--------------|
|more | more [file] | Dùng xem toàn bộ nội dung của thư muc | Đối với câu lênh này nôi dung được xem theo từng trang. Bạn dung dấu "cách" để chuyển  trang |
|tail | tail [file] | In ra 10 dòng cuối cùng nội dung của file | thêm tùy chọn -n [số dòng] sẽ in ra số dòng theo yêu cầu |
|head | head [file] | In ra 10 dòng đầu tiên của nôi dụng file |
|tail -f | tail -f [file] | Dùng để xem ngay lâp tức khi có log đến | Đây là câu lệnh dùng phổ biến nhất nó giúp ta có thể xem ngay lập tức log mới đến, và nó sẽ in ra 10 dong cuối cùng trong nội dung file đó |

##### 3. Chi tiết file cấu hình của syslog
 
 File cấu hình của syslog:file cấu hình của syslog đôi với centos nằm trong thư mục `/etc/rsyslog.conf`
 ``` Đối với các phiên bản ubuntu thì mặc định nằm trong : `/etc/rsyslog.d/50-defaul.conf`
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
- Trường Seletor ( như ô vuông 1 trong hình ) : Chỉ ra nguồn tạo ra log và mức cảnh bảo của log đó. Trong trường seletor có 2 thành phần và được tách nhau bằng dấu "."
  - Nguồn tao ra log có thể liệt kê một số nguồn sau

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

- Trường action:( như ô vuông thứ 2 trong hình ) là trường để chỉ ra nơi lưu log của tiến trình đó.Có 2 loại là lưu tại file trong localhost hoặc gửi đến IP của Máy chủ Log ( chi tiết phần máy chủ log )

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

##### 4. Rotating Log

##### 5. Log-server

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

*note: @ IP máy chủ log* khi đó các log của mail cũng sẽ gửi đến ip của máy chủ log với port 514 và dùng giao thức UDP. Các bạn nhớ máy chủ log mở port 514 với kiểu truyền vận UPD hay TCP thì trên client cũng phải truyền đúng với giao thức như trên server.
*@IPserver:514* : Đối với giao thức UDP.
*@@IPserver:514* : Đối với giao thức TCP

##### 6. Nâng cao với syslog

Qua tìm hiểu chúng tôi nhận thức được kiến thức nâng cao hơn về phần syslog này như:
- Chỉnh sửa bản tin log đầu ra để đáp ứng yêu cầu công việc
- Cấu hình trên máy chủ log sao nó nó ghi thông tin đối với từng máy client một các riêng biệt và khoa học phục nhằm phục vụ cho việc kiểm tra log sau này
- Dùng chức năng rotating log tốt
- Ghi log lại đối với các dịch vụ như http, mysql, ntp,..
- Có thể ghi log vào hăn một cơ sở dữ liệu để tiện cho quá trình xem log
- Cơ chế mã hóa ssl đối với bạn tin log khi truyền trên mạng internet tránh trường hợp bị bắt gói tin
-...

Nhưng trong phạm vi bài viết này, chúng tôi chỉ giới hạn ở mức độ sơ khai nhất, các vấn đề nêu trên chúng tôi sẽ sớm làm rõ và đưa ra trong loạt các bài viết sau này về syslog.
##### 7. Lời kết

Bài viết trên đây của chúng tôi giới thiệu về syslog và những thứ cần thiết nhất khi tìm hiểu về syslog.Nó cũng là kiến thức mà khi tìm hiểu chúng tôi nhận được. Nắm được syslog là điều thực sự cần thiết đối với một người quản trị hệ thống. Nó là công cụ đắc lực nhất cho việc quan trị và sửa chữa hệ thống.

**Người thực hiện:**
+ Cao Ngọc Uy

Skype: uy.cao
+ Nguyễn Hoài Nam

Skype: namptit307

Tài liệu tham khảo:

[Link trang chủ của syslog](http://www.rsyslog.com/doc/master/index.html)

[Link 1](https://www.digitalocean.com/community/tutorials/how-to-view-and-configure-linux-logs-on-ubuntu-and-centos)

[Link 2](http://www.thegeekstuff.com/2011/08/linux-var-log-files/)

[Link 3](http://www.golinuxhub.com/2014/01/syslog-tutorial.html)

