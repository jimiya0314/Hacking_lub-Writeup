# python３の簡易HTTPサーバーとPHPの関係
1.maliciousファイルを作成してみる
```
echo '<?php system("cat /etc/passwd");?>' > malicious.php
```
| <?php?> | PHPコードであることを表す |
| system(): | サーバ上でコマンドを実行する命令 |
| cat /etc/passwd | ファイルを閲覧する |
2.簡易HTTPサーバーを立ち上げる
```
sudo python3 -m http.server
```
- ※maliciousファイルが存在するディレクトリで行う
3.ブラウザからmaliciousファイルにアクセスする
```
http://localhost:8000/malicious.php
```
4.maliciousファイルを保存するか否かの挙動が起こり、PHPコード自体は実行されない<br>
なぜか…<br>
pythonのhttp.serverは、ただファイルを「提供する」のみ<br>
HTMLや画像などの静的なファイルを扱うことはできるがＰＨＰコードを実行する機能をもっていない<br>			
PHPを動かすにはApacheやNginxのようにPHPを解釈するための特別な設定がされたウェブサーバーが必要<br><br>
それを踏まえてＰＨＰを実行したい！！
 - php_server.pyファイル作成（ソースコード）
   ```
   import http.server
   import os
   import subprocess
   from urllib.parse import unquote

   class PHPRequestHandler(http.server.SimpleHTTPRequestHandler):
   def do_GET(self):
   if self.path.endswith(".php"):
   self.send_response(200)
   self.send_header("Content-Type", "text/html")
   self.end_headers()
   path = unquote(self.path[1:])
   output = subprocess.check_output(["php", path])
   self.wfile.write(output)
   else:
   super().do_GET()

   if name == "main":
   http.server.test(HandlerClass=PHPRequestHandler)
   ```
- pythonの簡易httpseverで待機
  ```
  python3 php_server.py
  ```
- ブラウザにアクセス
  ```
  http://localhost:8000/malicious.php
  ```
![kekka](images/kekka.png)
**etc/passwdファイルが出力されている				→PHPファイルが動作している**
- curlコマンドでも実行可能
  ```
  curl -X GET http://localhost:8000/malicious.php
  ```
