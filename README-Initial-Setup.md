# code-serverの初期設定スクリプトに関して

このスクリプトは[launch-code-server.sh](https://github.com/coder/deploy-code-server/blob/main/deploy-vm/launch-code-server.sh)と[Coder Usage](https://coder.com/docs/code-server/guide)を基にしていますが、2024年現在のAmazon Linux 2023環境で動作する様にカスタマイズしています。

UserDataで指定するので`root`ユーザーで動作する前提です。  

```bash
#!/usr/bin/bash

# Add aws_completer
echo 'complete -C /usr/bin/aws_completer aws' >> /home/ec2-user/.bashrc

# Install expect, git
dnf install -y expect git

# Install code-server service system-wide
export CODER_VERSION=$(curl -s https://api.github.com/repos/coder/code-server/releases/latest | jq .tag_name --raw-output | sed 's/v//')
curl -fOL https://github.com/coder/code-server/releases/download/v$CODER_VERSION/code-server-$CODER_VERSION-amd64.rpm
rpm -i code-server-$CODER_VERSION-amd64.rpm
rm -f code-server-$CODER_VERSION-amd64.rpm

# Setup code-server@ec2-user configurations
mkdir -p /home/ec2-user/.config/code-server/
touch /home/ec2-user/.config/code-server/config.yaml
echo "bind-addr: 0.0.0.0:443" > /home/ec2-user/.config/code-server/config.yaml
echo "auth: password" >> /home/ec2-user/.config/code-server/config.yaml
echo "password: $(mkpasswd-expect -l 32 -s 0 -d 8 -C 8)" >> /home/ec2-user/.config/code-server/config.yaml
echo "cert: true" >> /home/ec2-user/.config/code-server/config.yaml
chown -R ec2-user:ec2-user /home/ec2-user/.config
# Allows code-server to listen on port 443.
setcap cap_net_bind_service=+ep /usr/lib/code-server/lib/node

# start and enable code-server 
systemctl enable --now code-server@ec2-user
```
