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
