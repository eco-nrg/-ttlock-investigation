# car parking lock (parkinglock.qh.com.parkinglock)

## Описание протокола

Так же, как и TTLock, приложение связывается с умными замками через Bluetooth Low Energy (BLE). 

### Доступные Сервисы

|Сервис|Название|Примечание|
|-|-|-|
|0000fff0-0000-1000-8000-00805f9b34fb|SERVICE_DATA|Основной сервис для отправки и получения данных|
|0000180f-0000-1000-8000-00805f9b34fb|SERVICE_BATTERY|Сервис для получения информации об аккумуляторе замка|
|0000ffe0-0000-1000-8000-00805f9b34fb|SERVICE_PWD1|Сервис для настройки и проверки пароля|

#### Характеристики `SERVICE_DATA`

|Характеристика|Название|Примечание|
|-|-|-|
|0000fff1-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_PWD2||
|0000fff6-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_PWD||
|0000fff7-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_DATA||
|0000fff8-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_WRGB||

#### Характеристики `SERVICE_BATTERY`

|Характеристика|Название|Примечание|
|-|-|-|
|00002a19-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_BATTERY|Урвоень заряда|

#### Характеристики `SERVICE_PWD1`

|Характеристика|Название|Примечание|
|-|-|-|
|0000ffe1-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_PWDE1||
|0000ffe2-0000-1000-8000-00805f9b34fb|CHARACTERISTICS_PWD1||

#### Дескрипторы

|Дескриптор|Название|Примечание|
|-|-|-|
|00002902-0000-1000-8000-00805f9b34fb|CLIENT_CHARACTERISTIC_CONFIG||

## Описание команд

### readDeviceBattry

* Чтение
* `SERVICE_BATTERY`
* `CHARACTERISTICS_BATTERY`
* Ответ:
  * [battery]

У `SERVICE_BATTERY` запрашивается `CHARACTERISTICS_BATTERY`. В коллбеке `onCharacteristicRead` приходит сообщение длиной 1 байт. Это уровень заряда аккумулятора в процентах.

### checkPWD

* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_PWD`
* Параметры:
  * pwd - строка из 6 десятичных цифр
* Запрос:
  * [0xA0, pwd[0], pwd[1], pwd[2], pwd[3], pwd[4], pwd[5], 0x0A]

Проверяет пароль. Пароль должен состоять из 6 десятичных цифр (например, "123456"). Сообщение начинается с 0xa0, затем подряд идут 6 цифр пароля, последний байт - 0x0a

Пример на java:

```java
public void checkPWD(String pwd) {
  if(pwd == null || pwd.length() != 6) {
    return;
  }

  int ipwd = Integer.parseInt(pwd);
  writeCharacteristic(SERVICE_DATA, CHARACTERISTICS_PWD,
    new byte[]{
      0xA0, 
      ((byte)(ipwd / 100000)),
      ((byte)(ipwd / 10000 % 10)),
      ((byte)(ipwd / 1000 % 10)),
      ((byte)(ipwd / 100 % 10)),
      ((byte)(ipwd / 10 % 10)),
      ((byte)(ipwd % 10)), 10
    });
}
```

### resetPWD
* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_PWD`
* Параметры:
  * pwd - строка из 6 десятичных цифр
* Запрос:
  * [0xС0, pwd[0], pwd[1], pwd[2], pwd[3], pwd[4], pwd[5], 0x0С]

Сброс пароля. Пароль должен состоять из 6 десятичных цифр (например, "123456"). Сообщение начинается с 0xC0, затем подряд идут 6 цифр пароля, последний байт - 0x0C

Пример на java:

```java
public void resetPWD(String newpwd) {
  if(newpwd.length() != 6) {
    return;
  }

  byte[] v2 = {0xC0, 0, 0, 0, 0, 0, 0, 0x0C};
  int i;
  for(i = 0; i < 6; ++i) {
    v2[i + 1] = Byte.parseByte(newpwd.substring(i, i + 1));
  }

  this.writeCharacteristic(SERVICE_DATA, CHARACTERISTICS_PWD, v2);
}
```

### setAES
* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_WRGB`
* Параметры:
  * key - массив размером 16 байт, сеансовый ключ шифрования
* Запрос:
  * [0xF0, {16-byte key}, 0xF1]

Функция отправляет замку сообщение длиной 18 байт. Первый байт - 0xF0, затем 16-байтный сеансовый ключ, затем 0xF1. Вызывается при первой настройке замка.

Генерация сеансового ключа:
```java
private void updataSrc() {
  Random ra = new Random();
  int i;
  for(i = 0; i < this.srcAES.length; ++i) {
    int rad = ra.nextInt(90) + 1;
    this.srcAES[i] = (byte)(rad & 0xFF);    // сам сеансовый ключ
    this.sendsrcAES[i + 1] = (byte)(rad & 0xFF);  // переменная, которая хранит сообщение с сеансовым ключом
                                                  // (18 байт, первый байт 0xF0 и последний 0xF1)
  }
}

