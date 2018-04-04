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

Для вычисления этих параметров существуют различные методики. В нашем методе, на Raspberry Pi мы запустим сервер данных, чтобы на другом компьютере можно было получить значения измерений, записать их в файл, а затем с помощью программы magneto вычислить калибровочные параметры.
Скачать программу можно по адресу тут [https://sites.google.com/site/sailboatinstruments1/home](https://sites.google.com/site/sailboatinstruments1/home)

MatLab желательно использовать один из последних версий под управлением MacOS/Windows. Программу Magneto в MacOS можно запустить из под Wine.

Для сбора значений на вашей Raspberry Pi с подключенным IMU модулем замустите скрипт сервера калибровочных данных pyIMUCalibrationDataServer.py
Также скачайте и установите на вашем компьютере MatLab для сбора значений и подготовки их в файл. Для этого в MatLab загрузите скрипт magneto.m
После запуска скрипта в MatLab нужно некоторое время покрутить IMU датчик в руках примерно 4 минуты в разных плоскостях. MatLab будет визуализировать этот процесс и в идеале нужно получить облако значений похожее на сферойд.

Все данные сохраняются в файл calibration.txt, его и надо загрузить в программу magneto для расчета каллибровочных коэффициентов.

Калибровку можно производить в единицах сенсора RAW или конвертировать их в Гауссы.

Если возникли сложности, пройдем все поэтапно.
1. Загрузим с GitHub последнюю версию библиотеки в каталог с проектами. В каталоге calibration нужно запустить скрипт pyIMUCalibrationDataServer.py 
Если в консоли мы увидели 'waiting for connection...', то все идет как надо, сервер данных у нас заработал и перед переходом на рабочий компьютер узнаем IP адрес вашего микрокомпьютера Raspberry Pi с запущенным скриптом.
По умолчанию сервер выдает данные в Гаусах для магнитометра, и, если вам удобнее производить калибровку в целых единицах RAW датчика, исправьте как указано ниже.
```python
# Эту комментируем
# m_x, m_y, m_z = imu.magnetometer.read_gauss_xyz()

# А эту разкоментируем
m_x, m_y, m_z = imu.magnetometer.read_xyz()
```
2. В каталоге pyTroykaIMU/calibration/ лежит скрипт для MatLab, который служит для сбора данных по сети с сервера Raspberry Pi, но перед запуском его нужно настроить. В параметре HOST укажите IP адрес сервера вашей Raspberry Pi.
```matlab
% Enter Your Raspberry Pi pyIMUCalibrationDataServer host
HOST = '192.168.1.10';
```
3. IMU датчик должен быть на достаточно длинном кабеле и иметь надежное соединение, так как для сбора 3000 замеров, его придется вращать в разных плоскостях. Уберите всевозможные предметы, создающие дополнительные искажения магнитного поля в области вращения IMU сенсора, не менее полуметра. Это акустические системы, блоки питания а также некоторые мониторы. 
4. После того как все условия обеспечены и "железо" правильно настроено нажимаем на Run в MatLab и даблюдаем за постоением сферообразного облака в 3D графике. Вращать необходимо неспеша, и желательно в разных плоскостях. И если все сделано правильно. мы увидим окно "fugure1" и в каталоге со скрипптом MatLab будет необходимый файл calibration.txt

 ![alt-текст](https://pp.userapi.com/c847124/v847124296/172a0/-jSL54Ql6PE.jpg "Сбор значений в файл")

5. Далее нам понадобиться программа magneto. Выше указан адрес, по которому ее можно скачать бесплатно. В поле «Raw magnetic measurements» выбираем файл с исходными данными. В поле «Norm of Magnetic or Gravitational field» вводим величину магнитного поля Земли в точке нашей дислокации. Узнать это можно на разных WEB сервисах. В PyTroykaIMU есть библиотека igrf12py позволяющая самостоятельно расчитать эту величину в любой точке Земли. 
Мы воспользуемся этим сервисом [http://www.geomag.bgs.ac.uk](http://www.geomag.bgs.ac.uk/data_service/models_compass/wmm_calc.html)

 ![alt-текст](https://pp.userapi.com/c845520/v845520296/1c85c/pJitVeZTUvk.jpg "Узнаем напраженность поля в нашей местности")

Значение 49923 наноТесла переводим в Гауссы из соотношения 1 Тесла = 10 000 Гаусс. Получим общую напряженнность равную 0,49923 Гаусс

![alt-текст](https://pp.userapi.com/c847019/v847019296/174f5/bFBypUzZnvE.jpg "Вычисляем коэффициенты")

Даже можно не узнавать величину напряженности в вашей местности, а подогнать этот коэффициент так чтобы значения главной диагонали были близки к единице. Это не повлияет на точность определения значений датчика так как матрица изменит лишь свой масштаб.

6. Жмем кнопку Calibrate и получаем:
 - значения смещения по всем трем осям: Combined bias (b);
 - и матрицу масштаба и ортогонализации: Correction for combined scale factors, misalignments and soft iron (A-1).

```python
calibration_matrix = [[0.858751, 0.029588, 0.022668],
                      [0.029588, 0.871676, 0.001220],
                      [0.022668, 0.001220, 0.892654]]

bias = [0.265855, -0.356333, 0.586471]
```


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
