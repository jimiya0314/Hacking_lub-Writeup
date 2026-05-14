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
|  |  |
|-----------|---------|
| -sS | デフォルトカテゴリのスクリプトでスキャン |
| -A | OSとサービスの詳細表示|
| --system-dns | システムのDNSリゾルバを使う |
| -oX | スキャン結果をXMLファイルにも出力する |

**22,80,443**が開いていた
5.Webスクリーンショットの連続撮影（EyeWitnessツール使用）
```bash
eyewitness -x scan.xml --web
```
<img width="1122" height="454" alt="image" src="https://github.com/user-attachments/assets/ade47c43-ee22-403a-ac0a-b847b9c40386" />
`http://192.168.56.120`
を押す

6.HTTPアクセスする
<img width="1391" height="513" alt="image" src="https://github.com/user-attachments/assets/00061872-acfe-4161-af38-ff80b4ee7c6a" />

これらのコマンドのみが使えるサイト
prepare：動画が再生
fsociety：メッセージ表示
inform：記事のスクラップとMr.Robotからのメッセージ
question：画像が表示
wakeup：動画が再生
join：入会案内
help：使用できるコマンドの一覧表示

7.ブログにアクセスする

`http://192.168.56.120` ⇒　さっきのコマンドどれ指定しても同じページに移行した

<img width="1397" height="691" alt="image" src="https://github.com/user-attachments/assets/c199df23-559a-434d-9575-aeee7383b06b" />
ページのソースを確認（いくつかのディレクトリらしきものがみられる）
<img width="1401" height="696" alt="image" src="https://github.com/user-attachments/assets/d9a31c2b-4072-4f73-be6b-3d61090fc74c" />

8.robots.txtにアクセスしてみる

`http://192.168.56.120/robots.txt`

２つファイルを発見
<img width="1175" height="312" alt="image" src="https://github.com/user-attachments/assets/9d1b07cb-c7e5-49a1-82e2-edbc3c0bda6b" />

9.怪しいURLにアクセスする

`wget http://192.168.56.120/key-1-of-3.txt`
`wget http://192.168.56.120/fsociety.dic`

ダウンロードしたファイルをcatで開く
fsociety.dicが大きいのでパスワード候補に重複がないか確かめる
```bash
$sort fsocity.dic | uniq -c
75 000
75 000000
75 000080
75 001
75 002
75 003
75 0032
75 003s
ーーーーーーーーーーー　（　略　）ーーーーーーーーーーーーーーー
※全部のワードが７５ずつ重複しているのを確認
```
重複を省いたパスワードファイルを作成する
```bash
cat fsociety.dic | sort | uniq > filter_fsociety.dic
```
10.HTTPサービスの情報を収集する
```bash
nikto -h http://192.168.56.120:80
```
```
Nikto v2.5.0

---

Target IP: 192.168.56.114

Target Hostname: 192.168.56.114

Target Port: 80

Start Time: 2026-01-03 10:26:45 (GMT9)

---

Server: Apache

/: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/

/esE7X7bk.cfm: Retrieved x-powered-by header: PHP/5.5.29.

No CGI Directories found (use '-C all' to force check all possible dirs)

/index: Uncommon header 'tcn' found, with contents: list.

/index: Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. The following alternatives for 'index' were found: index.html, index.php. See: http://www.wisec.it/sectou.php?id=4698ebdc59d15,https://exchange.xforce.ibmcloud.com/vulnerabilities/8275

/admin/: This might be interesting.　　　　←　怪しい

/readme: This might be interesting.　　　　←　怪しい

/image/: Drupal Link header found with value: http://192.168.56.114/?p=23; rel=shortlink. See: https://www.drupal.org/

/wp-links-opml.php: This WordPress script reveals the installed version.

/license.txt: License file found may identify site software.　　　←　怪しい

/admin/index.html: Admin login page/section found.　　　　　←　怪しい

/wp-login/: Cookie wordpress_test_cookie created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies　　←　怪しい

/wp-login/: Admin login page/section found.

/wordpress/: A Wordpress installation was found.　　　　　　　　←　怪しい

/wp-admin/wp-login.php: Wordpress login found.

/wordpress/wp-admin/wp-login.php: Wordpress login found.

/blog/wp-login.php: Wordpress login found.

/wp-login.php: Wordpress login found.

/wordpress/wp-login.php: Wordpress login found.

/#wp-config.php#: #wp-config.php# file found. This file contains the credentials.

8102 requests: 0 error(s) and 19 item(s) reported on remote host

End Time: 2026-01-03 10:29:42 (GMT9) (177 seconds)

---

1 host(s) tested
```
```
http://192.168.56.120/admin/index.html

##################中身（ページソース）###################
<!doctype html>
<!--
\   //~~\ |   |    /\  |~~\|~~  |\  | /~~\~~|~~    /\  |  /~~\ |\  ||~~
 \ /|    ||   |   /__\ |__/|--  | \ ||    | |     /__\ | |    || \ ||--
  |  \__/  \_/   /    \|  \|__  |  \| \__/  |    /    \|__\__/ |  \||__
-->
<html class="no-js" lang="">
  <head>
    

    <link rel="stylesheet" href="css/A.main-600a9791.css.pagespeed.cf.6PBOtigNA6.css">

    <script src="js/vendor/vendor-48ca455c.js.pagespeed.jm.V7Qfw6bd5C.js"></script>

    <script>var USER_IP='208.185.115.6';var BASE_URL='index.html';var RETURN_URL='index.html';var REDIRECT=false;window.log=function(){log.history=log.history||[];log.history.push(arguments);if(this.console){console.log(Array.prototype.slice.call(arguments));}};</script>

  </head>
  <body>
    <!--[if lt IE 9]>
      <p class="browserupgrade">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
    

    <!-- Google Plus confirmation -->
    <div id="app"></div>

    
    <script src="js/s_code.js.pagespeed.jm.I78cfHQpbQ.js"></script>
    <script src="js/main-acba06a5.js.pagespeed.jm.YdSb2z1rih.js"></script>
</body>
</html>
```
http://192.168.56.114/license.txt

ページがスクロールできる

一番下にエンコード文字列を発見
```
what you do just pull code from Rapid9 or some s@#% since when did you become a script kitty?

do you want a password or something?

ZWxsaW90OkVSMjgtMDY1Mgo=
```

※デコードする
```bash
echo 'ZWxsaW90OkVSMjgtMDY1Mgo=' | base64 --decode
```
`elliot:ER28-0652` 　ユーザ：パスワード

`http://192.168.56.120/wp-login.php`
<img width="1401" height="768" alt="image" src="https://github.com/user-attachments/assets/910ebe33-df15-41a3-8c79-0752ec2d4724" />
<img width="356" height="506" alt="image" src="https://github.com/user-attachments/assets/67a95fda-81f6-49d0-a49e-54955707a5b1" />
<img width="1396" height="735" alt="image" src="https://github.com/user-attachments/assets/a85718f7-074c-48af-971f-0dd5bf2f0623" />

ログイン成功

11.Burpで認証時のHTTPリクエストを調べる
まず、Proxy＞Intercept＞Intercept is off　にしておく

Burpで不具合。

認証画面から始める（Intercept is on）

１．ユーザー「test」パスワード「test」で実行　⇒　HTTPリクエストをみる

２．forwardボタンをおす　⇒　ERROR:Invaild usernameが返ってくる

３．testユーザーというものは存在しない　

12.
