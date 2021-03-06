---
title: Как скриптовать гуглотаблицы
---

> У меня было 2 табличных процессора, 75 файлов с таблицами, домашняя бухгалтерия, аналитика инвестиций и гора бизнес-моделей, lean canvas и всего такого, всех цветов, а ещё результаты опросов в твиттере, статистика потребления памяти различными BGP-серверами с разных режимах и с разным количеством маршрутов, список дел, календарь и два планировщика. Не то, чтобы это всё было нужно в нормальной семейной жизни, но раз начал замещать софт таблицами, сделанными под себя, то иди в своём увлечении до конца. Единственное, что меня беспокоило — это app script. В мире нет никого более беспомощного, безответственного и безнравственного, чем человек пишущий на javascript для табличек. И я знал, что довольно скоро я и в это окунусь.

Вспомнил тут доклад с дампа 2018 - дендрофекальный подход. Там про прототипирование с помощью гуглотаблиц. Решил попробовать автоматизировать занудную вещь в моём планировщике - проставление даты последнего выполнения периодического дела (ну, типа не забывать к парикмахеру сходить). Ручками вбивать сегодняшнюю дату - это тяжело, там какой-то стрёмный виджет вылезает или печатать надо. А вот тыкнуть по кнопочке - вполне неплохо было бы.

**Как вообще начать кодить что-то**: "Инструменты" -> "Редактор скриптов". Там дайте название проекту, сохраните его и можно программировать.

Мне хватает стандартного триггера onEdit, который вызывается по ручному изменению ячейки. `e` - это event, из него можно получить указатели на текущий файл, оттуда на текущий лист, оттуда его название и сделать таким образом первичный фильтр, чтобы на остальных листах ничего не происходило. Также я ограничиваю "магию" 4 колонкой. В гуглотаблицах нет такого понятия как кнопка, поэтому у меня его симулирует чекбокс. С ним легко работать, по одному клику он вызывает нужный `onEdit()`. Чтобы дополнительно убедиться что это чекбокс я проверяю его на то, что он равен или `true` или `false`. В ячейку слева я прописываю текущую дату, а само значение сбрасываю (потому что ряд одинаковых пустых чекбоксов смотрится эстетичнее, чем разнобой значений).

``` javascript
function onEdit(e) {
  if (e.source.getActiveSheet().getName() != 'Периодика')
    return;
  if (e.range.getColumn() != 4)
    return;
  if (e.range.getValue() == true || e.range.getValue() == false) {
    e.range.offset(0, -1).setValue(new Date());
    e.range.setValue(false);
  }
}
```
