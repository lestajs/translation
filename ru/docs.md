# Quick start
```bash
npm i lesta
```
Чтобы использовать Lesta, загрузите последнюю версию или установите ее с помощью [npm](https://www.npmjs.com/package/lesta/).

Lesta использует классические модели декларативного и компонентного программирования. Работа фреймворка может начинаться с одной из двух функций: createWidget или createApp.

Код в документации используется из демонстрационных проектов, поэтому в разделах указаны ссылки на файлы, где этот код применяется.

## createWidget 
[CounterWidget/index.js](https://github.com/lestejs/examples/blob/main/CounterWidget/index.js)
```js
import { createWidget } from 'lesta'
const root = document.querySelector('#root')
const widget = createWidget({
   … // component options
}, root)
// await widget.destroy() 
```
`createWidget` - легковесная функция, с помощью которой можно создать только один компонент (виджет).

Для этой функции доступны только возможности описанные в разделе `Basic options` и
`Lifecycle Hooks`.

Функция принимает два аргумента:

- `options` опции из раздела Basic options
- `root` DOM узел для встраивания виджета

`createWidget` возвращает объект с асинхронной функцией `destroy`. `widget.destroy()` уничтожит виджет.

## createApp
```js
import component from './app'
import { createApp } from 'lesta'

const app = createApp({
    root: document.querySelector('#root'),
    directives: {},
    plugins: {},
})
app.mount(component)
```
[ToDoList/index.js](https://github.com/lestejs/examples/blob/main/ToDoList/index.js)

- `options`:
	- `root` DOM узел для монтирования.
	- `directives` глобальные директивы.
	- `plugins` шина данных, все свойства этого объекта будут доступны в store и во всех компонентах через `this`. Подходит для интеграции сторонних библиотек и модулей.

Возвращаемое значение: объект с двумя функциями `mount` и `unmount`.

- `mount(component)` Необходимо передать объект компонента для монтирования.

- `unmount()` Асинхронная функция, демонтирует компонент.

# Basic options

## template
```js
export default {
  template: `<h1>Hello!</h1>`
} 
```
HTML код компонента задается в  `template`. HTML код можно вынести в отдельный файл, импортировать в файле компонента и передать в качестве значения в поле template. В template используется только html код.

## nodes
```js
template: `<button class="next">+</button>`,
proxies: { count: 0 },
nodes() {
    return {
        next: {
            textContent: ({ offsetHeight, nodepath }) => `height: ${ offsetHeight }, node: ${ nodepath }`
            onclick: () => this.proxy.count++,
            disabled: () => this.proxy.count === 5
        }
…
```
```js
// nodes
profile: {
  href: () => this.router.link({ name: 'profile', params: this.method.locale() }),
  style: () => {
    return {
      visibility: this.proxy.auth ? 'visible' : 'hidden'
    }
  }
}
```
Node является DOM узлом, поэтому в нем доступны все соответствующие методы и свойства (такие `textContent` или `innerHTML`, `onclick`, `style` и т.д.). Функция `nodes` должна возвращать объект. Ключами объекта являются имена узлов. Имена узлов должны соответствовать именам классов, заданных в тегах html. Для изменения селектора используется `selectors` (см. ниже).

Чтобы устанавливать значения для нативных свойств узла, достаточно в параметрах узла указать свойство в качестве ключа объекта, в качестве значения указать либо значение, либо функцию, которая будет возвращать значение.

Такие свойства html элемента как `style` или `dataset` являются объектами. Чтобы изменить параметры таких свойств, в эти свойства необходимо передать функции, которые возвращают объект с параметрами для изменения.

Функция свойства узла, может принимать в качестве аргумента сам объект узла. Также в этом объект есть два дополнительных свойства:
- `nodepath` Путь, состоящий из имен узлов.
- `nodename` Имя узла компонента.

Используя деструктуризацию, можно из объекта узла получить нужные свойства.

Получить объект узла в компоненте можно через `this.node[‘имя узла’]`.

## selectors
```js
template: `
    <div class="wrapper">
      <div class="popup"></div>
      <main></main>
    </div>`,
  selectors: { main: 'main' },
  nodes() {
    return {
      popup: {},
      main: {}
   }
}
```
Если в качестве имени узла селектор класса не подходит, то в объекте selectors для узлов можно указать любые другие селекторы в виде строкового значения.

В этом случае `main` это селектор тега.

## directives

[ToDoList/index.js](https://github.com/lestejs/examples/blob/main/ToDoList/index.js)
```js
const app = createApp({
root: document.querySelector('#root'),
directives: {
        _class: {
            update: (node, options, key) => {
                const value = typeof options[key] === 'function' ? options[key]() : options[key]
                value ? node.classList.add(key) : node.classList.remove(key)
         }
     }
}})
```
```js
// nodes 
    filter: {
      _class: {
        gray: () => this.proxy.showIncomplete
      },
      onclick: () => {
        this.proxy.showIncomplete = !this.proxy.showIncomplete
      }
```
```js
directives: {
    _intersection: {
      create: (node, options, directive) => {
        const observer = new IntersectionObserver(options.callback, {
          rootMargin: options.rootMargin || "0px",
          threshold: options.threshold || 1.0
        })
        observer.observe(node)
      }
    }
```
```js
// nodes
     more: {
        _intersection: {
          callback: (entries) => entries.at(0).isIntersecting && this.method.more()
        }
      }
```
Директивы позволяют расширить функционал node. Свойства директив могут иметь параметры или функции. Если в функции встречается `proxy`, то эта функция становится реактивной.

Директивы описываются в объекте `directives`, название директивы должно начинаться с `_`. Так как директивы применяются к узлам, такое обозначение позволяет отличать директивы от нативных методов и свойств `node`.

Для создания директив предоставляются две callback функции:

- `create` Выполняется при обработки узла компонента, и работает с двумя аргументами `node` и `options`. `node` - это объект узла, к которому применяется директива. `options` - это параметры директивы, применяемой к данному узлу.

- `update` Выполняется для каждого параметра директивы. Помимо двух аргументов `node` и `options` предоставляется дополнительный параметр `key`, в который передается ключ параметра. Если параметр является реактивной функцией, то update будет вызываться при каждом реактивном изменении этого параметра.

- `destroy` Выполняется когда компонент размонтируется.


> Обработка событий в директивах должна устанавливаться только через функцию слушателя `node.addEventListener`, так как в свойствах обработки событий `node` уже могут быть установлены функции.

> Директивы можно задавать, как в компоненте, так и глобально для всего приложения

Встроенные директивы _html, _evalHTML,  _text, _class
_html, _evalHTML,  _text являются асинхронными


## params
```js
params: {
  delay: 2000
}
...
console.log(this.param.delay)
```
В params можно описать данные, которые не требующие отслеживания изменений. Тип данных может быть любой. Доступны в компоненте через this.param[‘имя’]

> Свойства params не доступны для расширения. Нельзя в this.param добавить новые свойства. Существующие свойства расширяются без ограничений.

## proxies
```js
proxies: {
  notifications: [],
}
…
methods: {
  async add(notify) {
    this.proxy.notifications.unshift(notify)
    await delay(this.param.delay)
    this.proxy.notifications.pop()
  }
}

```
В proxies описываются данные изменение которых необходимо отслеживать. Для каждого такого объекта будет создан объект-обертка proxy. Он доступен в компоненте через this.proxy. Изменение таких объектов можно отслеживать и использовать для реактивности.

> Все описанные в свойствах узла (node) функции, становятся реактивными, если в них присутствуют proxy. Эти функции будут вызываться при каждом изменении proxy обновляя свойства узла.

> Если вам не нужна реактивность для свойства, тогда значение нужно установить напрямуй без использования стрелочной функции: textContent: this.proxy.foo

> Свойства proxies не доступны для расширения. Нельзя добавить новый proxy после их создания. При этом существующие изменяются без ограничений.

> Все объекты устанавливаемые в прокси, проходят сериализацию. Таким образом происходит клонирование объектов. И при попытке сравнить такой объект с исходным, условие будет возвращать false.

> Свойства объектов которые имеют значение undefined при сериализации будут потеряны. Чтобы этого не произошло, в качестве значения рекомендуется устанавливать null.

## methods
```js
methods: {
  async add(notify) {
    …
  }
}
...
console.log(this.method.add)
```
```js
// parent component
this.node.notifications.method.add({ text: task.name })
```
Функции описанные в methods доступны в компоненте как this.method['имя'].

> Все методы дочернего компонента доступны в родительском через
this.node['имя узла'].method['название метода'].

## handlers
```js
proxies: {
  showIncomplete: false
},
handlers: {
  showIncomplete(v) {
    this.method.delayFilterStop()
  }
}
…
filter: {
  onclick: () => this.proxy.showIncomplete = !this.proxy.showIncomplete
},
add: {
  onclick: () =>  this.proxy.showIncomplete = false
}
```
В `handlers` описываются функции, которые срабатывают после изменения `proxy` объектов. В аргумент функций можно получать новое значение `proxy`. Имя функции задается в формате `keypath`.

`keypath` соответствует ключам вложенных свойств объекта `proxy` разделенными знаком “.”. Если `proxy` не имеет вложенности, то имя функции будет соответствовать имени `proxy`. Для `proxy` имеющих свойства, можно отслеживать изменение конкретных свойств. Например, если в `proxy` объекте `user: { age: 25 }` необходимо отследить изменение свойства `age`, то имя функции должно быть `user.age()`. Так как в `javascript` массив также является объектом, можно отслеживать изменение его элементов или длины: `array.0`, `array.0.name`, `array.length`, где `array` имя массива.

`handlers` особенно полезны, когда `proxy` меняется из нескольких мест и нужно выполнять одно и то же действие при его изменении.

## setters
```js
setters: {
    _card.completed(v) {
      if (typeof v !== 'boolean') {
        this.error('property completed in card: wrong data type')
        return false
      }
      return v
    }
  }
```
Функции описанные в setters перехватывают значение proxy до того как значение пройдет сериализацию и будет установлено. Для того чтобы значение было установлено функции setters должны возвращать значение. Имя функции должно соответствовать **keypath**, аналогично handlers.

> Если функция не вернет значение, то соответствующий proxy изменен не будет. Таким образом в setters можно контролировать реактивность и валидировать значения.

## sources

```js
sources: {
    count: () => import('../count')
  },
  nodes() {
    return {
      count: {
        component: {
          src: this.source.count,
          induce: () => this.proxy.showCompleted
        }
      }
```
Для динамической загрузки модулей необходимо описать функции импорта sources. Далее к ним можно будет обратиться через this.source[‘имя модуля’]. Поле src в component автоматически выполняет функцию динамического импорта. В сочетании с induce или integrate можно реализовать динамической загрузку компонентов (см ниже)

# Component
```js
import card from '../card/index.js'
nodes() {
    return {
      cards: {
        component: {
          src: card,
          params: {
            index: …
          },
          proxies: {
            _card: () => …
          },
          methods: {
            complete: () => …
          }
          // iterate: () => ...,
          // induce: () => ...,
          // sections: {}
        }
      }
    }
  }
```


Чтобы монтировать компонент в узел необходимо задать параметры для свойства component:

- `src` Ожидает объект компонента, который можно предварительно импортировать или определить в текущем файле.
- `params` Передаются данные, не требующие отслеживания. Чтобы в `params` передать ссылку на node, необходимо использовать стрелочную функцию.
- `proxies` Передаются proxy-объекты, которые будут обновляться на стороне дочернего компонента, работает только от родителя к потомку. Чтобы переданные прокси объекты были реактивными, в свойство необходимо передать функцию, в которой присутствует proxy объект. Такая функция может возвращать вычисляемые значения. Для глубинной синхронизации proxy, необходимо, чтобы имя свойства начиналось со символа “_”. Функция не должны иметь вычислений, она должны возвращать только proxy.
- `methods` Передаются функции или методы компонента, который можно будет вызывать на стороне дочернего компонента. `methods` должны быть свойством со стрелочной функцией, это сделано для того, чтобы можно было установить другой метод в качестве значения.

Все перечисленные выше свойства передаются в `props` дочернего компонента. Если в свойство объекта `proxies` не будет предана функция, изменять этот proxy из родительского компонента станет невозможно. Но на стороне дочернего компонента будет создан proxy с переданным значением.

В одном узле можно создать только один компонент.

Аналогично как и функции нативных свойств узлов, описанных выше, функции `params`, `proxies` в качестве аргумента принимают объект узла со всеми дополнительными свойствами. Исключения составляют итерируемые компоненты, подробная информация в разделе `iterate`.

## props
```js
props: {
    params: {
      time: {
        type: 'number',
        default: 2000,
        readonly: true
      }
    }
  }
```

```js
props: {
    params: {
	// …
    },
    proxies: {
      notify: {
        type: 'object',
        required: true,
        // default: ...
        // store: ''
      }
    },
    methods: {
      close: {
        required: true
      }
    }
  }
```
Чтобы получить данные от родительского компонента, необходимо указать их в объекте props: `params`, `proxies`, `methods`. В `methods` мы можем получить коллбэк функции от родительского компонента, которые можно использовать для коммуникации компонентов от потомка к родителю.

Дополнительные опции `props`:

`params`

- `required` [`boolean`] Проверка, чтобы значение не было пустым.
- `type` [`string`] Проверка на соответствие типу.
проверка осуществляется по стандарным javaScript типам и одному дополнительному не нативному типу array.
- `default` Задает значение по умолчанию.
- `store` [`string`] Имя store из которого необходимо принять `prop`.
- `readonly` [`boolean`] Делает значение неизменяемым.

> В `params` сериализация не применяется к следующим типам `Promise`, `HTMLCollection`, `NodeList`, `Element`. Таким образом в дочерний компонент можно передавать узлы или `HTML` элементы.

> Если в компонент необходимо передать ссылку на объект, необходимо чтобы имя параметра начиналось с “__” (двойное подчеркивание). Такие параметра не проходят сериализацию.

`proxies`

- `required` [`boolean`] Проверка, чтобы значение не было пустым.
- `type` [`string`] Проверка на соответствие типу. проверка осуществляется по стандарным javaScript типам и одному дополнительному не нативному типу array.
- `default` Задает значение по умолчанию.
- `store` [`string`] Имя store из которого необходимо принять `prop`.

Переданный в компонент `proxy` обязательно должен быть принят в `props`.
Валидация значений происходит не только при создании компонента, но и в runtime (не работает для `proxy`, имя которых начинается с '_').

`methods`

- `required` [`boolean`] Проверка, чтобы значение не было пустым.

> Все объекты передаваемые между компонентами проходят стерилизацию. Таким образом создаются копии этих объектов. Если передаются объекты, то значения свойств должны быть таких типов, которые поддерживает `JSON`.

> Все полученные `props` становятся частью компонента и доступны как `this.param`, `this.proxy`, `this.method`.

> Даже если `props` приняты, но не переданы, то на стороне дочернего компонента, они все равно будут созданы в `this.param`, `this.proxy`. Поэтому `param` и `proxy` можно создавать указав их в `props`.

## induce
```js
proxies: {
    showCompleted: false,
  },
  nodes() {
    return {
      count: {
        component: {
          src: this.source.count,
          induce: () => this.proxy.showCompleted // mount component if condition true
        }
      },
      filter: {
        onclick: () =>  this.proxy.showCompleted = !this.proxy.showCompleted
      }
```
Иногда необходимо, чтобы компоненты создавались только по какому-то условию. Условие необходимо указать в функции induce.

Изменение proxy объекта в функции induce, при условии **true** заново монтирует компонент, предварительно вызывая unmount.

> Не работает для компонентов со свойством iterate.

## iterate
```js
nodes() {
    return {
      notifyList: {
        component: {
          src: notify,
          iterate: () => this.proxy.notifications,
          params: {
            index: (notify, index) => index
          },
          proxies: {
            notify: (notify) => notify
          },
          methods: {
            close: (index) => this.proxy.notifications.splice(index, 1)
          }
        }
      }
    }
  }
```
Для создания компонентов в цикле в свойство iterate необходимо передать либо массив, либо функцию которая возвращает proxy объект массива.

Если функция возвращает proxy объект массива, то изменения этого объекта будут удалять или создавать новые компоненты или обновлять функции proxies передаваемые в props итерируемых компонентов.
Функции передаваемы в props компонентов, предоставляют два аргумента. В первый приходит текущий обрабатываемый элемент в массиве. Во второй индекс текущего обрабатываемого элемента в массиве.

> Для proxy объекта массива, в свойство iterate обязательно нужно передать функцию.

> Свойства induce не работает вместе со свойства iterate. Чтобы скрыть компонент, можно просто очистить массив, переданный в свойство iterate.

> Template интегрируемого компонента обязательно должен иметь один корневой тег.


## sections
```js
// dialog component
 template: `
  <dialog class="dialog">
    <div class="close"></div>
    <div section="content"></div>
  </dialog>`

// parent component
popup: {
  component: {
    src: dialog,
    sections: {
      content: {
	// src: …			
	// params: {},
	// proxies: {},
	// methods: {}
      }
    }
  }
}
```
`sections` позволяет интегрировать из родительского компонента в дочерний другие компоненты, в заранее выбранные теги (секции). В template дочернего компонента необходимо задать теги с атрибутами section, значениями этих атрибутов будут именами секций. В родительском компоненте в component properties нужно описать объект sections. Свойства этого объекта будут соответствующие имена секций. Значениями этих свойств могут быть component properties: src, params, proxies, methods. Если component properties отсутствуют, то компонент можно будет интегрировать динамически. Коммуникация с такими компонентами доступна только из родителя.

Свойство узла секции **nodepath**, помимо пути узла в конце имеет имя секции.

## integrate
```js
this.node.popup.integrate('content', {
	src: form,
	params: {
	  card: task,
	},
	methods: {
	  save: () => { }
	}
  })
```
Функции integrate позволяет динамически интегрировать компонент в секцию. Для этого в функцию необходимо передать два параметра.

this.node['имя узла'].integrate(‘имя секции’, component properties)

Повторный вызов integrate, заново монтирует компонент, предварительно вызывая unmount.

Получить доступ к узлам секций в родительском компоненте можно с помощью
this.node['имя узла'].section['имя секции']

this.node['имя узла'].section['имя секции'].method['название метода']

Демонтировать компонент из секции можно с помощью
this.node['имя узла'].section['имя секции'].unmount()

## unmount
```js
await this.node.mycomponent.unmount()
```
Эта функции демонтирует компонент из узла. this.node['имя узла'].unmount()

## refresh
```js
await this.node.mycomponent.refresh()
```
С помощью этой функции можно обновить компонент в узле. Функция предварительно выполняет unmount, затем компонент заново монтируется с исходными параметрами.

# Lifecycle Hooks
На разном этапе жизни компонента доступны следующие функции:

- `loaded`
Вызывается после загрузки файла компонента, но до обработки всех опции. На этом этапе все опции компонента будут доступны в this.options. В this.container находится ссылка на родительский узел. Во всех компонентах в this.root доступен корневой узел.

- `created`
Вызывается перед обработкой nodes, но после обработки всех props, params, proxies, methods.

- `mounted`
Вызывается после обработки всех узлов и дочерних компонентов.

- `unmounted`
Вызывается после удаления HTML содержимого родительского узла компонента.

- `updated`
Работает только в корневом компоненте, когда нужно сообщить приложения об изменении каких либо данных. Принимает один аргумент типа объект, который можно использовать для деструктуризации.

app.update({ foo })

# Functions

## reactivity
```js
console.log(this.node.foo.reactivity)
```
В объекте this.node[‘nodename’].reactivity описаны все реактивные функции используемые в node. Если необходимо отменить реактивность для узла, достаточно удалить соответствующую функцию в этом объекте.

## deleteReactive
```js
this.deleteReactive(reactivity, keypath)
```

## execute
```js
textContent: () => this.execute(() => this.proxy.foo)
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
// !!
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
