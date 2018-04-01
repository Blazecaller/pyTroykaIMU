pyTroykaIMU
==========

Библиотека на Python 3 для Raspberry Pi, позволяющая управлять [IMU-сенсором на 10 степеней свободы (Troyka-модуль)](http://amperka.ru/product/troyka-imu-10-dof)
от [Амперки](http://amperka.ru/).

IMU-сенсор (Inertial measurement unit — Инерционное измерительное устройство) — позволяет определить положение вашего девайса в пространстве. IMU-сенсор включает в себя:
- Гироскоп, определяющий угловую скорость вокруг собственных осей X, Y, Z.
- Акселерометр, определяющий величину ускорения свободного падения по осям X, Y, Z.
- Компас, определяющий углы между собственными осями сенсора X, Y, Z и силовыми линиями магнитного поля Земли
- Барометр, определяющий атмосферное давление, высоту над уровнем моря и температуру

![alt-текст](https://static-eu.insales.ru/images/products/1/799/58802975/troyka-imu-10-dof.1.jpg "IMU-сенсор на 10 степеней свободы (Troyka-модуль)")


Подключение
==========
Подключение библиотеки для работы с I2C шиной SMBus

Наберите в командной строке Python
```python
import smbus
```
Если интерпретатор выдал ошибку, то необходимо в установить дополнительную библиотеку через менеджер пакетов. Наберите в терминале:
```python
pip install smbus
```
Так же необходимо включить в Raspberry Pi шину через настройки:
- Запустите меню raspi-config через терминал
```python
sudo raspi-config
```

- Включите I2C

![alt-текст](http://wiki.amperka.ru/_media/продукты:troyka-gpio-expander:interfacing_1.png "Зайдите в настройки интерфейсов Вашего Raspberry")
![alt-текст](http://wiki.amperka.ru/_media/продукты:troyka-gpio-expander:i2c-02.png "Выберите интерфейс I2C")
![alt-текст](http://wiki.amperka.ru/_media/продукты:troyka-gpio-expander:enable_3.png "Подтвердите влючение")

- Подключение модуля

![alt-текст](https://preview.ibb.co/j7PKpc/IMU.png "Подключение аналогично любому модулю Troyka")

Для подключения большего большего количества Troyka модулей очень удобно использовать, например [Troyka Pad](http://amperka.ru/product/troyka-pad-1x4)

![alt-текст](https://static-eu.insales.ru/images/products/1/2757/98380485/troyka_pad_all_in.jpg "Troyka Pad")

Пример использования
====================
```python

from madgwickahrs import MadgwickAHRS
from pytroykaimu import TroykaIMU
 
imu = TroykaIMU()
filter = MadgwickAHRS(beta=1, sampleperiod=1/256)

while True:
    filter.update(imu.gyroscope.read_radians_per_second_xyz(),
                  imu.accelerometer.read_gxyz(),
                  imu.magnetometer.read_gauss_xyz())
    data = filter.quaternion.to_angle_axis()

    dataencode = str(data).encode('utf-8')
    if dataencode:
        print(data)

```

Состав библиотеки
====================
Название файла      | Содержание файла
--------------------|----------------------
igrf12py            | классы и утилиты для реализации стандартной геомагнитной модели поля Земли
gost4401_81.py      | класс реализация стандартной модели атмосферы по ГОСТ4401
l3g4200d.py         | класс гироскопа TroykaIMU модуля
lis3mdl.py          | класс магнитометра(компаса) TroykaIMU модуля
lis331dlh.py        | класс акселерометра TroykaIMU модуля
lps331ap.py         | класс барометра TroykaIMU модуля
madgwickahrs.py     | класс реализующий алгоритм Madgwick AHRS для определения положения в пространстве
pytroykaimu.py      | класс TroykaIMU модуля
quaternion.py       | класс реализации кватернионов и операций над ними



- Трёхосный гироскоп L3G4200D покажет скорость вращения относительно собственных осей X, Y и Z
- Трёхосный магнитометр/компас LIS3MDL покажет напряженность магнитного поля относительно собственных осей. Это поможет определить направление на Север
- Трёхосный акселерометр LIS331DLH покажет ускорение относительно собственных осей X, Y и Z. Это поможет определить направление к центру Земли 
- Барометр LPS331AP покажет атмосферное давление и поможет вычислить высоту над уровнем моря.


Калибровка магнитометра
==========
TroykaIMU оснащен магнитометром lis3mdl, который существенно расширяет возможности определения положения в пространстве. Но для точности данных, любой магнитометр нужно калибровать и желательно в тех же условиях в которых он будет эксплуатироваться "на месте".
На магнитометр действуют искажения которые принято делить на Hard Iron и Soft Iron. Ниже приведена картинка, иллюстрирующая суть этих искажений.

Hard Iron источники искажений магнитного поля воздействуя на магнитометр, придают некоторое смещение измеряемым значениям. Ликвидировать такое искажение очень просто: достаточно увеличить или уменьшить получаемые от прибора значения на величину смещения. Это смещение обозначают вектором bias.

Soft Iron искажение уже более сложное для вычислений и для его нивелирования вычисляется калибровочная матрица, которая позволяет преобразовать множество значений от прибора очень близко с сферойду. В нашей библиотеке эта матрица называется calibration_matrix.
![alt-текст](http://wiki.amperka.ru/_media/продукты:troyka-compass_calibration:compass_calibration1.jpg "Hard & Soft Iron distortion")  

Для вычисления этих параметров существуют различные методики. В нашем методе, мы соберем облако значений в файл и затем с помощью программы magneto вычислим калибровочные параметры.
Скачать программу можно по адресу тут [https://sites.google.com/site/sailboatinstruments1/home](https://sites.google.com/site/sailboatinstruments1/home)

Для сбора значений на вашей Raspberry Pi с подключенным IMU модулем замустите скрипт сервера калибровочных данных pyIMUDataServer.py
Также скачайте и установите MatLab для сбора значений и подготовки их в файл. Для этого в MatLab загрузите скрипт magneto.m
После запуска скрипта в MatLab нужно некоторое время покрутить IMU датчик в руках примерно 4 минуты в разных плоскостях. MatLab будет визуализировать этот процесс и в идеале нужно получить облако значений похожее на сферойд, как на рисунке ниже.
 ![alt-текст](https://pp.userapi.com/c845221/v845221853/116e0/xvJcfRLO_7o.jpg "Сбор значений в файл")
Все данные сохраняются в файл calibration.txt, его и надо загрузить в программу magneto для расчета каллибровочных коэффициентов.


IGRF12py
==========

Реализация класса стантартного [международного геомагнитного аналитического поля (IGRF)](https://ru.wikipedia.org/wiki/Международное_геомагнитное_аналитическое_поле) 
позволяющего более эффективно использовать магнитометр на TroykaIMU модуле.


![alt-текст](https://pbs.twimg.com/media/DNyvjhQVQAAl9Iu.png "pyigrf12")  

Про использования данных пожно почитать, например тут:
- http://geologyandpython.com/igrf.html
- http://geomag.nrcan.gc.ca/calc/calc-en.php


Обратная связь
==========

По вопросам о работе библиотеки или обнаруженным ошибкам писать на selidimail@gmail.com
