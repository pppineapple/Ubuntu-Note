## 抓取接入ip地址


shell脚本

```
#!/usr/bin/bash
netstat -an | awk '{print $5}' | sort | uniq  | cut -d ':' -f 1 >> /home/pineapple/shell_scrpts/tmp.log
cat /home/pineapple/shell_scrpts/tmp.log |sort| uniq > /home/pineapple/shell_scrpts/foreign_address.log
cat /home/pineapple/shell_scrpts/foreign_address.log > /home/pineapple/shell_scrpts/tmp.log
```

制作定时任务

```
crontab -e
* * * * * /home/pineapple/shell_scrpts/get_foreign_address.sh 
```