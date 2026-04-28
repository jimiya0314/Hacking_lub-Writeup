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
```
## まだあるよ
```
oh no
```
![kekka](images/kekka.png)
