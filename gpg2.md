# Подписывание коммитов gpg ключом

## Установка пакетов

```shell
sudo pacman -S gnupg gcr
```



## Генерация ключа
Имя и Email должны совпадать в git и в ключе <br>
[Инструкция](https://docs.gitlab.com/ee/user/project/repository/signed_commits/gpg.html)

## Настройка git
```
git config --global user.name "Mihail Syursin"
git config --global user.email "mixail.syursin@gmail.com"
git config --global commit.gpgsign true
```

## Pinentry
```shell
nano ~/.gnupg/gpg-agent.conf
```
```
pinentry-program /usr/bin/pinentry-gnome3
```
