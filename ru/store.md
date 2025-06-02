# State Management

```js
import component from './app'
import tasks from './stores/tasks'
import notices from './stores/notices'
import { createApp, createStores } from 'lesta'
const root = document.querySelector('#root')
const app = createApp({...})
createStores(app, { tasks, notices })
app.mount(component, root)

```

Часто бывает необходимость передавать данные между компонентами, которые не находятся в прямой зависимости друг от другу. Для коммуникации между такими компонентами можно использовать общее хранилище данных (store).

Чтобы создать одно или несколько хранилищ, необходимо в функцию createStores передать объект. Свойствами такого объекта будут имена хранилищ, а значениями объекты с опциями.

> Инициализация хранилищ должна производиться до инициализации роутера.

> Для модулей хранилищ поддерживается динамический импорт. Это значит, что они будут загружены только при обращение из компонентов.


# Store Options
Объект хранилища имеет схожие опции с опциями обычного компонента. Свойства sources, params, proxies, proxies, methods работают аналогично таким же свойствам в компоненте.


## setters
```js
export default {
  setters: {
    'notices.length'(v) {
      //array length changed
      return v
     }
  },
  proxies: {
    notices: [],
  }
}
```
Перехват изменения прокси. Аналогично setters в компонентах.

## params
```js
export default {
  params: {
    delay: 3000
  }
}
```
В params можно описать любые данные не требующие отслеживания изменений. Доступны как this.param.

## proxies
```js
export default {
  proxies: {
    notices: [],
  }
}
```
В proxies описываются данные, изменение которых необходимо отслеживать. Измененные proxy в хранилище автоматически обновляются и в компонентах. Доступны как this.proxy.


## methods
```js
export default {
  methods: {
    removeNotice({ index }) {
      this.proxy.notices.splice(index, 1)
    }
  }
}
```
Функции описанные в methods служат для коммуникации от компонента к хранилищу.
Такие функции в качестве параметра принимают только объект. Доступны как this.method. Принимают только один параметр (объекта). Доступ к нужным свойствам можно получить с помощью деструктуризации.

## middlewares
```js
export default {
   middlewares: {
    async addTask({ task }) {
      const { description, name } = task
      return { task: await api.addTask(description, name) }
    }
  },
  methods: {
    addTask({ task }) {
      this.proxy.tasks.unshift(task)
    }
  }
}
```
Для каждого вызова функции метода предварительно вызываются одноименные функцию в объекте middlewares. Эти функции в качестве параметра принимаю теже данные, что и соответствующие методы. Поэтому эти данные можно предварительно обработать, например сделать запрос на сервер и уже в методах изменить состояния proxy объектов.

> Функции middlewares обязательно должны возвращать те же данные что и получили, эти данные уже будут приняты в качестве параметра в одноименных методах.


# Store Lifecycle Functions
```js
export default {
  …
  async loaded() {...},
  async created() {...}
}
```
Аналогично функциям жизненного цикла в компонентах, в store поддерживаются две функции:
- `loaded`
Вызывается до обработки всех опций. На этом этапе все опции модуля будут доступны в this.options.

- `created`
Вызывается после обработки params, proxies, methods. На этом этапе они доступны в this.

> Функции жизненного цикла вызываются только в момент первого обращения к данным в props компонента.

# Store in Component

```js
props: {
    proxies: {
      notices: { store: 'notices' },
    },
    methods: {
      removeNotice: { store: 'notices' },
    }
}
…
// this.proxy.notices[index]
// this.method.removeNotice({ index })
```
Коммуникация с хранилищем осуществляется через props, как если бы хранилищем был родительский компонент. Чтобы получить proxies, proxies, methods в компоненте нужно для каждого prop указать свойство `store` с именем хранилища.

> Модуль хранилища инициализирует тот компонент, который обратился первым.
