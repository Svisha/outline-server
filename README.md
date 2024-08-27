# Outline Server

![Build and Test](https://github.com/Jigsaw-Code/outline-server/actions/workflows/build_and_test_debug.yml/badge.svg?branch=master) [![Mattermost](https://badgen.net/badge/Mattermost/Outline%20Community/blue)](https://community.internetfreedomfestival.org/community/channels/outline-community) [![Reddit](https://badgen.net/badge/Reddit/r%2Foutlinevpn/orange)](https://www.reddit.com/r/outlinevpn/)

Outline Server is the component that provides the Shadowsocks service (via [outline-ss-server](https://github.com/Jigsaw-Code/outline-ss-server/)) and a service management API. You can deploy this server directly following simple instructions in this repository, or if you prefer a ready-to-use graphical interface you can use the [Outline Manager](https://github.com/Jigsaw-Code/outline-apps/).

**Components:**

- **Outline Server** ([`src/shadowbox`](src/shadowbox)): The core proxy server that runs and manages [outline-ss-server](https://github.com/Jigsaw-Code/outline-ss-server/), a Shadowsocks backend. It provides a REST API for access key management.

- **Metrics Server** ([`src/metrics_server`](src/metrics_server)): A REST service for optional, anonymous metrics sharing.

**Join the Outline Community** by signing up for the [IFF Mattermost](https://wiki.digitalrights.community/index.php?title=IFF_Mattermost)!

## Shadowsocks and Anti-Censorship

Outline's use of Shadowsocks means it benefits from ongoing improvements that strengthen its resistance against detection and blocking.

**Key Protections:**

- **AEAD ciphers** are mandatory.
- **Probing resistance** mitigates detection techniques.
- **Protection against replayed data.**
- **Variable packet sizes** to hinder identification.

See [Shadowsocks resistance against detection and blocking](docs/shadowsocks.md).

## Installation

**Prerequisites**

- [Node](https://nodejs.org/en/download/) LTS (`lts/hydrogen`, version `18.16.0`)
- [NPM](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) (version `9.5.1`)
- [Go](https://go.dev/dl/) 1.21+

1. **Install dependencies**

   ```sh
   npm install
   ```

1. **Start the server**

   ```sh
   ./task shadowbox:start
   ```

   Exploring further options:

   - **Refer to the README:** Find additional configuration and usage options in the core server's [`README`](src/shadowbox/README.md).
   - **Learn about the build system:** For in-depth build system information, consult the [contributing guide](CONTRIBUTING.md).

1. **To clean up**

   ```sh
   ./task clean
   ```
# [Статья переехала на ypermitin.github.io](https://ypermitin.github.io/devoooops/2022/03/30/Установка-Outline-VPN-на-Ubuntu-20.04.html)

# Установка Outline VPN на Ubuntu 20.04

Outline VPN - это бесплатный инструмент с открытым исходным кодом, позволяющий развернуть собственную VPN на Вашем собственном сервере или на машине облачного провайдера. Подробную информацию Вы можете узнать [здесь](https://getoutline.org/ru/) и [здесь](https://en.wikipedia.org/wiki/Outline_VPN).

В своем составе имеет как графические инструменты, так и средства работы через командную строку. Позволяет использовать VPN как на настольных компьютерах, так и на мобильных устройствах.

## Прежде чем начать

Вам нужен сервер. Да, его нужно арендовать, учитывая его местоположение. Например, если Вам нужно получать доступ к ресурсам, которые недоступны в текущем местоположении, но доступны, например, в Канаде, то смело арендуйте виртуальную машину в AWS, Digital Ocean или любом другом месте.

Для простых задач, например открытие сайтов, обмена текстовыми сообщениями в мессенджерах и т.д., подойдет хост с минимальными ресурсами:

* 1 ядро CPU.
* 1 ГБ RAM (можно и меньше).
* 10 GB HDD для файлов ОС в основном.

Стоимость аренды такого сервера от 3$ до 5$ в месяц, плюс-минус. По необходимости ресурсов можно добавлять поболее. Операционная система - Ubuntu 20.04, т.к. инструкция именно для нее :)

Как арендовать виртуальную машину тут рассматривать не будем, все зависит от провайдера.

## Начальная настройка сервера

И так, виртуальная машина арендована, доступ к ней по SSH имеется, Ubuntu установлена. Далее устанавливаем последние обновления.

```bash
sudo apt update
sudo apt upgrade
```

Сделайте перезагрузку машины после этого, если есть необходимость после установки обновлений.

Сразу настроим брэндмауэр, чтобы защитить машину от несанкционированного доступа. Открываем доступ через SSH.

```bash
sudo ufw allow 22/tcp
```

Если у Вас статический IP, то для безопасности доступ по SSH можно разрешить только для него.

```bash
sudo ufw allow from <ВашПостоянныйIP> to any port 22
```

И включаем брэндмауэр.

```bash
sudo ufw enable
```

Отлично! Машина защищена. Идем дальше.

## Установка Outline Server

Для установки воспользуемся готовыми скриптами из проекта [outline-server](https://github.com/Jigsaw-Code/outline-server) компании Jigsaw.

Скрипт находится по адресу:

```
https://github.com/Jigsaw-Code/outline-server/blob/master/src/server_manager/install_scripts/install_server.sh
```

Для установки достаточно выполнить следующую команду.

```bash
sudo wget -qO- https://raw.githubusercontent.com/Jigsaw-Code/outline-server/master/src/server_manager/install_scripts/install_server.sh | bash
```

Будет установлен Docker и службы самого Outline, а также все зависимости. При необходимости Вы можете установить Docker самостоятельно перед запуском скрипта.

```bash
sudo curl https://get.docker.com | sh
```

Когда скрипт закончит, то выведет примерно такое содержимое.

```
{ 
  "apiUrl": "https://0.0.0.0:0000/XXXXXXXXXXXX", 
  "certSha256": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX" 
}
```

Также в консольном выводе будет информация о портах, которые нужно открыть для работы OutlineVPN Manager. Добавляем их в разрешенные (замените только на свои значения).

```bash
sudo ufw allow 31111/tcp
sudo ufw allow 24311/tcp
```

Сохраните это себе на будущее. По факту сервер Outline VPN уже установлен и нам лишь нужно его настроить для своих нужд.

## Клиент для управления сервером

Управление серверном VPN, в т.ч. раздача доступов, осуществляется с помощью [Outline Manager](https://getoutline.org/ru/get-started/#step-1), доступной для Windows, Max и Linux.

При запуске нужно добавить сервер и выбрать "Настроить Outline где угодно". Появится инструкция по установке с помощью скрипта, который мы ранее запускали. А после поле для ввода ключа и адреса, который Вы до этого сохранили.

После этого у Вас появится доступ к управлению сервером.

## Добавляем ключ

В Outline Manager добавляем новый ключ в управлении сервером. Программа покажет [ссылку на инструкцию](https://github.com/Jigsaw-Code/outline-client/blob/master/docs/invitation-instructions.md) и сам ключ в виде строки:

```bash
ss://XXXXXXXXXXXX@9.9.9.9:0/?outline=1
```
Скопируйте этот ключ, он понадобиться при запуске клиента Outline.

## Клиент для подключения

И последний шаг - установка [клиента для подключения](https://getoutline.org/ru/get-started/#step-3). Есть приложения для Android, Windows, Chrome, iOS, MacOS, Linux.

При первом запуске нужно нажать "Добавить сервер" и вставить полученный выше ключ.

Готово!

## Послесловие

Теперь Вы можете использовать Outline VPN. Надеюсь, что Вы будете его использовать без злого умысла.

## Полезные ссылки

* [Официальный сайт](https://getoutline.org/ru/how-it-works/)
* [Официальный репозиторий на GitHub](https://github.com/outline/outline)
* [Официальная документация](https://app.getoutline.com/share/770a97da-13e5-401e-9f8a-37949c19f97e/doc/hosting-outline-nipGaCRBDu)
* [Outline: Делаем свой личный VPN от Google за 5$ в месяц (и за 1€ для продвинутых)](https://habr.com/ru/post/358828/)
* [How to setup an Outline VPN Server on Ubuntu 16.04](https://gist.github.com/okeehou/275bb83601be346921622e05f13cba70)