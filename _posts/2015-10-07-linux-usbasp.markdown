---
layout: post
title:  "Настройка прав для подключения программатора USBasp в Linux Mint 17"
date:   2015-10-07 11:20:50
categories: avr usbasp linux
---
При попытке прошивки attiny13 из Arduino IDE (Linux Mint 17.2) при помощи [USBasp](https://rover.ebay.com/rover/1/711-53200-19255-0/1?icep_id=114&ipn=icep&toolid=20004&campid=5338190330&mpre=http%3A%2F%2Fwww.ebay.com%2Fitm%2FUSBASP-USBISP-AVR-Programmer-Adapter-10-Pin-USB-Cable-ATMEGA8-ATMEGA128-Arduino-%2F141924793771) возникла ошибка:

```
avrdude: Warning: cannot open USB device: Permission denied
avrdude: error: could not find USB device with vid=0x16c0 pid=0x5dc
```

Очевидно, что у текущего пользователя нет прав на использование устройства.
Рецепты по настройке нашлись в интернете, но помогло далеко не первое руководство.

Поэтому, выкладываю.

Сперва, отключаем USBasp и смотрим на какое устройство сел USBasp (**16c0** это **vid** из сообщения об ошибке):

```
$ lsusb | grep 16c0
Bus 003 Device 024: ID 16c0:05dc Van Ooijen Technische Informatica shared ID for use with libusb
```

Посмотрим **Bus 003** (у вас номер может быть другим):

```
$ ls -la /dev/bus/usb/003/
total 0
drwxr-xr-x 2 root root      180 Oct  7 11:07 .
drwxr-xr-x 8 root root      160 Oct  7 06:32 ..
crw-rw-r-- 1 root root 189, 256 Oct  7 06:32 001
crw-rw-r-- 1 root root 189, 257 Oct  7 06:32 002
crw-rw-r-- 1 root root 189, 258 Oct  7 06:32 003
crw-rw-r-- 1 root root 189, 259 Oct  7 06:32 004
crw-rw-r-- 1 root root 189, 260 Oct  7 06:32 005
crw-rw-r-- 1 root root 189, 261 Oct  7 06:32 006
crw-rw-r-- 1 root root 189, 279 Oct  7 11:07 024
```

Обратите внимание, у устройства 024 владелец и группа - root root.
Наша задача - добиться, чтобы у USBasp была группа, в которую входит текущий пользователь.

Создаем файл usbasp.rules со следующим содержимым:

```
ATTR{idVendor}=="16c0", ATTR{idProduct}=="05dc", MODE:="0664", GROUP:="<your group>"
```

Проще всего указать группу, одноименную с именем вашего пользователя. Если хочется другую, можно посмотреть командой **groups**, в каких группах состоит пользователь.

Далее копируем файл:

```
sudo cp usbasp.rules /etc/udev/rules.d/
```

udev сам подхватывает изменения, ничего рестартить не надо.

Отключаем и подключаем программатор.
Смотрим шину (повторюсь, у вас номер может быть другим):

```
$ ls -la /dev/bus/usb/003/
total 0
drwxr-xr-x 2 root root       180 Oct  7 11:15 .
drwxr-xr-x 8 root root       160 Oct  7 06:32 ..
crw-rw-r-- 1 root root  189, 256 Oct  7 06:32 001
crw-rw-r-- 1 root root  189, 257 Oct  7 06:32 002
crw-rw-r-- 1 root root  189, 258 Oct  7 06:32 003
crw-rw-r-- 1 root root  189, 259 Oct  7 06:32 004
crw-rw-r-- 1 root root  189, 260 Oct  7 06:32 005
crw-rw-r-- 1 root root  189, 261 Oct  7 06:32 006
crw-rw-r-- 1 root wayne 189, 280 Oct  7 11:15 025
```

Как видим, программатор сел на устройство с новым номером 025 и имеет уже группу моего пользователя wayne.
В Arduino IDE все прекрасно шьется.

P.S. Еще в linux по умолчанию не хватает прав для заливки скетчей в ардуино стандартным способом.
USB-to-Serial адаптер ардуино присоединяется с правами группы dialout.
Надо добавить себя в эту группу и все будет работать:

```
sudo adduser <your user> dialout
```
