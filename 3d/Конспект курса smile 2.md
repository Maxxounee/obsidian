## Хоткеи



## Видео 2_1. Фото референса. Модификатор Subdivision Surface. Создание сферы

* Референс можно открыть в окне Image Editor

* Во время bevel (фаски) можно крутить колесо мыши, чтобы изменить количество граней

### Модификатор subdivision surface

Добавляет дополнительные грани, тем самым скругляя объект
* Модификаторы - generate - subdivision surface

Можно регулировать значение для определенных граней:
* В режиме редактирования выбираем грань
* В правом верхнем меню Item регулируем ползунок crease
* Так же уточнить, что у модификатора установлена галочка use creases
* Важно! - Параметр crease будет работать только в blender. При переносе в другие программы этой настройки уже не будет. Так что регулировать скругление/остроту лучше через дополнительные грани/фаску

### Создание сферы
* Создаем куб
* Добавляем модификатор subdivision surface
* Добавляем модификатор deform - cast (для полноценного придания формы сфере)
* Добавляем модификатор (через ПКМ) shade auto smooth