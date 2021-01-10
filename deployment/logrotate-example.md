# 6.转储示例

{% hint style="success" %}
对应的[页面地址](https://github.com/dani-garcia/bitwarden_rs/wiki/Logrotate-example)
{% endhint %}

随着时间的流逝，Bitwarden\_RS 日志文件的大小可能会增长到很大。使用 logrotate，我们可以定期转储日志。

```text
sudo nano /etc/logrotate.d/bitwarden
```

```python
/var/log/bitwarden/*.log {
    # Perform logrotation as the bitwarden user and group
    su bitwarden bitwarden
    # Rotate daily
    daily
    # Rotate when the size is bigger than 5MB
    size 5M
    # Compress old log files
    compress
    # Keep 4 rotations of log files before removing or mailing to the address specified in a mail directive
    rotate 4
    # Truncate the original log file in place after creating a copy
    copytruncate
    # Don't panic if not found
    missingok
    # Don't rotate log if file is empty
    notifempty
    # Add date instaed of number to rotated log file
    dateext
    # Date format of dateext
    dateformat -%Y-%m-%d-%s
}
```

无需手动解压缩而查看压缩的日志文件：

```python
zcat logfile.gz
zless logfile.gz
zgrep -i keyword_search logfile.gz
```

