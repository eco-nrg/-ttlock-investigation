# ttlock-investigation

## Описание протокола умных замков TTLock

### Немного теории

Замки TTLock работают через Bluetooth LE (BLE). С устройством BLE можно общаться через т.н. "Сервисы" - через чтение и запись "Характеристик" этих Сервисов.

* Подробно про BLE, GATT, Сервисы и Характеристики: [тут](https://devzone.nordicsemi.com/nordic/short-range-guides/b/bluetooth-low-energy/posts/ble-services-a-beginners-tutorial)
* Работа с BLE на Rasbperry Pi: [тут](https://elinux.org/RPi_Bluetooth_LE)

### Доступные Сервисы

Для общения с приложением в замке предусмотрены следующие Сервисы:

|UUID|Название|Примечание|
|----|--------|----------|
||asd|
