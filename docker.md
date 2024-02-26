# Docker
```shell
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker containerd
```
```shell
sudo groupadd docker;
sudo usermod -aG docker $USER;
newgrp docker;
```