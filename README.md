# ttlock-investigation

**Это черновик документа!** Информация будет дополняться.

В этом документе описан протокол взаимодействия с умными замками TTLock. Протокол восстановлен на основе анализа приложения [TTLock](https://play.google.com/store/apps/details?id=com.tongtongsuo.app) для Android.

<!-- MarkdownTOC autolink="true" -->

- [Описание протокола умных замков TTLock](#%D0%9E%D0%BF%D0%B8%D1%81%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BF%D1%80%D0%BE%D1%82%D0%BE%D0%BA%D0%BE%D0%BB%D0%B0-%D1%83%D0%BC%D0%BD%D1%8B%D1%85-%D0%B7%D0%B0%D0%BC%D0%BA%D0%BE%D0%B2-ttlock)
  - [Немного теории](#%D0%9D%D0%B5%D0%BC%D0%BD%D0%BE%D0%B3%D0%BE-%D1%82%D0%B5%D0%BE%D1%80%D0%B8%D0%B8)
  - [Доступные Сервисы](#%D0%94%D0%BE%D1%81%D1%82%D1%83%D0%BF%D0%BD%D1%8B%D0%B5-%D0%A1%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D1%8B)
  - [Формат сообщений](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D1%81%D0%BE%D0%BE%D0%B1%D1%89%D0%B5%D0%BD%D0%B8%D0%B9)
  - [Список кодов команд](#%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA-%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4)
  - [Вычисление DS CRC8](#%D0%92%D1%8B%D1%87%D0%B8%D1%81%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-ds-crc8)
  - [Шифрование поля data](#%D0%A8%D0%B8%D1%84%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BF%D0%BE%D0%BB%D1%8F-data)
  - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0)
- [Работа с замком](#%D0%A0%D0%B0%D0%B1%D0%BE%D1%82%D0%B0-%D1%81-%D0%B7%D0%B0%D0%BC%D0%BA%D0%BE%D0%BC)
  - [Подключение к замку](#%D0%9F%D0%BE%D0%B4%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BA-%D0%B7%D0%B0%D0%BC%D0%BA%D1%83)
  - [Инициализация замка](#%D0%98%D0%BD%D0%B8%D1%86%D0%B8%D0%B0%D0%BB%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F-%D0%B7%D0%B0%D0%BC%D0%BA%D0%B0)
    - [1. COMM_INITIALIZE](#1-comm_initialize)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-1)
    - [2. COMM_GET_AES_KEY](#2-comm_get_aes_key)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-2)
    - [3. COMM_ADD_ADMIN](#3-comm_add_admin)
      - [Код для генерации паролей:](#%D0%9A%D0%BE%D0%B4-%D0%B4%D0%BB%D1%8F-%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D0%B8-%D0%BF%D0%B0%D1%80%D0%BE%D0%BB%D0%B5%D0%B9)
      - [Формирование команды:](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B)
      - [Конвертация паролей в массив байт:](#%D0%9A%D0%BE%D0%BD%D0%B2%D0%B5%D1%80%D1%82%D0%B0%D1%86%D0%B8%D1%8F-%D0%BF%D0%B0%D1%80%D0%BE%D0%BB%D0%B5%D0%B9-%D0%B2-%D0%BC%D0%B0%D1%81%D1%81%D0%B8%D0%B2-%D0%B1%D0%B0%D0%B9%D1%82)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-3)
    - [4. COMM_TIME_CALIBRATE](#4-comm_time_calibrate)
      - [Преобразование даты и времени](#%D0%9F%D1%80%D0%B5%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B4%D0%B0%D1%82%D1%8B-%D0%B8-%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%B8)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-4)
    - [5. COMM_CHECK_USER_TIME](#5-comm_check_user_time)
      - [Формирование команды](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-1)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-5)
    - [6. COMM_CHECK_RANDOM](#6-comm_check_random)
      - [Формирование команды](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-2)
        - [Генерация кода разблокировки](#%D0%93%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F-%D0%BA%D0%BE%D0%B4%D0%B0-%D1%80%D0%B0%D0%B7%D0%B1%D0%BB%D0%BE%D0%BA%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B8)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-6)
    - [To be continued...](#to-be-continued)
  - [Поднятие замка](#%D0%9F%D0%BE%D0%B4%D0%BD%D1%8F%D1%82%D0%B8%D0%B5-%D0%B7%D0%B0%D0%BC%D0%BA%D0%B0)
    - [1. COMM_CHECK_ADMIN](#1-comm_check_admin)
      - [Формирование команды](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-3)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-7)
    - [2. COMM_FUNCTION_LOCK](#2-comm_function_lock)
      - [Формирование команды](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-4)
        - [Генерация метки времени](#%D0%93%D0%B5%D0%BD%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F-%D0%BC%D0%B5%D1%82%D0%BA%D0%B8-%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%B8)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-8)
        - [Первый ответ](#%D0%9F%D0%B5%D1%80%D0%B2%D1%8B%D0%B9-%D0%BE%D1%82%D0%B2%D0%B5%D1%82)
        - [Второй ответ](#%D0%92%D1%82%D0%BE%D1%80%D0%BE%D0%B9-%D0%BE%D1%82%D0%B2%D0%B5%D1%82)
  - [Опускание замка](#%D0%9E%D0%BF%D1%83%D1%81%D0%BA%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B7%D0%B0%D0%BC%D0%BA%D0%B0)
    - [1. COMM_CHECK_ADMIN](#1-comm_check_admin-1)
    - [2. COMM_UNLOCK](#2-comm_unlock)
      - [Формирование команды](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-5)
      - [Формат ответа](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B0%D1%82-%D0%BE%D1%82%D0%B2%D0%B5%D1%82%D0%B0-9)

<!-- /MarkdownTOC -->

## Описание протокола умных замков TTLock

### Немного теории

Приложение связывается с умными замками через Bluetooth Low Energy (BLE). У BLE устройства есть список т.н. "Сервисов". Каждый Сервис содержит набор "Характеристик", и общение с BLE устройством происходит через чтение и запись значений этих характеристик.

* Подробно про BLE, GATT, Сервисы и Характеристики: [тут](https://devzone.nordicsemi.com/nordic/short-range-guides/b/bluetooth-low-energy/posts/ble-services-a-beginners-tutorial)
* Работа с BLE на Rasbperry Pi: [тут](https://elinux.org/RPi_Bluetooth_LE)

### Доступные Сервисы

Для общения с приложением в замке есть следующие Сервисы:

|Сервис|Название|Примечание|
|-|-|-|
|00001910-0000-1000-8000-00805f9b34fb|TTL_SERVICE|Cервис для отправки команд замку и получения от замка ответов|
|0000180a-0000-1000-8000-00805f9b34fb|DEVICE_INFORMATION_SERVICE|Сервис для получения сведений об устройстве|

У `TTL_SERVICE` есть следующие характеристики:

|Характеристика|Название|Примечание|
|--------------|--------|----------|
|0000fff2-0000-1000-8000-00805f9b34fb|TTL_WRITE|Характеристика для отправки команд замку|
|0000fff4-0000-1000-8000-00805f9b34fb|TTL_READ|Характеристика для получения ответов на команды от замка|

У `DEVICE_INFORMATION_SERVICE` есть следующие характеристики:

|Характеристика|Название|Примечание|
|--------------|--------|----------|
|00002a26-0000-1000-8000-00805f9b34fb|READ_FIRMWARE_REVISION_UUID|???|
|00002a27-0000-1000-8000-00805f9b34fb|READ_HARDWARE_REVISION_UUID|???|
|00002a29-0000-1000-8000-00805f9b34fb|READ_MANUFACTURER_NAME_UUID|???|
|00002a24-0000-1000-8000-00805f9b34fb|READ_MODEL_NUMBER_UUID|???|

Для обмена информацией с замком используются Характеристики TTL_WRITE и TTL_READ сервиса TTL_SERVICE. В TTL_WRITE мы пишем команду, из TTL_READ читаем ответ. Остальные Характеристики и Сервисы нам пока не интересны.

### Формат сообщений

Команды замку и ответы от него имеют следующий формат:

|Размер|Описание|Примечание|
|-|-|-|
|2 байта|Заголовок|Всегда **0x7F 0x5A**|
|1 байт|Тип протокола|Для замка - **0x05**|
|1 байт|Версия протокола|Для замка - **0x03**|
|1 байт|scene (?)|Для замка - **0x01**|
|2 байта|groupId (?)|Для замка - **0x00 0x01**|
|2 байта|orgId (?)|Для замка - **0x00 0x01**|
|1 байт|Код команды|Все возможные коды описаны ниже|
|1 байт|encrypt|Всегда **0xAA**|
|1 байт|length|Размер поля **data**, может быть 0|
|**length** байт|data|Параметры команды, обычно зашифрованы алгоритмом AES, могут отсутствовать|
|1 байт|CRC|Контрольная сумма сообщения, вычисляется по алгоритму Dallas Semiconductor 8-bit CRC (реализация приведена ниже)|
|2 байта|Окончание|Всегда **0x0D 0x0A**|

### Список кодов команд

|Команда|Название|Примечание|
|-|-|-|
|0x01|COMM_SEARCHE_DEVICE_FEATURE||
|0x19|COMM_GET_AES_KEY|Получить сеансовый ключ у замка|
|0x25|COMM_GET_OPERATE_LOG||
|0x41|COMM_CHECK_ADMIN||
|0x43|COMM_TIME_CALIBRATE||
|0x47|COMM_UNLOCK|Разблокировать замок|
|0x45|COMM_INITIALIZATION|Инициализация замка|
|0x54|COMM_RESPONSE|Все ответы от замка приходят с таким кодом команды. Кроме ответа на COMM_GET_AES_KEY, он приходит с кодом COMM_GET_AES_KEY (WTF???)|
|0x56|COMM_ADD_ADMIN||
|0x57|COMM_GET_ALARM_ERRCORD_OR_OPERATION_FINISHED||
|0x58|COMM_FUNCTION_LOCK||
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

Поля `data` у сообщений типа COMM_RESPONSE имеют похожий формат. После расшифрования они имеют следующий вид:

|Размер|Описание|Примечание|
|-|-|-|
|1 байт|Код команды|Содержит код команды, в ответ на которую было отправлено это сообщение|
|1 байт|Статус выполнения команды|**0x01**: успех, остальные значения: not so much|
|остальное|Другие данные, формат зависит от того, на какую команду отвечает это сообщение|Примеры ответов будут рассмотрены ниже|

## Работа с замком

Мы знаем, как генерировать команды и разбирать ответы. Теперь мы можем разобраться в управлении замком.

### Подключение к замку

После установления соединения с замком, необходимо его настроить. Алгоритм следующий:

1. Получить сервис TTL_SERVICE
2. Получить характеристики TTL_READ и TTL_WRITE этого сервиса (см. [Доступные Сервисы](#доступные-сервисы)). TTL_WRITE будет использоваться для записи сообщений, TTL_READ - для чтения
3. Включить уведомления для характеристики TTL_READ (см. код ниже)

Пример кода на java для включения уведомления:
```java
BluetoothLeService.UUID_HEART_RATE_MEASUREMENT = UUID.fromString("00002902-0000-1000-8000-00805f9b34fb");

if(characteristic.getUuid().toString().equals(BluetoothLeService.UUID_READ)) {
    bluetoothGatt.setCharacteristicNotification(characteristic, true);
    BluetoothGattDescriptor heartRateDescriptor = characteristic.getDescriptor(BluetoothLeService.UUID_HEART_RATE_MEASUREMENT);
    
    // public static final byte[] ENABLE_NOTIFICATION_VALUE = {0x01, 0x00}; - http://androidxref.com/7.1.2_r36/xref/frameworks/base/core/java/android/bluetooth/BluetoothGattDescriptor.java#36
    heartRateDescriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);

    if(bluetoothGatt.writeDescriptor(heartRateDescriptor)) {
        LogUtil.d("writeDescriptor successed", true);
    } else {
        LogUtil.d("writeDescriptor failed", true);
    }
}
```

### Инициализация замка

Для того, чтобы приложение могло работать с замком, замок необходимо проинициализировать. Для этого замку необходимо отправить следующие команды:

#### 1. COMM_INITIALIZE

* Код: `0x45`
* Параметры: `нет`

Эта команда всегда будет иметь один и тот же вид:

`[0x7f,0x5a,0x05,0x03,0x01,0x00,0x01,0x00,0x01,0x45,0xaa,0x00,0xb3,0x0d,0x0a]`


##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x18,0x3a,0x8a,0x95,0x62,0x1a,0xdd,0x5d,0xef,0x91,0xb7,0x85,0x82,0xb1,0x8c,0x86,0xa7,0x0d,0x0a]`

Ответ на эту команду можно не расшифровывать. Приложение TTLock этого не делает. Более того, у меня не получилось расшифровать это сообщение ни стандартным ключом, ни полученным от замка позже.

Зачем нужен ответ? Возможно, чтобы узнать тип замка. Думаю, его можно игнорировать.

#### 2. COMM_GET_AES_KEY

* Код: `0x19`
* Параметры:
    - вендор (строка `"SCIENER"`)

В поле `data` этой команды мы помещаем строку "SCIENER", зашифрованную алгоритмом AES на стандартном ключе (напомню, стандартный ключ AES = `[0x98,0x76,0x23,0xe8,0xa9,0x23,0xa1,0xbb,0x3d,0x9e,0x7d,0x03,0x78,0x12,0x45,0x88]`).

Должно получиться `[0x26,0xed,0x1d,0x92,0x58,0xa1,0xe4,0x28,0xfc,0xb4,0x8b,0xbd,0x1c,0x0e,0x5a,0x46]` (эту последовательность байт видно в примере команды). Размер команды - 16 байт, поэтому поле length = 0x10.

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x01,0x00,0x01,0x00,0x01,0x19,0xaa,0x10,0x26,0xed,0x1d,0x92,0x58,0xa1,0xe4,0x28,0xfc,0xb4,0x8b,0xbd,0x1c,0x0e,0x5a,0x46,0xbc,0x0d,0x0a]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x19,0xaa,0x20,0xfd,0xa0,0xe8,0x17,0xd5,0x96,0xe2,0xf4,0x8c,0xac,0xbb,0x58,0xef,0x5f,0xf5,0x69,0x5b,0x9f,0x0a,0x3f,0x4d,0x4f,0xd4,0x61,0x25,0x83,0xcf,0x18,0xed,0x6e,0x5e,0xe9,0xf7,0x0d,0x0a]`

Ответ приходит зашифрованный стандартным AES ключом.
* поле `data` ответа: `[0xfd,0xa0,0xe8,0x17,0xd5,0x96,0xe2,0xf4,0x8c,0xac,0xbb,0x58,0xef,0x5f,0xf5,0x69,0x5b,0x9f,0x0a,0x3f,0x4d,0x4f,0xd4,0x61,0x25,0x83,0xcf,0x18,0xed,0x6e,0x5e,0xe9]`
* После расшифрования: `[0x19,0x01,0xc0,0x15,0x7a,0x26,0x89,0x3a,0xde,0x25,0xc8,0xc4,0x71,0x7e,0xb8,0xdf,0x0b,0x84]`

Здесь:
* `0x19` - код команды
* `0x01` - статус выполнения команды (успех)
* `[0xc0,0x15,0x7a,0x26,0x89,0x3a,0xde,0x25,0xc8,0xc4,0x71,0x7e,0xb8,0xdf,0x0b,0x84]` - новый сеансовый ключ, которым мы будем пользовать для шифрования и расшифрования всех последующих сообщений.

#### 3. COMM_ADD_ADMIN

* Код: `0x56`
* Параметры:
    - `adminPassword`
    - `unlockKey`
    - вендор (строка `"SCIENER"`)

`adminPassword` и `unlockKey` - случайно сгенерированные 10-значные пароли:

```java
String adminPassword = new String(DigitUtil.generateDynamicPassword(10));
String unlockKey = new String(DigitUtil.generateDynamicPassword(10));
```

##### Код для генерации паролей:

```java
public static byte[] generateDynamicPassword(int length) {
    byte[] res = new byte[length];
    res[0] = 0x30;
    int i;
    for(i = 1; i < length; ++i) {
        double digit = Math.random() * 10;
        if(digit >= 10) {
            digit = 9;
        }

        res[i] = (byte)(((int)(digit + 48)));
    }

    return res;
}
```

##### Формирование команды:

```java
byte[] data = new byte[15];
byte[] adminPasswd = DigitUtil.integerToByteArray(Integer.valueOf(adminKey).intValue());
byte[] unlockCode = DigitUtil.integerToByteArray(Integer.valueOf(unlockKey).intValue());
System.arraycopy(((Object)adminPasswd), 0, ((Object)data), 0, adminPasswd.length);
System.arraycopy(((Object)unlockCode), 0, ((Object)data), 4, unlockCode.length);
System.arraycopy("SCIENER".getBytes(), 0, ((Object)data), 8, 7);
```

##### Конвертация паролей в массив байт:

```java
public static byte[] integerToByteArray(int integer) {
    byte[] res = new byte[4];
    byte[] bitShift = new byte[]{24, 16, 8, 0};
    int byteIdx;
    for(byteIdx = 0; byteIdx < 4; ++byteIdx) {
        res[byteIdx] = (byte)(integer >> bitShift[byteIdx]);
    }

    return res;
}
```

Например, у нас сгенерировались `adminPassword` = "0970456745" и `unlockKey` = "0344382141". Тогда после конвертации они будут иметь вид `[0x39,0xd7,0xfe,0xa9]` и `[0x14,0x86,0xda,0xbd]` соответственно. Эти массивы байт нужно склеить вместе, и добавить к ним строку "SCIENER" - и мы получим готовое поле `data`.

Тогда поле `data` целиком будет иметь вид `[0x39,0xd7,0xfe,0xa9,0x14,0x86,0xda,0xbd,0x53,0x43,0x49,0x45,0x4e,0x45,0x52]`

Зашифрованное поле `data` в нашем случае будет иметь вид `[0xf2,0x72,0xcb,0xaa,0x95,0x81,0x94,0xeb,0x37,0x8a,0x57,0x89,0x65,0x54,0x88,0x97]`

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x56,0xaa,0x10,0xf2,0x72,0xcb,0xaa,0x95,0x81,0x94,0xeb,0x37,0x8a,0x57,0x89,0x65,0x54,0x88,0x97,0xc5,0xd,0xa]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x79,0x46,0x14,0x1f,0x51,0x97,0x73,0xb0,0x2b,0xda,0x22,0x40,0x28,0xf0,0x6e,0x8d,0xb7,0xd,0xa]`

Расшифрование поля `data` из ответа даёт нам следующее: `[0x56,0x01,0x53,0x43,0x49,0x45,0x4e,0x45,0x52]`

Здесь:
* `0x56` - код команды
* `0x01` - статус выполнения команды (успех)
* `0x53,0x43,0x49,0x45,0x4e,0x45,0x52]` - строка "SCIENER"

#### 4. COMM_TIME_CALIBRATE

* Код: `0x43`
* Параметры:
  - Текущие дата и время в специальном формате (описание ниже)

##### Преобразование даты и времени

Для начала, необходимо получить текущие дату и время в формате "yyMMddHHmmss". Например, 11 Ноября 2020 года, 19:52:39 будет записано как "201111195239".

Затем эту строку необходимо превратить в массив байт следующим образом:

```java
public static byte[] convertTimeToByteArray(String timeStr) {
    int len = timeStr.length() / 2;
    byte[] res = new byte[len];
    int i;
    for(i = 0; i < len; ++i) {
        int _2i = i * 2;
        res[i] = (byte)Byte.valueOf(timeStr.substring(_2i, _2i + 2));
    }

    return res;
}
```

То есть из строки вида "201111195239" должен получиться массив байт вида `[0x14,0x0b,0x0b,0x13,0x34,0x27]`

Затем этот массив байт шифруется сеансовым AES ключом, после чего поле `data` у команды будет таким:

`[0x18,0x84,0xc3,0xfd,0x05,0x26,0xe4,0x1d,0x6e,0x20,0xf8,0x70,0x60,0x7d,0x7c,0x73]`

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x43,0xaa,0x10,0x18,0x84,0xc3,0xfd,0x05,0x26,0xe4,0x1d,0x6e,0x20,0xf8,0x70,0x60,0x7d,0x7c,0x73,0xeb,0xd,0xa]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x1e,0x34,0x6f,0x19,0x03,0x66,0x6d,0x5f,0x41,0x79,0x72,0x77,0x41,0x30,0x7d,0xc7,0xb9,0xd,0xa]`

Поле `data` после расшифровки сеансоывым AES ключом будет иметь вид:

`[0x43,0x01,0x64]`, где

* `0x43` - код команды
* `0x01` - статус выполнения команды (success)
* `0x64` - скорее всгео, уровень заряда аккумулятора замка (приложение TTLock игнорирует это поле, и мы поступим так же)

#### 5. COMM_CHECK_USER_TIME

* Код: `0x55`
* Параметры:
  - `startDate`
  - `endDate`
  - `lockFlagPos`
  - UID

##### Формирование команды

Перед шифрованием поле `data` имеет следующий формат:

|Размер|Параметр|Примечание|
|-|-|-|
|5 байт|`startDate`|всегда `[0x00,0x01,0x1f,0x14,0x00]`|
|5 байт|`endDate`|всегда `[0x63,0x0b,0x1e,0x14,0x00]`|
|3 байта|`lockFlagPos`|всегда `[0x00, 0x00, 0x00]`|
|4 байта|UID|Нужен для шаринга замков через Интернет, можно один раз сгенерировать случайно|

`startDate` и `endDate` всегда одинаковые. Если интересно, они генерируются следующим образом:

```java
long startDate = 949338000000L;
long endDate = 4099741200000L;

String startDateStr = new SimpleDateFormat(new Date(startDate)).format("yyMMddHHmm");
String endDateStr = new SimpleDateFormat(new Date(endDate)).format("yyMMddHHmm");

byte[] data = new byte[17];
System.arraycopy(DigitUtil.convertTimeToByteArray(startDateStr + endDateStr), 0, data, 0, 10);
```

Функция `convertTimeToByteArray` приведена в пункте [Преобразование даты и времени](#%D0%9F%D1%80%D0%B5%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B4%D0%B0%D1%82%D1%8B-%D0%B8-%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%B8)

UID - некоторое число. Оно преобразуется в массив байт так же, как описано в [шаге 3](#3-comm_add_admin).

В нашем примере UID = 4377454, поэтому в `data` последние 4 байта будут `[0x00,0x42,0xcb,0x6e]`

В итоге, `data` будет такой: `[0x00,0x01,0x1f,0x14,0x00,0x63,0x0b,0x1e,0x14,0x00,0x00,0x00,0x00,0x00,0x42,0xcb,0x6e]`

`data` после шифрования сеансовым ключом: `[0x57,0x49,0xc9,0x72,0xd5,0x75,0x15,0x6b,0x85,0x50,0xc1,0x53,0x41,0xcc,0xd8,0x03,0xa3,0x55,0x1e,0x09,0x93,0x75,0x5e,0xba,0xa1,0xd0,0x82,0x8e,0xc0,0x34,0xbc,0x64]`

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x55,0xaa,0x20,0x57,0x49,0xc9,0x72,0xd5,0x75,0x15,0x6b,0x85,0x50,0xc1,0x53,0x41,0xcc,0xd8,0x03,0xa3,0x55,0x1e,0x09,0x93,0x75,0x5e,0xba,0xa1,0xd0,0x82,0x8e,0xc0,0x34,0xbc,0x64,0x76,0xd,0xa]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x22,0x65,0x3f,0x6f,0xf5,0xdc,0x15,0x47,0x95,0xe5,0x5f,0x72,0x82,0xd4,0xac,0x2e,0xf2,0xd,0xa]`

Поле `data` после расшифрования сеансоывым AES ключом будет иметь вид:

`[0x55,0x01,0x00,0x00,0xb7,0x01]`, где:

* `0x55` - код команды
* `0x01` - статус (success)
* `[0x00,0x00,0xb7,0x01]` - сеансовый пароль ключа. Видимо, он используется в следующих командах, стоит его сохранить.

#### 6. COMM_CHECK_RANDOM

* Код: `0x30`
* Параметры:
  - `unlockKey`
  - сеансовый пароль замка

##### Формирование команды

Перед шифрованием поле `data` имеет следующий формат: 

|Размер|Параметр|Примечание|
|-|-|-|
|4 байта|Код разблокировки|Генерируется из `unlockKey` и сеансового пароля замка, алгоритм описан ниже|

###### Генерация кода разблокировки

Код разблокировки генерируется из `unlockKey` и сеансового пароля замка. `unlockKey` мы сгенерировали при инициализации замка [на шаге 3](#3-comm_add_admin). Сеансовый пароль мы получили от замка [на предыдущем шаге](#5-comm_check_user_time). Код разблокировки генерируется так:

TL;DR: сеансовый пароль конвертируется в число, складывается с `unlockKey`, результат превращается в массив байт.

```java
String lockPwd = DigitUtil.getUnlockPwd_new(DigitUtil.fourBytesToLong(psFromLock), Long.valueOf(unlockKey).longValue());
byte[] lockPwdArr = DigitUtil.integerToByteArray(Integer.valueOf(lockPwd).intValue());
```

Функция `integerToByteArray` приведена [здесь](#конвертация-паролей-в-массив-байт).

Функции `getUnlockPwd_new` и `fourBytesToLong` приведены ниже:

```java
public static String getUnlockPwd_new(long arg0, long arg2) {
    return String.valueOf(((int)(arg0 + arg2)));
}

public static long fourBytesToLong(byte[] arg5) {
    long res = ((long)(arg5[0] << 24)) & 0xFF000000L | 0L | ((long)(arg5[1] << 16 & 0xFF0000)) | ((long)(arg5[2] << 8 & 0xFF00)) | ((long)(arg5[3] & 0xFF));
    return res;
}
```

**Пример:** Допустим, мы сгенерировали `unlockKey` = "0344382141", и замок прислал нам сеансовый пароль = `[0x00,0x00,0xb7,0x01]`.

После преобразования в число с помощью `fourBytesToLong` получим сеансовый пароль = 46849.

После этого мы складываем 344382141 + 46849 и получаем 344428990.

Затем конвертируем 344428990 в массив байт с помощью `integerToByteArray`, и получаем `[0x14,0x87,0x91,0xbe]`.

В итоге, `data` будет такой: `[0x14,0x87,0x91,0xbe]`

После шифрования сеансовым AES ключом получим такое поле `data`: `[0x30,0xfa,0x81,0x65,0xe9,0xbd,0x4a,0x50,0x02,0x2c,0x38,0x15,0x52,0x0b,0x0c,0xc3]`

**Пример итогвой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x30,0xaa,0x10,0x30,0xfa,0x81,0x65,0xe9,0xbd,0x4a,0x50,0x02,0x2c,0x38,0x15,0x52,0x0b,0x0c,0xc3,0x52,0xd,0xa]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x7e,0x1d,0x53,0x3d,0x8c,0x07,0x94,0xcb,0x02,0x5d,0x83,0x47,0xfa,0xf0,0xbb,0xdb,0x66,0xd,0xa]`

Поле `data` после расшифрования сеансоывым AES ключом будет иметь вид:

`[0x30,0x01,0x64]`, где:

* `0x30` - код команды
* `0x01` - статус (success)
* `0x64` - уровень заряда батареи, в процентах

#### To be continued...

### Поднятие замка

Для того, чтобы поднять замок, необходимо отправить на него следующие команды:

#### 1. COMM_CHECK_ADMIN

* Код: `0x41`
* Параметры:
  - Пароль администратора
  - `lockFlagPos`
  - UID

##### Формирование команды

Перед шифрованием поле `data` имеет следующий формат: 

|Размер|Параметр|Примечание|
|-|-|-|
|4 байта|Пароль администратора|Генерируется при инициализации замка [на шаге 3](#3-comm_add_admin)|
|3 байта|`lockFlagPos`|Нужен для шаринга замков через Интернет, всегда `[0x00, 0x00, 0x00]`|
|4 байта|UID|Нужен для шаринга замков через Интернет, можно один раз сгенерировать случайно|

Здесь отправляем пароль администратора, сгенерированный [на шаге 3](#3-comm_add_admin) при инициализации замка. Как сконвертировать его в массив байт, описано там же.
Например, если у нас был пароль "0970456745", первые 4 байта в `data` будут следующими: `[0x39,0xd7,0xfe,0xa9]`

Следующие три байта выставляем в `[0x00, 0x00, 0x00]`.

Следующие 4 байта - UID - идентификатор пользователя. Его возвращает сервер при регистрации или логине в приложении. Мы не работаем с сервером, поэтому, наверное, можно один раз сгенерировать какой-то случайный UID и использовать его во всех последующих командах. В логах, из которых я вытащил команды, UID был 4377454, поэтому в `data` последние 4 байта были `[0x00,0x42,0xcb,0x6e]` (преобразование числа в массив байт здесь происходит так же, как описано в [шаге 3](#3-comm_add_admin))

В этом примере `data` будет иметь следующий вид: `[0x39,0xd7,0xfe,0xa9,0x00,0x00,0x00,0x00,0x42,0xcb,0x6e]`

Затем этот массив байт шифруется сеансовым AES ключом, после чего поле `data` у команды будет таким:

`[0xb8,0xf7,0x88,0x4d,0xf2,0x66,0x5e,0x9f,0x5f,0xbc,0xef,0x94,0x20,0x9b,0xcb,0x64]`

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x41,0xaa,0x10,0xb8,0xf7,0x88,0x4d,0xf2,0x66,0x5e,0x9f,0x5f,0xbc,0xef,0x94,0x20,0x9b,0xcb,0x64,0x46,0xd,0xa]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x57,0x58,0x01,0x3d,0x16,0xf2,0xab,0x5a,0xad,0x9b,0x80,0xeb,0xaa,0xe7,0xb3,0xa8,0x09,0xd,0xa]`

Поле `data` после расшифрования сеансоывым AES ключом будет иметь вид:

`[0x41,0x01,0x00,0x00,0xce,0x2b]`, где:

* `0x41` - код команды
* `0x01` - статус (success)
* `[0x00,0x00,0xce,0x2b]` - сеансовый пароль замка

Сеансовый пароль испольузется в следующей команде. Каждый раз замок присылает новый сеансовый пароль, чтобы его нельзя было заблокировать или разблокировать, отправив повторно одну и ту же команду.

#### 2. COMM_FUNCTION_LOCK

* Код: `0x58`
* Параметры:
  - `unlockKey`
  - сеансовый пароль замка
  - `unlockDate`

##### Формирование команды

Перед шифрованием поле `data` имеет следующий формат: 

|Размер|Параметр|Примечание|
|-|-|-|
|4 байта|Код разблокировки|Генерируется из `unlockKey` и сеансового пароля замка так же, как и [в COMM_CHECK_RANDOM]((#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-2)|
|4 байта|unlockDate|текущая метка времени (?), формат описан ниже|

Первые 4 байта - код разблокировки, алгоритм его генерации описан [в COMM_CHECK_RANDOM]((#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-2).

**Пример:** Допустим, мы сгенерировали `unlockKey` = "0344382141", и замок прислал нам сеансовый пароль = `[0x00,0x00,0xce,0x2b]`.

После преобразования в число с помощью `fourBytesToLong` получим сеансовый пароль = 52779.

После этого мы складываем 344382141 + 52779 и получаем 344434920.

Затем конвертируем 344434920 в массив байт с помощью `integerToByteArray`, и получаем `[0x14,0x87,0xa8,0xe8]`.

Это и будут первые 4 байта поля data.

Вторые 4 байта - `unlockDate`, некая метка времени. Я не смог понять из кода, какая именно метка здесь используется. Скорее всего, достаточно отправить текущую. 

###### Генерация метки времени

Следующие 4 байта поля `data` - метка времени. Как её получить:
* Взять количество секунд с Epoch (в java - `System.currentTimeMillis() / 1000`, в python - `int(time.time())`). 
* Сконвертировать в массив байт, функцией `integerToByteArray` (функция описана [здесь](#конвертация-паролей-в-массив-байт))

Допустим, у нас была метка времени 1605113559. Тогда после преобразования у нас получится массив байт `[0x5f,0xac,0x16,0xd7]` - это следующие 4 байта поля `data`.

В итоге, поле `data` будет иметь вид `[0x14,0x87,0xa8,0xe8,0x5f,0xac,0x16,0xd7]`

После шифрования сеансовым AES ключом получим поле `data` вида `[0xc9,0x81,0x3d,0xe3,0xda,0x71,0x7f,0x2a,0xe9,0x4d,0x7a,0xf4,0xf8,0xa2,0x4d,0x2a,0xd,0xa]`

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x58,0xaa,0x10,0xc9,0x81,0x3d,0xe3,0xda,0x71,0x7f,0x2a,0xe9,0x4d,0x7a,0xf4,0xf8,0xa2,0x4d,0x2a,0x6e,0xd,0xa]`

##### Формат ответа

В ответ на эту команду замок присылает сразу два ответа (???):

**Пример ответов от замка:**
* `[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x20,0xd9,0x5e,0x73,0xe0,0x46,0x1f,0x88,0x68,0x32,0xeb,0x31,0x29,0x43,0xc3,0x09,0x15,0x09,0x15,0x3d,0x3a,0x0c,0xb2,0x67,0x84,0x1c,0x63,0xfe,0xf3,0x18,0x55,0xa7,0x4b,0xf0,0xd,0xa]`
* `[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x10,0x9a,0x96,0xab,0x9c,0xdd,0x21,0x04,0xdf,0x12,0xb4,0xf6,0xfb,0xa1,0x29,0xc4,0x3f,0x38,0xd,0xa]`

Первый ответ - обычный. Это статус выполнения команды. Второй ответ - сообщение о состоянии замка (поднят\опущен). Разребёрся с ответами по порядку:

###### Первый ответ

Поле `data` после расшифрования сеансоывым AES ключом будет таким:

`[0x58,0x01,0x64,0x00,0x42,0xcb,0x6e,0x5f,0xac,0x16,0xd7,0x14,0x0b,0x0b,0x13,0x34,0x39]`

Здесь:

|offset|размер|значение|описание|
|-|-|-|-|
|0|1 байт|`0x58`|код команды|
|1|1 байт|`0x01`|статус (success)|
|2|1 байт|`0x64`|заряд батареи в процентах|
|3|4 байта|`[0x00,0x42,0xcb,0x6e]`|`uid` - не используется|
|7|4 байта|`[0x5f,0xac,0x16,0xd7]`|`uniqueId` - приложение отправляет это на сервер, в остальном не используется|
|11|1 байт|`0x14`|последние 2 цифры года|
|12|1 байт|`0x0b`|№ месяца (`0x0b` = 11 = ноябрь)|
|13|1 байт|`0x0b`|число месяца|
|14|1 байт|`0x13`|часы|
|15|1 байт|`0x34`|минуты|
|16|1 байт|`0x39`|секунды|

Начиная с 11 байта в ответе лежит метка времени. Зачем она нужна - кто знает.

###### Второй ответ

Поле `data` после расшифрования сеансоывым AES ключом будет таким:

`[0x14,0x01,0x64,0x00,0x00]`

Здесь:

* `0x14` - код команды (COMM_SEARCH_BICYCLE_STATUS) (не спрашивайте, почему так называется, я сам не знаю)
* `0x01` - статус (success)
* `0x64` - заряд батареи в процентах
* `0x00` - состояние замка (`0x00` = "поднят", `0x01` = "опущен") (???)
* `0x00` - не используется

### Опускание замка

Для того, чтобы опустить замок, необходимо отправить на него следующие команды:

#### 1. COMM_CHECK_ADMIN

Команда точно такая же, как и при [поднятии замка](#1-comm_check_admin)

В ответ на эту команду замок пришлёт сеансовый пароль, который мы будем использовать в следующей команде.

#### 2. COMM_UNLOCK

* Код: `0x47`
* Параметры:
  - `unlockKey`
  - сеансовый пароль замка
  - `unlockDate`
  
##### Формирование команды

Перед шифрованием поле `data` имеет следующий формат: 

|Размер|Параметр|
|-|-|
|4 байта|Код разблокировки|
|4 байта|`unlockDate`|текущая метка времени (?), формат такой же, как и у [метки времени в COMM_FUNCTION_LOCK](#генерация-метки-времени)|
|6 байт|`unlockDate` c поправкой на часовой пояс|

Код разблокировки генерируется из `unlockKey` и сеансового пароля замка так же, как и в [в COMM_CHECK_RANDOM](#%D0%A4%D0%BE%D1%80%D0%BC%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%BC%D0%B0%D0%BD%D0%B4%D1%8B-2)

Например, в прошлой команде замок вернул сеансовый пароль `[0x00,0x00,0x7a,0xa7]`. `unlockKey` был `[0x14,0x86,0xda,0xbd]`. Тогда код разблокировки должен получиться `[0x14,0x87,0x55,0x64]`.

`unlockDate` нужно сконвертировать в массив байт так же, как [метку времени в COMM_FUNCTION_LOCK](#генерация-метки-времени)

Например, если `unlockDate` была = 1605113597000, после конвертации должно получиться `[0x5f,0xac,0x16,0xfd]`.

Последнее поле - `unlockDate` c поправкой на часовой пояс. Чтобы его сгенерировать, берём метку `unlockDate` и превращаем её в формат "yyMMddHHmmss". Например, для метки времени 1605113597000 получится строка "201111195317". После этого конвертируем эту строку в массив байт по такому же алгоритму, как в [пунктк 4 инициализации замка](#преобразование-даты-и-времени). Должно получиться `[0x14,0x0b,0x0b,0x13,0x35,0x11]`

В итоге, в нашем примере поле `data` будет иметь вид `[0x14,0x87,0x55,0x64,0x5f,0xac,0x16,0xfd,0x14,0x0b,0x0b,0x13,0x35,0x11]`.

После зашифрования сеансовым ключом поле `data` будет иметь вид `[0x98,0xe4,0x2b,0xa9,0x39,0x9e,0xad,0x76,0xdc,0xa4,0x0c,0xdd,0x47,0x5f,0x47,0x9f]`

**Пример итоговой команды:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x47,0xaa,0x10,0x98,0xe4,0x2b,0xa9,0x39,0x9e,0xad,0x76,0xdc,0xa4,0x0c,0xdd,0x47,0x5f,0x47,0x9f,0xb3,0xd,0xa]`

##### Формат ответа

**Пример ответа от замка:**
`[0x7f,0x5a,0x05,0x03,0x07,0x00,0x01,0x00,0x01,0x54,0xaa,0x20,0x06,0xef,0xfb,0xb7,0x44,0x86,0x30,0x3c,0xcc,0xaa,0xa9,0x00,0xa1,0xa4,0xa8,0x74,0x45,0x10,0xf4,0x24,0x96,0x6f,0x1e,0x76,0x30,0xa1,0xc6,0x82,0x0c,0x30,0x0f,0x02,0xd4,0xd,0xa]`

Поле `data` после расшифрования получится таким:

`[0x47,0x01,0x64,0x00,0x42,0xcb,0x6e,0x5f,0xac,0x16,0xfd,0x14,0x0b,0x0b,0x13,0x35,0x12]`

Здесь:
|offset|размер|значение|описание|
|-|-|-|-|
|0|1 байт|`0x47`|код команды|
|1|1 байт|`0x01`|статус (success)|
|2|1 байт|`0x64`|заряд батареи в процентах|
|3|4 байта|`[0x00,0x42,0xcb,0x6e]`|`uid` - не используется|
|7|4 байта|`[0x5f,0xac,0x16,0xd7]`|`uniqueId` - приложение отправляет это на сервер, в остальном не используется|
|11|6 байт|`[0x14,0x0b,0x0b,0x13,0x35,0x12]`|Метка времени в таком же формате, как и в [ответе на COMM_FUNCTION_LOCK](#пример-ответа-5)|

