# Tutorial

Знакомство с LestaJS на примере разработки галереи. Полный код компонента галереи представлен в песочнице [CodePen](https://codepen.io/kossyak/pen/dPobVQY).

## Lesta Widget
```js
import { mountWidget, camelToKebab } from ... // импорт по CDN

function createGallery(element, options) {
  const { images, infinite, dots, arrows, time } = options
  const component = {...}
  mountWidget(component, element, { 
    selectors: (str) => '.' + camelToKebab(str) 
  })

}
```

Для реализации однокомпонентного решения лучше всего подойдет упрощенная (облегченная) версия фреймворка с базовыми возможностями. Подключаем widget-модуль по CDN ссылке, импортируя следующие функции:
- `mountWidget` монтирует компонент в указанный элемент.
- `camelToKebab` автоматически преобразует имена узлов из camelCase в kebab-case для поиска в DOM. (например lgImg > lg-img)

В функции mountWidget третьим аргументом ожидается глобальный объект приложения. Этот объект используется фреймворком для расширения стандартных возможностей компонента, использует зарезервированные свойства. Все остальные свойства становятся частью контекста компонента и доступны как this.app.<foo>

> `selectors` Зарезервированное свойство-функция, которая преобразует селекторы для поиска узлов в разметке. (По умолчанию: поиск по классам)

## createGallery
```js
createGallery(document.body, {
  images: [...], // массив с ссылками на изображения
  infinite: true, // переключения (последнее → первое).
  dots: true, // точечная навигация по изображениям.
  arrows: true, // кнопки "Вперед/Назад".
  time: 3000 // автоматическая смена изображений. (мс)
}
```

Для удобства создается отдельная функция, которая монтирует виджет галереи в указанный узел с заданными опциями.

Параметры:
- `element` httml-элемент для монтирования виджета галереи
- `options` опции, с которыми должны работать галерея

## Component API
Lesta использует классический декларативный подход и option API для описания объекта компонента и технологию реактивного DOM. Фреймворк находит все узлы описанные в функции nodes, а свойства узлов, в которых встречаются proxy (отслеживаемые данные) делает реактивными.

## template
```js
template: `<div class="lg-container">
      <div class="lg-btn"><button class="lg-btn-prev"></button></div>
      <div class="lg-img-wr"><img class="lg-img"></div>
      <div class="lg-btn"><button class="lg-btn-next"></button></div>
      <div class="lg-dots-wr"><nav></nav></div>
    </div>`,
```

HTML-код компонента (виджета). Свойство template можно не указывать, если разметка уже есть в узле для монтирования.

### nodes function

```js
nodes({ proxy }) { // const { proxy } = this
  return {
    lgImg: {...},
    lgBtnPrev: {...},
    lgBtnNext: {...},
    dots: {...}
  }
}
```
Функция возвращает объект, где ключи — имена узлов, а значения — их свойства.
Используя деструктуризацию получаем proxy из контекста.

Аргументы:
- `context` глобальный контекст компонента this. (this.proxy, this.param, this.method…)


## node properties

### selector
```js
nodes() {
  return {
    dots: {
      selector: '.lg-dots-wr nav', // найдет <nav> внутри элемента с классом lg-dots-wr
    }
  }
}
```
Зарезервирование свойство. Определяет селектор по которому можно найти элемент в DOM. Если не указан, используется class элемента.

### on*
```js
nodes({ proxy }) {
  return {
    lgBtnNext: {
      onclick: (event) => proxy.index++
    }
  }
}
```
Все стандартные DOM-события поддерживаются через свойства on*. Все события  создают объект event, который можно получить как аргумент функции.

### DOM property
```js
nodes({ proxy }) {
  return {
      dots: {
        hidden: !dots,
        innerHTML: () => images.reduce((accum, _, index) => accum + 
          `<button data-index="${index}" class="${proxy.index === index ? 'active' : ''}"></button>`, ''),
     }
  }
}
```
Генерация разметки на основе данных с помощью метода массива reduce. При изменении proxy, функция innerHTML будет перевыполнена и html содержимое узла будет обновлено.

> Изменяемые свойства узлов могут принимать функцию в качестве значения, если нужно сделать свойство реактивным.


### params
```js
params: {
  intervalID: false, // ID таймера для автоматической смены изображений
}
```
Внутреннее состояние компонента (не реактивное). Изменения параметров не вызывают реактивных функций.

### proxies
```js
proxies: {
  index: 0 // текущий индекс активного изображения
}
```
Реактивные свойства компонента. Их изменения автоматически обновляют состояния свойств в узлах.

### setters

```js
setters: {
  index(v) {
    const lastIndex = images.length - 1
    this.method.stopTimer()
    if (infinite || v !== lastIndex) this.method.startTimer()
    if (infinite) {
      if (v === images.length) return 0
      if (v === -1) return lastIndex
    }
    return v
  }
}
```
Сеттеры для реактивных свойств, позволяющие модифицировать значения перед установкой. При попытке изменить одноименный прокси происходит корректировка его окончательного значения.

## methods
```js
methods: {
  stopTimer() {
    clearInterval(this.param.intervalID)
  }
}
```
Внутренние методы компонента. Вызывается в компоненте как this.method.<name>.

## mounted

```js
mounted() {
  this.method.startTimer() // запускаем таймер при инициализации
}
```
Функция жизненного цикла. Вызывается после монтирования компонента.


> Здесь рассмотрена малая часть возможностей фреймворка, пожалуйста, ознакомьтесь с разделом документации [Lesta Basic](https://lesta.dev/basic)

