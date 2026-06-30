<p align="center">
  <a href="#-english-version">🇬🇧 English</a> •
  <a href="#-中文版本">🇨🇳 中文</a>
</p>

---

## 🇨🇳 中文版
本项目基于<a href="https://github.com/vergoh/vnstat">vnstat</a>进行修改，主要是用来监控云服务器的流量以免超标而产生不必要的资费，感谢原作者的开源贡献！
这我第一次fork，也是我第一次写project，算是新手，如果有任何任何问题大概是无法解答的，可以问问信任的AI们:)

### 部署
1. git源码至本地
```bash
git clone https://github.com/vergoh/vnstat
cd vnstat
```
2. 创建断网时执行的脚本
  选择一个目录（如/usr/local/bin/）下创建脚本文件，如下内容中用1.sh作为脚本名
```bash
sudo nano /usr/local/bin/1.sh
```
3. 编写脚本内容
  推荐用sudo nano命令，比较符合编辑逻辑
  Nano 编辑器中按 Ctrl+O 回车保存，Ctrl+X 退出
首先编写vnstat.config，记得删除参数前边的分号‘;’
```
sudo nano /cfg/vnstat.conf
# 1. 设置默认监控网卡（改为你通过 ip link 查到的真实网卡名，如 eth0 或 ens33）
Interface "eth0"

# 2.【可选】修改单位进制为 SI 十进制（kB/MB/GB，按 1000而非1024计算 留有余地）
UnitMode 2

# 3. 修改月流量结算起始日（假设流量每月 1 号重置，因人而异）
MonthRotate 1
```
保存并退出，重启服务使配置生效
```
sudo systemctl restart vnstat
```
A情况：禁用对应网卡实现断网（适用于服务器在你手边的情况）
  其中eth0是你通过ip link命令或ifconfig命令获取的目标网卡名、
```
#!/bin/bash
INTERFACE="eth0"
# vnstat 检查月流量是否超过 190 GB
vnstat -i $INTERFACE --alert 0 3 month total 190 GB
# 如果返回状态码 1，说明超标，执行断网
if [ $? -eq 1 ]; then
    # 记录日志
    echo "$(date): [方案A] 流量超标，正在禁用网卡 $INTERFACE..." >> /home/yushuai/vnstat_alert.log
    # 禁用网卡
    sudo /sbin/ip link set dev $INTERFACE down
fi
```
B情况：通过iptables封锁所有流量，并保留22号SSH端口（适用于云服务器纯白嫖情况）
```
#!/bin/bash
INTERFACE="eth0" # "eth0"为你的云服务器网卡名，因人而异
# vnstat 检查月流量是否超过设定流量 这里以190GB为例
vnstat -i $INTERFACE --alert 0 3 month total 190 GB
# 如果返回状态码 1，说明超标，执行
if [ $? -eq 1 ]; then
    # 日志
    echo "$(date):流量超标，正在通过 iptables 锁死流量并保留 22 端口..." >> /home/yushuai/vnstat_alert.log
    # 1. 允许本地回环网卡（127.0.0.1）通信，保证系统内部组件不崩溃
    sudo /sbin/iptables -A INPUT -i lo -j ACCEPT
    sudo /sbin/iptables -A OUTPUT -o lo -j ACCEPT
    # 2. 允许22 SSH的出入站流量
    sudo /sbin/iptables -A INPUT -p tcp --dport 22 -j ACCEPT
    sudo /sbin/iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
    # 3. 允许已经建立的、或者相关的连接继续通信（防止当前切断时你直接被踢出）
    sudo /sbin/iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
    sudo /sbin/iptables -A OUTPUT -m conntrack --ctstate ESTABLISHED -j ACCEPT
    # 4. 将默认策略改为 DROP（拒绝所有未被上面规则放行的流量）
    sudo /sbin/iptables -P INPUT DROP
    sudo /sbin/iptables -P OUTPUT DROP
    sudo /sbin/iptables -P FORWARD DROP
fi
```
4. 赋予脚本可执行权限
```shell
sudo chmod +x /usr/local/bin/1.sh
```
5. 配置 vnstat 用户免密 sudo 权限
```
sudo visudo
```
在最后一行添加以下内容：
```
vnstat ALL=(ALL) NOPASSWD: /etc/1.sh
```
6.设置自动执行
打开当前用户的自动任务编辑器
```
crontab -e
```
在最后一行添加如下内容：
```
*/5 * * * * /etc/1.sh
```


<p align="right">(<a href="#top">回到顶部</a>)</p>

---

## 🇬🇧 English Version

Welcome to my modified version! This project is forked from...

### Features
* Optimized performance
* Added automation scripts


<p align="right">(<a href="#top">back to top</a>)</p>

---
