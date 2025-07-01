
> [!CAUTION]
> Для данных действий требуется хост или ВМ с установленной macOS(можно использовать [OSX-KVM](https://github.com/kholia/OSX-KVM) от kholia)

## Присоединение диска
````
hdiutil attach -imagekey diskimage-class=CRawDiskImage -blocksize 4096 nvme.1
````
## Включение режима R/W
````
sudo diskutil enableownership /Volumes/System
sudo mount -urw /Volumes/System
````
## Патчинг Dyld Shared Cache
Важно скопировать оригинальный dyld в тот же каталог, что и [PatchDYLD.sh](https://github.com/ChefKissInc/QEMUAppleSiliconTools/raw/refs/heads/master/PatchDYLD.sh)

Запуск патчинга делается этой командой:
````
sudo chmod +x ./PatchDYLD.fish && sudo ./PatchDYLD.fish # fish shell
sudo chmod +x ./PatchDYLD.sh && sudo ./PatchDYLD.sh # other shells
````

## Отключение проблематичных сервисов
В настоящий момент требуется отключение данных сервисов для корректного запуска: ``com.apple.voicemail.vmd``, ``com.apple.CommCenter``, ``com.apple.locationd``

Бекап оригинально конфига с сервисами:
````
cp /Volumes/System/System/Library/xpc/launchd.plist launchd.plist
````

Конвертация конфига в читабельный plist:
````
sudo plutil -convert xml1 /Volumes/System/System/Library/xpc/launchd.plist
````
Изменение конфига:
````
sudo nano /Volumes/System/System/Library/xpc/launchd.plist
````
Далее находим эти сервисы через CTRL + W и вставляем это:
````
<key>Disabled</key>
<true/>
````

Это должно выглядеть примерно так:
````
<key>/System/LaunchDaemons/com.apple.voicemail.vad.plist<key/>
<dict>
        <key>Disabled<key/>
        <true/>
````

Отключение диска:
````
diskutil eject /Volumes/System
````

## Победа
И всё готово!
Обратите внимание, что теперь система не сможет загрузиться без параметра загрузки ``launchd_unsecure_cache=1``, пока вы не восстановите оригинальный конфиг.
Теперь снова запустите эмулятор и дождитесь завершения восстановления. Когда процесс завершится, вы увидите экран начальной настройки. Приятного использования!
