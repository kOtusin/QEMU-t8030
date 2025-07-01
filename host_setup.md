# Настройка хост машины

### Установка зависимостей
Для дистрибутива Debian/Ubuntu based
````
sudo apt-get install -y build-essential libtool meson ninja-build pkg-config libcapstone-dev device-tree-compiler libglib2.0-dev gnutls-bin libjpeg-turbo8-dev libpng-dev libslirp-dev libssh-dev libusb-1.0-0-dev liblzo2-dev libncurses5-dev libpixman-1-dev libsnappy-dev vde2 zstd libgnutls28-dev libgmp10 libgmp3-dev lzfse liblzfse-dev libgtk-3-dev libsdl2-dev nettle-dev
````
Для дистрибутива Arch based
````
sudo pacman -S git glib2 dtc pixman zlib libtasn1 ninja base-devel cmake gnutls pkgconf sdl2 libssh capstone gtk3 nettle
git clone https://aur.archlinux.org/lzfse.git && cd lzfse && makepkg -si && exit # or use AUR helper
````
Для macOS
````
brew install libtool glib libtasn1 meson ninja pixman gnutls libgcrypt pkgconf lzfse capstone nettle ncurses libslirp libssh libpng jpeg-turbo zstd
````
### Клонирование и компиляция
#### Клонирование репозитория
````
git clone https://github.com/ChefKissInc/QEMUAppleSilicon
git submodule update --init
````
#### Компиляция QEMU

Для mac(ARM)
````
mkdir build && cd build
LIBTOOL="glibtool" ../configure --target-list=aarch64-softmmu,x86_64-softmmu --disable-bsd-user --disable-guest-agent --enable-lzfse --enable-slirp --enable-capstone --enable-curses --enable-libssh --enable-virtfs --enable-zstd --extra-cflags=-DNCURSES_WIDECHAR=1 --disable-sdl --disable-gtk --enable-cocoa --enable-nettle --enable-gnutls --extra-cflags="-I/opt/homebrew/include" --extra-ldflags="-L/opt/homebrew/lib" --disable-werror
make -j$(sysctl -n hw.logicalcpu)
````

Для mac(x86)
````
mkdir build && cd build
LIBTOOL="glibtool" ../configure --target-list=aarch64-softmmu,x86_64-softmmu --disable-bsd-user --disable-guest-agent --enable-lzfse --enable-slirp --enable-capstone --enable-curses --enable-libssh --enable-virtfs --enable-zstd --extra-cflags=-DNCURSES_WIDECHAR=1 --disable-sdl --disable-gtk --enable-cocoa --enable-nettle --enable-gnutls --disable-werror
make -j$(sysctl -n hw.logicalcpu)
````

Для Linux
````
mkdir build && cd build
../configure --target-list=aarch64-softmmu,x86_64-softmmu --enable-lzfse --enable-slirp --enable-capstone --enable-curses --enable-libssh --enable-virtfs --enable-zstd --enable-nettle --enable-gnutls --enable-gtk --enable-sdl --disable-werror
make -j$(nproc)
````
