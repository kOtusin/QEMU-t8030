### USB подключение эмулированного iPhone
В данный момент USB эмулированного iPhone не может подключиться напрямую к основному компьютеру. Вместо этого он подключается к другой виртуальной машине (через UNIX-сокет или TCP) или другому компьютеру (через TCP).

## Настройки удаленного USB подключения
**В эмуляторе iPhone** (указываются в флаге -M):
- `usb-conn-type`: unix (по умолчанию), ipv4 или ipv6
- `usb-conn-addr`: Путь или IP-адрес (для UNIX-сокета используется путь)
- `usb-conn-port`: Порт сервера (только для ipv4/ipv6)
  
*Примеры:*
- Без указания параметров (используются значения по умолчанию), `-M t8030,...,usb-conn-type=ipv4,usb-conn-addr=127.0.0.1,usb-conn-port=8030`

**На стороне ВМ компаньона (используя -device usb-tcp-remote)**:
- Те же параметры, но без префикса usb-
  
*Примеры:*
- `-usb -device usb-ehci,id=ehci -device usb-tcp-remote,bus=ehci.0`
- `-usb -device usb-ehci,id=ehci -device usb-tcp-remote,conn-type=ipv4,conn-addr=127.0.0.1,conn-port=8030,bus=ehci.0`

> [!CAUTION]
> ВМ компаньон всегда должен запускаться ДО эмулированного iPhone, иначе USB-подключение не будет установлено.

## Настройка виртуальной машины
Настройка ВМ делается по тому же пути, что и в обычном случае (предпочтительно использовать легковесную ОС без графической оболочки, например Arch Linux или Artix Linux).

Для запуска x86 компаньона используется файл qemu-system-x86_64 из сборочной директории.

*Примечание:*
Для дистрибутивов без systemd может потребоваться дополнительная настройка правил udev и сервисов автозапуска.

## Настройка инструментов для iDevice
> [!NOTE]
> Все действия выполняются в ВМ компаньоне.

Мы будем использовать сторонние инструменты [libimobiledevice](https://github.com/libimobiledevice). Они аналогичны проприетарным инструментам Apple, но являются открытыми и кроссплатформенными.

Последние релизы сильно устарели, поэтому необходимо собрать инструменты из исходного кода (подробности в README каждого проекта).

Необходимые проекты для сборки:
- idevicerestore
- libimobiledevice 
- libimobiledevice-glue
- libirecovery
- libplist 
- libtatsu
- libusbmuxd
- usbmuxd

Пример команды сборки:
```bash
PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/ ./autogen.sh && make -j$(nproc) && sudo make install
```

Для idevicerestore необходимо применить [этот патч](https://github.com/user-attachments/files/20678871/idevicerestore.patch):
```bash
git apply ../idevicerestore.patch
```

## Перенос файлов в ВМ

Первым делом необходимо включить NBD на хосте 
````
modprobe nbd max_part=8
````
Далее нужно подлкючить диск ВМ к хосту
````
qemu-nbd --connect=/dev/nbd0 /path/to/qcow2_file
````
После этого важно узнать раздел, в котором содержиться директория home(обычно раздел с rootfs) командой ``fdisk /dev/nbd0 -l``

Команда для монтирование раздела с home в папку mnt
````
mount /dev/nbd0p1 /mnt
````
Для копирования файлов можно использовать утилиту ``cp`` или ``rsync``.
