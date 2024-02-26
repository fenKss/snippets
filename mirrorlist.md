Российские зеркала:
```sh
cat /etc/pacman.d/mirrorlist
```
```
Server = https://mirror.yandex.ru/archlinux/$repo/os/$arch
Server = https://mirror.kpfu.ru/archlinux/$repo/os/$arch
Server = https://mirror.yal.sl-chat.ru/archlinux/$repo/os/$arch
Server = https://mirror.truenetwork.ru/archlinux/$repo/os/$arch
Server = https://mirror.kamtv.ru/archlinux/$repo/os/$arch
```

Сформировать новые:
[Reflector](https://wiki.archlinux.org/title/reflector)
[Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/)