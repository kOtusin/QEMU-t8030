> [!CAUTION]
> Не распространяйте файлы — будь то изменённые или оригинальные образы дисков, а также расшифрованные, пропатченные, модифицированные или стоковые прошивки и прочее. Это прямое нарушение лицензии Apple (EULA). Правда, если само нарушение EULA — не уголовное преступление в вашей стране, то другие действия могут подпадать под другие случаи. Лучше сверьтесь с местным законодательством.

> [!WARNING]
> Не помещайте файлы в папку сборки или source tree эмулятора, в противном случае высок риск их сломать.

## Что нужно подготовить
Вам необходимо установить ``python3-pyasn1 python3-pyasn1-modules`` из менеджера пакетов вашего дистрибутива или pip для скриптов Python, используемых далее.

## Создание виртуальных дисков
````
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.1 16G
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.2 8M
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.3 128K
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.4 8K
./QEMUAppleSilicon/build/qemu-img create -f raw nvram  8K
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.6 4K
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.7 1M
./QEMUAppleSilicon/build/qemu-img create -f raw nvme.8 3M
./QEMUAppleSilicon/build/qemu-img create -f raw sep_nvram 2K
./QEMUAppleSilicon/build/qemu-img create -f raw sep_ssc 128K
````

> [!NOTE]
> nvme.1 может быть большего размера.

## IOS Firmware
### Скачивание прошивки
iOS 14.0 beta 5 для ``iPhone12,1`` [доступна здесь](https://updates.cdn-apple.com/2020SummerSeed/fullrestores/001-35886/5FE9BE2E-17F8-41C8-96BB-B76E2B225888/iPhone11,8,iPhone12,1_14.0_18A5351d_Restore.ipsw)

### Извлечение файлов
````
mkdir iPhone11_8_iPhone12_1_14.0_18A5351d_Restore && cd iPhone11_8_iPhone12_1_14.0_18A5351d_Restore
unzip ../iPhone11,8,iPhone12,1_14.0_18A5351d_Restore.ipsw
cd ..
````
Самый большой файл в папке ``iPhone11_8_iPhone12_1_14.0_18A5351d_Restore`` можно удалить, так как это файл основной ОС.

### Создание AP Ticket
Используемая версия iOS не подписана, поэтому нам придется костылить собстенный AP Ticket.

Для этого используется [create_apticket.py](https://github.com/ChefKissInc/QEMUAppleSiliconTools/raw/refs/heads/master/create_apticket.py)

SHSH ticket можно получить [тут](https://github.com/ChefKissInc/QEMUAppleSiliconTools/raw/refs/heads/master/ticket.shsh2)

Запустите скрипт командой
````
python3 create_apticket.py n104ap iPhone11_8_iPhone12_1_14.0_18A5351d_Restore/BuildManifest.plist ticket.shsh2 root_ticket.der
````

> [!CAUTION]
> Не изменяйте данный ticket, если только не собираетесь делать полный сброс. Этот ticket необходим на всех этапах загрузки, включая период после завершения установки.

## Загрузка SEP ROM
Не могу вставить сюда ссылку, так как Apple очень не любит подобное. Нужен файл ``AppleSEPROM-Cebu-B1``, который вы можете найти в интернете сами.

## Подготовка SEP FIRMWARE
### То, что нужно скачать и установить
[Скрипт](https://github.com/ChefKissInc/QEMUAppleSiliconTools/raw/refs/heads/master/create_septicket.py) для создания Ticket.

[img4tool](https://github.com/tihmstar/img4tool)

[img4](https://github.com/xerub/img4lib)

## Загрузка более новой прошивки
Скачать iOS 14.7.1 для ``iPhone12,1`` можно [тут](https://updates.cdn-apple.com/2021SummerFCS/fullrestores/071-73868/321919C4-1F21-4387-936D-B72374C39DD6/iPhone11,8,iPhone12,1_14.7.1_18G82_Restore.ipsw)

> [!NOTE]
> Из этого ipsw нужно взять ``sep-firmware.n104.RELEASE.im4p``. Остальные файлы, такие как ``BuildManifest`` должны быть из iOS 14.0 beta 5

### Создание Ticket
````
python3 create_septicket.py n104ap iPhone11_8_iPhone12_1_14.0_18A5351d_Restore/BuildManifest.plist ticket.shsh2 sep_root_ticket.der
````

### Извлечение файлов 
````
mkdir iPhone11,8,iPhone12,1_14.7.1_18G82_Restore && cd iPhone11,8,iPhone12,1_14.7.1_18G82_Restore
unzip ../iPhone11,8,iPhone12,1_14.7.1_18G82_Restore.ipsw
cd ..
````

### Дешифровка прошивки
````
img4tool -e --iv THE_SEP_FW_IV --key THE_SEP_FW_KEY -o sep-firmware.n104.RELEASE iPhone11,8,iPhone12,1_14.7.1_18G82_Restore/Firmware/all_flash/sep-firmware.n104.RELEASE.im4p
````
В THE_SEP_FW_IV и THE_SEP_FW_KEY вставляем нужные значения, которые можно нагуглить запросом "iOS firmware keys".

### Перепаковка прошивки в IMG4

````
img4tool -t rsep -d ff86cbb5e06c820266308202621604696d706c31820258ff87a3e8e0730e300c1604747a3073020407e78000ff868bc9da730e300c160461726d73020400d84000ff87a389da7382010e3082010a160474626d730482010036373166326665363234636164373234643365353332633464666361393732373734353966613362326232366635643962323032383061643961303037666635323834393936383138653962303461336434633034393061663833313630633464356330313832396536633635303836313230666133346539663263323165373237316265623231636139386237386464303064363037326530366464393962666163623262616362623261373830613465636161303363326361333930303931636334613461666231623737326238646234623865653566663365636437373135306531626566333633303034336637373665666265313130316538623433ff87a389da7282010e3082010a160474626d720482010034626631393164373134353637356364306264643131616166373734386138663933373363643865666234383830613130353237633938393833666636366538396438333330623730626237623561333530393864653735353265646635373762656166363137353235613831663161393838373838613865346665363734653936633439353066346136366136343231366561356438653333613833653530353962333536346564633533393664353539653337623030366531633637343633623736306336333164393163306339363965366662373130653962333061386131396338333166353565636365393835363331643032316134363361643030 -c sep-firmware.n104.RELEASE.im4p sep-firmware.n104.RELEASE
img4 -F -o sep-firmware.n104.RELEASE.new.img4 -i sep-firmware.n104.RELEASE.im4p -M sep_root_ticket.der
````
