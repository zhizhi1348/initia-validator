## Install Oracle


```
cd $HOME
rm -rf slinky
```
```
git clone https://github.com/skip-mev/slinky.git
```
```
cd slinky
```
```
git checkout v0.4.3
```
```
make build
```
```
mv build/slinky /usr/local/bin/
```

## Create Oracle Service file

```ruby
sudo tee /etc/systemd/system/slinkyd.service > /dev/null <<EOF
[Unit]
Description=Initia Slinky Oracle
After=network-online.target

[Service]
User=$USER
ExecStart=$(which slinky) --oracle-config-path $HOME/slinky/config/core/oracle.json --market-map-endpoint 127.0.0.1:15090
Restart=on-failure
RestartSec=30
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Start Oracle SystemD

```
sudo systemctl daemon-reload
sudo systemctl enable slinkyd
sudo systemctl restart slinkyd
```

## Check Logs 

```
journalctl -fu slinkyd --no-hostname
```
***press Ctl + C to exit***

```
nano /root/.initia/config/app.toml
```

**Scroll down to the end of the file , and replace the placeholders with the following codes as shown in the below images.**

```
enabled = "true"
oracle_address = "127.0.0.1:8080"
client_timeout = "500ms"
```

**Now Restart Initia SystemD**
```
sudo systemctl daemon-reload
sudo systemctl restart initiad
sudo systemctl restart slinkyd
sudo journalctl -u initiad -f --no-hostname -o cat
```

![Before](https://github.com/zhizhi1348/initia-validator/blob/main/331811143-a0d264ad-2f40-4ebc-bc9f-0920246a83b8.png)
![After](https://github.com/zhizhi1348/initia-validator/blob/main/331811146-d3e03408-011b-4580-9ddd-050c54bcfabd.png)