```

### setup
* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_PWD2`
* Параметры:
  * isUp - флаг
* Запрос:
  * [isUp ? 0x0A : 0x0B]

Используется Для поднятия и опускания замка (isUp == true чтобы поднять, false чтобы опустить)

## Неиспользуемые команды (?):

### getdata

* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_WRGB`
* Запрос:
  * [0x66, 0xF0, 0x00, 0x77]

???? Не используется (?)

### open

* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_WRGB`
* Параметры:
  * open - флаг
  * isForcible - флаг
* Запрос:
  * if (open):
    * [isForcible ? 0xFF : 0x00, 0x4F, 0x50, 0x45, 0x4E, 0, 0, 0, 0xF0, 0xFE]
  * else:
    * [isForcible ? 0xFF : 0x00, 0x43, 0x4C, 0x4F, 0x53, 0x45, 0, 0, 0xF0, 0xFE]

????? Не используется (?)

### setParam

* Запись
* `SERVICE_DATA`
* `CHARACTERISTICS_WRGB`
* Параметры:
  * isAutoClose - флаг
  * closeTime - байт
* Запрос:
  * [0xCF, (isAutoClose ? 0xF0 : 0x0F), closeTime, 0x00, 0x00, 0xFC]

????? Не используется (?)

## Первое подключение

При первом подключении к замку, происходит следующее:

### 1. Включаем уведомления для следующих характеристик:
* `SERVICE_DATA.CHARACTERISTICS_DATA`
* `SERVICE_PWD1.CHARACTERISTICS_PWDE1`
* `SERVICE_PWD1.CHARACTERISTICS_PWD1`

Пример на java:

```java
public void setNotify() {
  BluetoothGattService mService = this.mBluetoothGatt.getService(SERVICE_DATA);
  if(mService != null) {
    this.photoCharacteristic = mService.getCharacteristic(CHARACTERISTICS_DATA);
    if(this.photoCharacteristic != null) {
      this.mBluetoothGatt.setCharacteristicNotification(this.photoCharacteristic, true);
      BluetoothGattDescriptor descriptor = this.photoCharacteristic.getDescriptor(CLIENT_CHARACTERISTIC_CONFIG);
      descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
      this.mBluetoothGatt.writeDescriptor(descriptor);
    }
  }
}
```

Насколько я понял, после этого при изменении этих характеристик будет дёргаться коллбек `onCharacteristicChanged`

### 2. Генерируем сеансовый ключ и отправляем на замок

Вызываем функцию `setAES()` и передаём туда сгенерированный ключ. Java код для генерации ключа есть в описании функции `setAES()`

### 2.1. Получаем ответ от замка в `onCharacteristicChanged` для `SERVICE_DATA.CHARACTERISTICS_DATA`

Ответ имеет длину 18 байт и формат [0xF9, msg{16}, 0xF8]

Затем происходит проверка, что всё классно, и замок получил верный сеансовый ключ. Возможно, повторять эту проверку не обязательно, но на всякий случай логика следующая:

- Сеансовый ключ шифруется мастер-ключом, алгоритмом AES в режиме AES/ECB/NoPadding
- Мастер-ключ: [0xD0, 0xF9, 0x58, 0x8C, 0x59, 0xA2, 0x69, 0x26, 0x18, 0x53, 0xCB, 0xDA, 0x80, 0x82, 0x83, 0x39]
- Пример шифрования на java:

```java
public static byte[] Encrypt(byte[] sSrc) throws Exception {
  SecretKeySpec skeySpec = new SecretKeySpec(sKey, "AES");  // sKey - мастер-ключ
  Cipher cipher = Cipher.getInstance("AES/ECB/NoPadding");
  cipher.init(Cipher.ENCRYPT_MODE, skeySpec);
  byte[] encrypted = cipher.doFinal(sSrc);
  return encrypted;
}
```

- После этого проверяется, что зашифрованный сеансовый ключ совпадает с тем, что пришло от замка:

```java
yte[] mysrc = AES2.Encrypt(srcAES);  // srcAES - сеансовый ключ
for(v3_1 = 0; v3_1 < mysrc.length; ++v3_1) {
  if(mysrc[v3_1] != data[v3_1 + 1]) {
    ischeck = 0;
  }
}
```

### 3. Если проверка прошла успешно, сверяем пароль

Для этого выполняем `checkPWD()`. Пароль по умолчанию = 123456

### 3.1. Получаем ответ от замка в `onCharacteristicChanged` для `SERVICE_PWD1.CHARACTERISTICS_PWD1`

В ответ от замка приходит один байт. Его возможные значения:
* 30: Пароль неверный
* 26: Пароль верный
* 46: Не удалось изменить пароль
* 42: Пароль успешно изменён

### 4. Получаем уровень заряда батареи 

Вызываем `readDeviceBattry()`

## Открытие и закрытие замка

Когда подключение к замку установлено, его можно открыть или закрыть командой `setup`
