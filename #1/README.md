1.IPアドレスを調べる
```bash
$sudo ifconfig
```
2.DC-1マシンのIPアドレスを特定する
  Pingスイープ：端末が存在するかどうかを調べるために、Pingで広域スキャンすること
```bash
$sudo fping -aqg 192.168.56.0/24
```
![kekka](images/kekka.png)
