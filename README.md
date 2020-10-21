# gitlab

自己証明書によるローカル環境用CI/CD設定です。Mac(MacOS Mojave 10.14.6)で動作確認しています。

[Docker socket binding](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html#use-docker-socket-binding)を使用しています。

# Requirement

* Docker Engine 19.03.13

# Usage

自己証明書を発行します。以下はホストのローカルIP例として192.169.10.5を指定しています。
```bash
openssl req -newkey rsa:4096 -days 365 -nodes -x509 -subj "/C=JP/ST=AICHI/L=AICHI/O=NAN/OU=NAN/CN=192.168.10.5" -extensions v3_ca -config <( cat /System/Library/OpenSSL/openssl.cnf <(printf "[v3_ca]\nsubjectAltName='IP:192.168.10.5'")) -keyout server.key -out server.crt
```

生成した証明書ファイル、秘密鍵ファイルを、gtlab/config/sslと、gitlab-runner/config/certsに配置します。
```bash
cp server.crt gitlab/config/ssl/192.168.10.5.crt
cp server.crt gitlab-runner/config/certs/192.168.10.5.crt
cp server.key gitlab/config/ssl/192.168.10.5.key
cp server.key gitlab-runner/config/certs/192.168.10.5.key
```

docker用に証明書ファイル、秘密鍵ファイルをそれぞれclient.cert、client.keyとして配置します。  
[クライアント証明書の追加](https://matsuand.github.io/docs.docker.jp.onthefly/docker-for-mac/#add-client-certificates)

docker-composeではgitリポジトリ用ポートとして8001を、レジストリ用ポートとして5050を指定しています。  
以下でgitlabを起動し、ログにて完全に起動完了確認後、`https://192.168.10.5:8001`のようにホストのIPアドレス指定でアクセスします。
```bash
docker-compose up -d gitlab
```

runner登録トークン確認後、gitlab-runner registerを使い、既存のconfig.tomlを削除、再生成します。  
再生成後以下パラメータとし、runner起動します。
```bash
privileged = true
volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/certs/client", "/cache"]
```

# Note

Windows環境ではテストしていません。