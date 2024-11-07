
wsl 창에서
```
ip addr show eth0 | grep "inet\b" | awk '{print $2}' | cut -d/ -f1
```


fail2ban
```
[sshd]
enabled = true
port = 22 # SSH 포트를 변경했다면 그에 맞게 수정
filter = sshd
logpath = /var/log/auth.log
maxretry = 5
findtime = 300
bantime = 3600
ignoreip = 127.0.0.1/8 ::1 your_github_actions_ip your_team_ip1 your_team_ip2
~                                                                                                 
```
