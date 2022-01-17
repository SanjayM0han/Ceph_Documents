### Installing main host

```
sudo apt install -y curl
```

```
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/pacific/src/cephadm/cephadm
chmod +x cephadm
```

```
sudo ./cephadm add-repo --release pacific
sudo ./cephadm install
```

```
sudo cephadm bootstrap --mon-ip 192.168.6.98
```

### Preparing additional hosts.

```
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

```
sudo apt install -y lvm2
```
