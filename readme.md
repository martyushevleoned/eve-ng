# eve-ng

## Установка eve-ng

Ссылки:
* [страница со списком образов](https://www.eve-ng.net/index.php/download/)
* [direct ссылка на скачивания](https://customers.eve-ng.net/eve-ce-prod-6.2.0-4-full.iso)


## Файлы виртуальных машин

Расположение файлов:
* Лаборатории - `/opt/unetlab/labs`
* Статические образы - `/opt/unetlab/addons`
* Live образы - `/opt/unetlab/tmp`

В Лабораториях используются следующие образы:
* ubuntu desktop 22 - пользовательские устройства (root/Test123)
* ubuntu server 22 - узлы сети с системным ПО (root/Test123)
* Cisco IOS 7206VXR - маршрутизатор

После добавления дополнительных образов следует обновить права доступа:
```shell
scp -r ./<path>/* root@<ip>:/opt/uetlab/addons
/opt/wrappers/unl_wrapper -a fixpermissions
```

## Перенос лаборатории

Нужно перенести директории:
* `/opt/unetlab/labs`
* `/opt/unetlab/addons`
* `/opt/unetlab/tmp`