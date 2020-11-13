# ttlock-investigation

## Описание протокола умных замков TTLock

### Немного теории

Замки TTLock работают через Bluetooth LE (BLE). С устройством BLE можно общаться через т.н. "Сервисы" - через чтение и запись "Характеристик" этих Сервисов.

* Подробно про BLE, GATT, Сервисы и Характеристики: [тут](https://devzone.nordicsemi.com/nordic/short-range-guides/b/bluetooth-low-energy/posts/ble-services-a-beginners-tutorial)
* Работа с BLE на Rasbperry Pi: [тут](https://elinux.org/RPi_Bluetooth_LE)

### Доступные Сервисы

Для общения с приложением в замке есть следующие Сервисы:

|UUID|Название|Примечание|
|----|--------|----------|
|0000fff2-0000-1000-8000-00805f9b34fb|TTL_WRITE|Сервис для отправки команд замку|
|0000fff4-0000-1000-8000-00805f9b34fb|TTL_READ|Сервис для получения ответов на команды от замка|
|00001910-0000-1000-8000-00805f9b34fb|TTL_SERVICE|???|
|0000180a-0000-1000-8000-00805f9b34fb|DEVICE_INFORMATION_SERVICE|???|
|00002a26-0000-1000-8000-00805f9b34fb|READ_FIRMWARE_REVISION_UUID|???|
|00002a27-0000-1000-8000-00805f9b34fb|READ_HARDWARE_REVISION_UUID|???|
|00002a29-0000-1000-8000-00805f9b34fb|READ_MANUFACTURER_NAME_UUID|???|
|0000180a-0000-1000-8000-00805f9b34fb|DEVICE_INFORMATION_SERVICE|???|
|00002a24-0000-1000-8000-00805f9b34fb|READ_MODEL_NUMBER_UUID|???|

Для обмена информацией с замком используются Сервисы TTL_WRITE и TTL_READ. В TTL_WRITE мы пишем команду, из TTL_READ читаем ответ. Остальные Сервисы нам пока не интересны.

### Формат сообщений

Команды замку и ответы от него имеют следующий формат:

|Размер|Описание|Примечание|
|-|-|-|
|2 байта|Заголовок|Всегда **0x7F 0x5A**|
|1 байт|Тип протокола|Для замка - **0x05**|
|1 байт|Версия протокола|Для замка - **0x03**|
|1 байт|scene (?)|Для замка - **0x01** или **0x07**|
|2 байта|groupId (?)|Для замка - **0x00 0x01**|
|2 байта|orgId (?)|Для замка - **0x00 0x01**|
|1 байт|Код команды|Все возможные коды описаны ниже|
|1 байт|encrypt|Всегда **0xAA**. Скорее всего, не используется|
|1 байт|length|Размер поля **data**, может быть 0|
|**length** байт|data|Параметры команды, обычно зашифрованы алгоритмом AES, могут отсутствовать|
|1 байт|CRC|Контрольная сумма сообщения, вычисляется по алгоритму Dallas Semiconductor 8-bit CRC (реализация приведена ниже)|
|2 байта|Окончание|Всегда **0x0D 0x0A**|

### Список команд

|Команда|Название|Примечание|
|-|-|-|
|0x01|COMM_SEARCHE_DEVICE_FEATURE||
|0x19|COMM_GET_AES_KEY||
|0x43|COMM_TIME_CALIBRATE||
|0x45|COMM_INITIALIZATION||
|0x54|COMM_RESPONSE|Все ответы от замка приходят с таким кодом команды. Кроме ответа на COMM_GET_AES_KEY, он приходит с кодом COMM_GET_AES_KEY (WTF???)|
|0x56|COMM_ADD_ADMIN||
|0x57|COMM_GET_ALARM_ERRCORD_OR_OPERATION_FINISHED||
|0x62|COMM_AUDIO_MANAGE||

### Вычисление DS CRC8

Пример кода на Python
```python
dscrc_table = [0, 0x5E, 0xBC, 0xE2, 0x61, 0x3F, 0xDD, 0x83, 0xC2,
    0x9C, 0x7E, 0x20, 0xA3, 0xFD, 0x1F, 0x41, 0x9D, 0xC3,
    0x21, 0x7F, 0xFC, 0xA2, 0x40, 0x1E, 0x5F, 1, 0xE3,
    0xBD, 0x3E, 0x60, 0x82, 0xDC, 0x23, 7, 0x9F, 0xC1,
    0x42, 0x1C, 0xFE, 0xA0, 0xE1, 0xBF, 0x5D, 3, 0x80,
    0xDE, 0x3C, 0x62, 0xBE, 0xE0, 2, 0x5C, 0xDF, 0x81,
    0x63, 0x3D, 0x7C, 0x22, 0xC0, 0x9E, 0x1D, 0x43, 0xA1,
    0xFF, 0x46, 0x18, 0xFA, 0xA4, 0x27, 0x79, 0x9B, 0xC5,
    0x84, 0xDA, 0x38, 0x66, 0xE5, 0xBB, 0x59, 7, 0xDB,
    0x85, 0x67, 0x39, 0xBA, 0xE4, 6, 0x58, 0x19, 0x47,
    0xA5, 0xFB, 0x78, 0x26, 0xC4, 0x9A, 0x65, 0x3B, 0xD9,
    0x87, 4, 0x5A, 0xB8, 0xE6, 0xA7, 0xF9, 0x1B, 0x45,
    0xC6, 0x98, 0x7A, 0x24, 0xF8, 0xA6, 0x44, 0x1A, 0x99,
    0xC7, 0x25, 0x7B, 0x3A, 0x64, 0x86, 0xD8, 0x5B, 5,
    0xE7, 0xB9, 0x8C, 0xD2, 0x30, 0x6E, 0xED, 0xB3, 0x51,
    0xF, 0x4E, 0x10, 0xF2, 0xAC, 0x2F, 0x71, 0x93, 0xCD,
    0x11, 0x4F, 0xAD, 0xF3, 0x70, 0x2E, 0xCC, 0x92, 0xD3,
    0x8D, 0x6F, 0x31, 0xB2, 0xEC, 0xE, 0x50, 0xAF, 0xF1,
    0x13, 0x4D, 0xCE, 0x90, 0x72, 0x2C, 0x6D, 0x33, 0xD1,
    0x8F, 0xC, 0x52, 0xB0, 0xEE, 0x32, 0x6C, 0x8E, 0xD0,
    0x53, 0xD, 0xEF, 0xB1, 0xF0, 0xAE, 0x4C, 0x12, 0x91,
    0xCF, 0x2D, 0x73, 0xCA, 0x94, 0x76, 0x28, 0xAB, 0xF5,
    0x17, 0x49, 8, 0x56, 0xB4, 0xEA, 0x69, 0x37, 0xD5,
    0x8B, 0x57, 9, 0xEB, 0xB5, 0x36, 0x68, 0x8A, 0xD4,
    0x95, 0xCB, 0x29, 0x77, 0xF4, 0xAA, 0x48, 0x16, 0xE9,
    0xB7, 0x55, 0xB, 0x88, 0xD6, 0x34, 0x6A, 0x2B, 0x75,
    0x97, 0xC9, 0x4A, 0x14, 0xF6, 0xA8, 0x74, 0x2A, 0xC8,
    0x96, 0x15, 0x4B, 0xA9, 0xF7, 0xB6, 0xE8, 0xA, 0x54,
    0xD7, 0x89, 0x6B, 0x35]


def crccompute(to_compute):
    res = 0
    for el in to_compute:
        res = dscrc_table[(res ^ el) & 0xFF]

    return res

if __name__ == '__main__':
    assert crccompute([0x7f,0x5a,0x05,0x03,0x01,0x00,0x01,0x00,0x01,0x45,0xaa,0x00]) == 0xb3
    assert crccompute([0x7f,0x5a,0x05,0x03,0x01,0x00,0x01,0x00,0x01,0x19,0xaa,0x10,0x26,0xed,0x1d,0x92,0x58,0xa1,0xe4,0x28,0xfc,0xb4,0x8b,0xbd,0x1c,0x0e,0x5a,0x46]) == 0xbc
    assert crccompute([0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x18,0x3a,0x8a,0x95,0x62,0x1a,0xdd,0x5d,0xef,0x91,0xb7,0x85,0x82,0xb1,0x8c,0x86]) == 0xa7
    print("Tests passed")
```

### Шифрование поля data

Наши друзья китайцы пекутся о секьюрности, поэтому поле `data` у сообщений всегда зашифровано. Для шифрования и расшифрования `data` используется алгоритм AES.

Функции для шифрования и расшифрования (вытащены из приложения TTLock):

```java
package com.ttlock;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

public class AESUtil {
    private static byte[] Decrypt(byte[] data, byte[] key, byte[] IV) throws Exception {
        if (key == null) {
            return null;
        }

        if(IV.length != 16) {
            System.err.println("the length of IV vector is not 16");
            return null;
        }

        SecretKeySpec keySpec = new SecretKeySpec(key, "AES");
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, keySpec, new IvParameterSpec(IV));

        return cipher.doFinal(data);
    }

    private static byte[] Encrypt(byte[] arg3, byte[] arg4, byte[] arg5) throws Exception {
        if(arg4 == null) {
            System.out.println("Key is null");
            return null;
        }

        if(arg4.length != 16) {
            System.out.println("the length of Key is not 16");
            return null;
        }

        if(arg5.length != 16) {
            System.out.println("the length of IV vector is not 16");
            return null;
        }

        SecretKeySpec v0 = new SecretKeySpec(arg4, "AES");
        Cipher v4 = Cipher.getInstance("AES/CBC/PKCS5Padding");
        v4.init(1, v0, new IvParameterSpec(arg5));
        return v4.doFinal(arg3);
    }

    public static byte[] aesDecrypt(byte[] data, byte[] aesKey) {
        try {
            return AESUtil.Decrypt(data, aesKey, aesKey);
        }
        catch(Exception v0) {
            v0.printStackTrace();
            return null;
        }
    }

    public static byte[] aesEncrypt(byte[] data, byte[] aesKey) {
        try {
            return AESUtil.Encrypt(data, aesKey, aesKey);
        }
        catch(Exception v0) {
            v0.printStackTrace();
            return null;
        }
    }
}
```

Для шифрования `data` в приложении вызывается `AESUtil.aesEncrypt(data, key)`, для расшифрования - `AESUtil.aesDecrypt(data, key)`

При инициализации замка приложение шифрует команды стандартным ключом, который зашит в само приложение:

```
[0x98,0x76,0x23,0xe8,0xa9,0x23,0xa1,0xbb,0x3d,0x9e,0x7d,0x03,0x78,0x12,0x45,0x88]
```

Затем приложение вызывает команду `COMM_GET_AES_KEY` и получает от замка новый ключ шифрования, который используется для шифрования всех последующих команд и расшифрования всех ответов замка.

### Формат ответа

Поле `data` в ответе от замка (после расшифрования) имеет следующий вид:

|Размер|Описание|Примечание|
|-|-|-|
|1 байт|Код команды|Содержит код команды, в ответ на которую было отправлено это сообщение|
|1 байт|Статус выполнения команды|**0x01**: успех, остальные значения: not so much|
|остальное|Другие данные, формат зависит от того, на какую команду отвечает это сообщение|Примеры ответов будут рассмотрены ниже|

## Работа с замком

Мы знаем, как генерировать и отправлять команды замку, и как получать от замка ответы. Теперь мы можем разобраться в управлении замком.

### Инициализация замка

Для того, чтобы приложение могло работать с замком, замок необходимо проинициализировать. Для этого замку необходимо отправить следующие команды:

> Возможно, какие-то команды необязательны - здесь приведены все команды, перехваченные из приложения TTLock.

#### 1. COMM_INITIALIZE

* Код: `0x45`
* Параметры: `нет`
* Пример сообщения: `[0x7f,0x5a,0x05,0x03,0x01,0x00,0x01,0x00,0x01,0x45,0xaa,0x00,0xb3,0x0d,0x0a]`
* Пример ответа: `[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x18,0x3a,0x8a,0x95,0x62,0x1a,0xdd,0x5d,0xef,0x91,0xb7,0x85,0x82,0xb1,0x8c,0x86,0xa7,0x0d,0x0a]`

Ответ на эту команду можно не расшифровывать. Приложение TTLock этого не делает. Более того, у меня не получилось расшифровать это сообщение ни стандартным ключом, ни полученным от замка позже.

Зачем нужен ответ? Возможно, чтобы узнать тип замка.

#### 2. COMM_GET_AES_KEY

* Код: 0x19
* Параметры: вендор (строка `"SCIENER"`)
* Пример сообщения: `[0x7f,0x5a,0x05,0x03,0x01,0x00,0x01,0x00,0x01,0x19,0xaa,0x10,0x26,0xed,0x1d,0x92,0x58,0xa1,0xe4,0x28,0xfc,0xb4,0x8b,0xbd,0x1c,0x0e,0x5a,0x46,0xbc,0x0d,0x0a]`
* Пример ответа: `[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x19,0xaa,0x20,0xfd,0xa0,0xe8,0x17,0xd5,0x96,0xe2,0xf4,0x8c,0xac,0xbb,0x58,0xef,0x5f,0xf5,0x69,0x5b,0x9f,0x0a,0x3f,0x4d,0x4f,0xd4,0x61,0x25,0x83,0xcf,0x18,0xed,0x6e,0x5e,0xe9,0xf7,0x0d,0x0a]`

В этой команде мы помещаем в поле `data` строку "SCIENER", зашифрованную алгоритмом AES на стандартном ключе (`[0x98,0x76,0x23,0xe8,0xa9,0x23,0xa1,0xbb,0x3d,0x9e,0x7d,0x03,0x78,0x12,0x45,0x88]`
