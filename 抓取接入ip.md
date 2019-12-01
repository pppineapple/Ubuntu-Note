## 抓取接入ip地址


shell脚本

```
#!/usr/bin/bash
netstat -an | awk '{print $5}' | sort | uniq >> ./tmp.log
cat ./tmp.log |sort| uniq > ./foreign_address.log
cat foreign_address.log > tmp.log
```

制作定时任务

```
crontab -e
* * * * * /home/pineapple/shell_scrpts/get_foreign_address.sh 
```