# Utils

## reactivity
```js
console.log(this.node.foo.reactivity)
```
В объекте this.node[‘nodename’].reactivity описаны все реактивные функции используемые в node. Если необходимо отменить реактивность для узла, достаточно удалить соответствующую функцию в этом объекте.

## deleteReactive
```js
deleteReactive(reactivity, keypath)
```

## exclude
```js
textContent: () => this.exclude(() => this.proxy.foo)
```
Эта функция позволяет получить данные прокси, не включая его в реактивность для функции в которой он используется.

## replicate
```js
import { replicate } from 'lesta'
…
const clone = replicate(nestedObject)
```
Функция replicate делает глубокое клонирование объекта с помощью сериализации. Работает только с типами данных которые поддерживает JSON, для клонирования объектов с другими типами можно использовать нативную javascript функцию structuredClone().

## delay
```js
import { delay } from 'lesta'
…
async add(notify) {
  this.proxy.notifications.unshift(notify)
  await delay(this.param.time)
  this.proxy.notifications.pop()
}
```
```js
if (incomplete) {
  this.param.delayFilter = delay(2000)
	this.param.delayFilter.then(() => { .. }).catch(()=> {})
} else if (!this.param.delayFilter.process) {…}
…
this.param.delayFilter?.stop()
```

## debounce
```js
import { debounce } from 'lesta'
…
nodes() {
  return {
    search: {
      oninput: debounce((e) => this.method.search(e.target.value), 300)
    }
  }
}
```

## throttling
```js
import { throttling } from 'lesta'
…
const throttled = throttle(() => console.log("Throttling test: "), 1000)
window.addEventListener('resize', throttled)
```
Функция throttling позволяет ограничить частоту вызовов функции до определенного значения. Это может помочь избежать проблем с производительностью при большом количестве вызовов и/или при работе с медленными операциями. Она также может быть полезной для случаев, когда нужно предотвратить ненамеренные двойные нажатия на кнопку или отправку нескольких запросов.
Функция throttling принимает два аргумента:
- функция, которую нужно ограничить по частоте вызовов.
- время в миллисекундах, через которое функция может быть вызвана повторно.

## loadModule
Функция для загрузки модулей. Если передать функция которая возвращает промис, то дождется выполнения промиса и вернет дефолтное значение. Если передать объект то вернет сам объект.


## uid
```js
import { uid } from 'lesta'
…
const id = uid()
```
С помощью этой функции можно получать случайные uid в формате xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx. Это формат UUID v4, который используется в стандарте RFC 4122.

## deepFreeze
```js
import { deepFreeze } from 'lesta'
…
const readOnlyObject = deepFreeze(nestedObject)
```
Если вам нужно сделать объект доступным только для чтения вы можете использовать нативную javascript функцию Object.freeze(). Для объектов имеющих вложенность используйте deepFreeze.

## mapProps
```js

```
Эта функция позволяет сгруппировать пропсы с одинаковыми опциями. Она используется только для создание более короткой записи и дает никаких дополнительных возможностей.

## deliver

## queue
