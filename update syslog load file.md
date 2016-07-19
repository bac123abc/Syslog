* Rsyslog có một tính năng khá hay, đó là hỗ trợ load module có thể đọc file log bất kỳ trong hệ thống.
* Ví dụ đọc log error của apache:
```
# Load file  1 apache error
$ModLoad imfile
$InputFileName /var/log/httpd/error_log
$InputFileTag apache-error:
$InputFileStateFile state-apache-error
$InputRunFileMonitor
```

* Giải thích các tham số:
