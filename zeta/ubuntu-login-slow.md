---
title: 'ubuntuのSSHログインが遅い'
emoji: 😇
type: tech
topics: [ubuntu,raspberry pi4,ssh]
published: true
---
## Raspberry Pi4を使ってるけど、OSはUbuntu  22.04.4 LTS
一家に一台、Linuxサーバが家にあると便利。  
OSはRaspberry PiOSじゃなくて、素のUbuntu。  

sshログインする時にたまーに遅くなる現象。  
20秒ぐらいなので確かに遅いのだけど我慢はできなくもないという微妙な間。  
なんとかならんかと思い、
調べてみたらあっさり解決できたので備忘録的にメモとして残しとく。  

## 原因：sshログイン時の名前解決に時間がかかっている
参考：[【Linux】SSH接続に時間がかかる場合の対処法【備忘】](https://gametech.vatchlog.com/2019/02/13/ssh-timewait/)

どうやらSSH接続する時にクライアントIPを名前解決しようとしているよう。  
ということであれば、hostsファイルに逆引きレコードを書くか、  
sshd_configにDNSを使わないように設定を変える必要がある。  

今回は後者のsshd_config変更をやってみた。  

## sshd_config変更
sshd_configを編集する。  
```
sudo vi /etc/ssh/sshd_config
```

`UseDNS no`がコメントになっているので、コメントアウトをするだけ。

変更後はsshd再起動。

```
sudo service sshd restart
```

以上！  


## おまけ:Amazon Linux 2023などはデフォルトでUseDNS noになっている
それだけと味気ないので、AWS(Amazon Linux 2023)ではどうなってるか調べてみたｗ  

https://docs.aws.amazon.com/ja_jp/linux/al2023/ug/compare-with-al2.html#ssh-host-key

>さらに、**デフォルトの sshd_config ファイルの AL2023 構成設定には UseDNS=no が含まれます。この新しい設定により、DNS 障害によってインスタンスとの ssh セッションを確立できなくなる可能性が低下します。**
>その対価として、authorized_keys ファイル内の from=hostname.domain,hostname.domain ラインのエントリが解決されなくなります。
>sshd が DNS 名を解決しようとしなくなるため、カンマで区切られた hostname.domain のそれぞれの値を IP address に対応する値に変換する必要があります。


というわけで、DNS無効化で良さげ。  
その代わり上記のような制約もあるので注意。  

ま、個人で使う分には全然問題なし。  
