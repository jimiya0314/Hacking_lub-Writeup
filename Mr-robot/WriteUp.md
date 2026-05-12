1.Mr-robotのIPアドレスを特定する（今回はarp-scan）
```bash
sudo arp-scan -l
```
```
Interface: enp0s3, type: EN10MB, MAC: 08:00:27:39:c7:bb, IPv4: 192.168.56.101
Starting arp-scan 1.10.0 with 256 hosts (https://github.com/royhills/arp-scan)
192.168.56.1	0a:00:27:00:00:0a	(Unknown: locally administered)
192.168.56.100	08:00:27:0d:df:51	PCS Systemtechnik GmbH
192.168.56.114	08:00:27:e2:32:c5	PCS Systemtechnik GmbH
```

2.疎通確認
```bash
sudo ping 192.168.56.114
```

3.実験用ディレクトリの作成
```bash
cd vulnhub/
mkdir mr-robot
cd mr-robot/
```

4.ポートスキャンする
```bash
sudo nmap -sS -A 192.168.56.120 --system-dns -oX scan.xml
```
```
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache
443/tcp open   ssl/http Apache httpd
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:BC:54:D9 (Oracle VirtualBox virtual NIC)
```
