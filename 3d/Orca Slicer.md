
# Термины


## Горизонтальное расширение

Для настройки допусков используется отрицательное горизонтальное расширение.

**Quality - Precision - X-Y contour compensation**

Vase Mode (Spiral Vase) - печать непрерывным движением (others - spiral vase )
## Retract

Втягивание при холостом перемещении


# Особенности филамента
## PETG

[Хорошее видео по калибровкам от LenDizzle](https://www.youtube.com/watch?v=fhvpshjHYk4)


[Статья TRIPO](https://www.tripo3d.ai/blog/collect/how-to-print-petg-like-a-pro-6mc63ehzs88?utm_source=google&utm_medium=cpcpmaxpru&utm_campaign=all-11apphome&utm_term=&gad_source=2&gad_campaignid=22703867026&gclid=CjwKCAjwsZPDBhBWEiwADuO6y7WUmCkEoxuJJ_d70d5xadHvFk_zVFOZuhqtWQljAhIhZJn2GKmSyRoCd3oQAvD_BwE)

 * Layer Height от 0.2 и больше
 * Количество стенок - 3шт
 * Толщина верхних слоев - в соответствии с толщиной стенок
 * Плотность заполнения - около 10%
 * Модель заполнения - простая. Шестиугольник или куб
 * Скорость печати - начинать с 20мм/с для стенок
 * Температура - начинать с 230 и 70
 * retraction speed - около 25мм/с



Мои Petg наблюдения:

* Высота слоя больше 0.2. При 0.2 сломалась печать калибровки по температуре
* Температура 235
* 1 Flow Test - при flow ratio 1 получились хорошо от -5 до -15
# [Урок от Tech Blog](https://www.youtube.com/watch?v=LeLj5oCacak)


## Настройка пластика:
### Filament

* Коэффициент потока Flow ratio - не больше 0.95
* Diameter и Density (плотность) указывать в соответсвии с пластиком
* Max Volumetric speed (максимальная объемная скорость) настраивается после калибровок 

### Cooling
* No cooling for the first layer - 2-3 слоя

## Калибровка

### Temperature

Печатаем тестовую температурную модель, чтобы потом установить корректную температуру при печати. Затем значения заносятся в профиль пластика (Print Temperature - Nozzle)

### Flow Rate

Выбираем тест с множителями, затем донастраиваем значение flow ratio в настройках филамента

## Spiral Vase

* Others - spiral vase

[Designing for vase mode - 3D design for 3D printing](https://www.youtube.com/watch?v=dtU9N-2Mimw&list=PLI3cyZrNGXKxQYI6JHIC3y9D2CmIQ-H6M&index=21)

### Требования:
* Нет верхнего слоя
* Wall Loops - 1
* Quality - Line width - outer wall - ставить побольше для прочности. В видео советуют 0.7мм 
* Модель должна быть solid, а не hollow
* У модели должно быть плоское дно
* Должна быть одна outline line (чтобы принтер мог печатать непрерывно)
* Никаких горизонтальных поверхностей
 


## [# Cut Your 3D Printing Time In Half!](# Cut Your 3D Printing Time In Half!)

* Настройки принтера - Z hop - Z-hop heihgt - 0 (Default - 0.4)
* Quality - walls and surfaces - avoid crossing walls OK
* Quality - walls and surfaces - avoid crossing walls max detour length - 200mm
* Strength - infill - sparse infill density (для некоторых моделей)
* Strength - detect narrow internal solid infill - NO
* Ensure vertical shell thickness - NO
## [Hacks for perfect prints](https://www.youtube.com/watch?v=dWL2swAqcyY&list=PLI3cyZrNGXKxQYI6JHIC3y9D2CmIQ-H6M&index=32)


