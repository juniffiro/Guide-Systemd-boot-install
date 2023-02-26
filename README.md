# 🚀 Systemd-boot install Guide
Установка простого загрузчика ОС

*Небольшой Guide для удобства.*

*Общие понятия и сокращения <br/>*
**EFI** - системный раздел FAT32, откуда происходит запуск
загрузчика и приложения EFI (Bootloader'а) <br/>
Также именуется как **ESP** или **EFISYS**.

Для установки нужно сначала примонтировать диск с EFI разделом.
Список дисков можно посмотреть воспользовавшись утилитой *GParted*
либо командой

```
sudo fdisk -l
```

Находим EFI раздел. В моем случае это */dev/sdc1*
```
sudo mount /dev/sdc1 /boot
```

## Простая установка systemd-boot в EFI
```
sudo bootctl install
```
Это действие найдет EFI по адресу */efi* или */boot* и скопирует раздел *EFI systemd-boot* в него.

Если EFI не примонтирован в /boot, опцией --path= можно явно указать точку монтирования, например:

```
sudo bootctl --path=/efi
```

> Поскольку, мы смонтировали EFI по адресу /boot, systemd-boot будет установлен именно там.

> Обратите внимание!\
> Установка systemd-boot перезапишет все существующие esp/EFI/BOOT/BOOTX64.EFI
>
> **(!)** systemd-boot будет установлен как приложение EFI по умолчанию.

## Фиксы
Иногда возникают проблемы с отображением *systemd-boot* загрузчика в меню загрузки *UEFI* \
На моем опыте такое происходило с использованием *внешнего USB диска*: на ноутбуке загрузчик отображался
и запускался, на ПК - нет. Причиной этому могут стать другие установленные загрузчики.

> (Возможно это поможет вам, не уверен в этом на 100%)

На примере загрузчика Windows\
Создаем внутри раздела EFI каталог EFI\

```
sudo mkdir /boot/EFI/EFI
```

Далее, копируем */EFI/Microsoft* в */EFI/EFI*

```
sudo cp /boot/EFI/Microsoft /EFI/EFI
```

## Создание загрузочной записи
По умолчанию, systemd-boot ищет загрузочные записи по пути:

- Windows Boot Manager - /EFI/Microsoft/Boot/Bootmgfw.efi
* UEFI Shell - /boot/shellx64.efi
+ Linux - /EFI/Linux/~

Это не всегда срабатывает, поэтому есть возможность создания своей загрузочной записи.\
Все записи находятся в каталоге\
*/boot/EFI/loader/entries*

**Windows 10**\
Создаем новую запись windows.conf\
Загрузчик Windows в нашем случае находится по пути\
*/boot/EFI/EFI/Microsoft*

Настройка файла конфигурации
```
title Windows 10
efi /EFI/EFI/Microsoft/Boot/Bootmgfw.efi
```

**Linux Mint**\
Смонтируем EFI предварительно отмонтировав все точки монтирования */boot*, если они имеются.
Для удобства смонтируем раздел как /mnt/esp
```
sudo su
umount /boot
mkdir /mnt/esp
mount /dev/sdc1 /mnt/efi
```

Создаем новую запись **mint.conf** в */mnt/esp/EFI/loader/entries* и новый каталог **mint** в */mnt/esp/EFI/*

```
sudo su
touch /mnt/esp/EFI/loader/entries/mint.conf
mkdir /mnt/esp/EFI/mint
```

Теперь, копируйте ваше ядро и initramfs в ESP из */boot* системы
в каталог */mnt/esp/EFI/mint*

```
cp /boot/vmlinuz-linux /mnt/esp/EFI/mint
cp /boot/initramfs-linux.img /mnt/esp/EFI/mint
```

> Ядро и файл initramfs в разных дистрибутивах Linux
> могут именоваться по-разному.

Настроим файл конфигурации
```
title Linux Mint
linux /EFI/mint/vmlinuz-linux
initrd /EFI/mint/initramfs-linux.img
options root=UUID=your_UUID rw
```

Чтобы узнать *UUID* диска нужно выполнить команду

```
sudo blkid
```

Также в опции root можно передать Label диска
```
options root=LABEL=os_mint rw
```

Детальнее про именование дисков\
:point_right: https://wiki.archlinux.org/title/persistent_block_device_naming

Детальнее про systemd-boot <br/>
:point_right: https://wiki.archlinux.org/title/systemd-boot