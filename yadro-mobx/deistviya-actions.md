# Действия (actions)

{% hint style="info" %}
Оригинал этой страницы доступен [здесь](https://mobx.js.org/actions.html)
{% endhint %}

### Обновляем состояние через действия

Использование:

* _`action`_
* _`action(fn)`_
* _`action(name, fn)`_

Все приложения имеют действия. Действие - это любая часть кода, изменяющая состояние. Как правило, действия всегда происходят в ответ на событие. Например, была нажата кнопка, что-то ввели в поле ввода, пришло сообщение из websocket и т. д.

MobX требует, чтобы вы объявили свои действия, хотя использование [makeAutoObservable](sozdanie-nablyudaemogo-sostoyaniya-observable-state.md) может автоматизировать большую часть работы. Действия помогают вам лучше структурировать код и обеспечивают следующие преимущества в производительности:

1. Они выполняются внутри [транзакций](https://mobx.js.org/api.html#transaction). Никакие реакции не будут выполняться, пока не завершится самое внешнее действие, что гарантирует, что промежуточные или неполные значения, созданные во время действия, не будут видны остальной части приложения, пока действие не будет завершено.
2. По умолчанию, изменять состояние вне действий запрещено. Это помогает четко определить, где происходят обновления состояния в вашей кодовой базе.

Тэг _`action`_ следует использовать только в функциях _изменяющих_ состояние. Функции, получающие информацию (выполняющие поиск или фильтрацию данных), не должны быть помечены как действия. Это позволит MobX отслеживать их вызовы. Методы помеченые как _`action`_ будут неперечислимыми.

### Примеры

{% tabs %}
{% tab title="makeObservable" %}
```
import { makeObservable, observable, action } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeObservable(this, {
            value: observable,
            increment: action
        })
    }

    increment() {
        // Промежуточные состояния не будут видны наблюдателям
        this.value++
        this.value++
    }
}

```


{% endtab %}

{% tab title="makeAutoObservable" %}
```
import { makeAutoObservable } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeAutoObservable(this)
    }

    increment() {
        this.value++
        this.value++
    }
}
```
{% endtab %}

{% tab title="action.bound" %}
```
import { makeObservable, observable, action } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeObservable(this, {
            value: observable,
            increment: action.bound
        })
    }

    increment() {
        this.value++
        this.value++
    }
}

const doubler = new Doubler()

// Вызов инкремента таким способом безопасен, так как он уже привязан.
setInterval(doubler.increment, 1000)
```
{% endtab %}

{% tab title="action(fn)" %}
```
import { observable, action } from "mobx"

const state = observable({ value: 0 })

const increment = action(state => {
    state.value++
    state.value++
})

increment(state)
```
{% endtab %}

{% tab title="runInAction(fn)" %}
```
import { observable, runInAction } from "mobx"

const state = observable({ value: 0 })

runInAction(() => {
    state.value++
    state.value++
})
```
{% endtab %}
{% endtabs %}

### Оборачиваем функции с помощью _`action`_

Чтобы максимально использовать транзакционный характер MobX, действия следует передавать как можно дальше вовне. Хорошо пометить метод класса как действие, если он изменяет состояние. Еще лучше пометить обработчики событий как действия, поскольку учитывается самая внешняя транзакция. Один немаркированный обработчик событий, который впоследствии вызывает два действия, по-прежнему генерирует две транзакции.

Чтобы помочь создать обработчики событий на основе действий, _`action`_ - это не только маркер, но и функция более высокого порядка. _`action`_ можно вызвать с функцией в качестве аргумента, и в этом случае он вернет функцию, обернутую действием с той же сигнатурой.

Например, в React обработчик _`onClick`_ можно обернуть так, как показано ниже.

```
const ResetButton = ({ formState }) => (
    <button
        onClick={action(e => {
            formState.resetPendingUploads()
            formState.resetValues()
            e.preventDefault()
        })}
    >
        Reset form
    </button>
)
```

В целях отладки мы рекомендуем либо дать название обернутой функции, либо передать имя в качестве первого аргумента действия.

<details>

<summary>Примечание: действия не отслеживаются</summary>

Еще одна особенность действий - они не отслеживаются. Когда действие вызывается изнутри побочного эффекта или вычисляемого значения (что очень редко!), наблюдаемые значения, считываемые действием, не будут учитываться в производных зависимостях.

_`makeAutoObservable`_, _`extendObservable`_ и _`observable`_ используют особую разновидность действия, называемую _`autoAction`_, которая определяет во время выполнения, является ли функция производной или действием.

</details>

### `action.bound`

Использование:

* _`action.bound`_ (annotation)

Аннотацию _`action.bound`_ можно использовать для автоматической привязки метода к правильному экземпляру, чтобы _`this`_  всегда был правильно привязан внутри функции.

<details>

<summary>Cовет: используйте <em><code>makeAutoObservable(o, {}, { autoBind: true })</code></em> чтобы автоматически связать действия и поток</summary>

```
import { makeAutoObservable } from "mobx"

class Doubler {
    value = 0

    constructor(value) {
        makeAutoObservable(this, {}, { autoBind: true })
    }

    increment() {
        this.value++
        this.value++
    }

    *flow() {
        const response = yield fetch("http://example.com/value")
        this.value = yield response.json()
    }
}
```

</details>

### _`runInAction`_

Использование:

* `runInAction(fn)`

Используйте эту утилиту, чтобы создать временное действие, вызываемое немедленно. Это может быть полезно в асинхронных процессах. Взгляните на [приведенный выше блок кода](deistviya-actions.md#primery) для примера.

### Действия и наследование

Только действия, **определенные в прототипе**, могут быть **переопределены** подклассом:

```
class Parent {
    // в экземпляре
    arrowAction = () => {}

    // в прототипе
    action() {}
    boundAction() {}

    constructor() {
        makeObservable(this, {
            arrowAction: action
            action: action,
            boundAction: action.bound,
        })
    }
}
class Child extends Parent {
    // Выбросит ошибку: TypeError: Cannot redefine property: arrowAction
    arrowAction = () => {}

    // OK
    action() {}
    boundAction() {}

    constructor() {
        super()
        makeObservable(this, {
            arrowAction: override,
            action: override,
            boundAction: override,
        })
    }
}
```

Чтобы **привязать** одиночное действие к _`this`_, вместо стрелочных функций можно использовать _`action.bound`_. Подробнее в разделе [подклассы](../sovety-i-rekomendacii/subclassing.md).

### Асинхронные действия

По сути, асинхронные процессы не нуждаются в особой обработке в MobX, поскольку все реакции обновляются автоматически, независимо от момента времени, когда они были вызваны. И поскольку наблюдаемые объекты изменчивы, как правило, безопасно хранить ссылки на них на время действия. Однако каждый шаг, который обновляет наблюдаемые значения в асинхронном процессе, должен быть помечен как _`action`_. Этого можно достичь несколькими способами, используя указанные выше API, как показано в примере ниже.

Например, при обработке промисов обработчики, обновляющие состояние, должны быть заключены в оболочку _`action`_ или быть действием, как показано ниже.

{% tabs %}
{% tab title="Оборачивание обработчиков в 'action'" %}
```
Обработчики разрешения промисов обрабатываются в потоке, 
но запускаются после завершения исходного действия, 
поэтому их нужно обернуть в action:

import { action, makeAutoObservable } from "mobx"

class Store {
    githubProjects = []
    state = "pending" // "pending", "done" or "error"

    constructor() {
        makeAutoObservable(this)
    }

    fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        fetchGithubProjectsSomehow().then(
            action("fetchSuccess", projects => {
                const filteredProjects = somePreprocessing(projects)
                this.githubProjects = filteredProjects
                this.state = "done"
            }),
            action("fetchError", error => {
                this.state = "error"
            })
        )
    }
}
```
{% endtab %}

{% tab title="Обработка обновлений в отдельных действиях" %}
```
Если обработчики промисов являются полями классов, то они будут 
автоматически обёрнуты в action makeAutoObservable'ом

import { makeAutoObservable } from "mobx"

class Store {
    githubProjects = []
    state = "pending" // "pending", "done" or "error"

    constructor() {
        makeAutoObservable(this)
    }

    fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        fetchGithubProjectsSomehow().then(this.projectsFetchSuccess, this.projectsFetchFailure)
    }

    projectsFetchSuccess = projects => {
        const filteredProjects = somePreprocessing(projects)
        this.githubProjects = filteredProjects
        this.state = "done"
    }

    projectsFetchFailure = error => {
        this.state = "error"
    }
}
```
{% endtab %}

{% tab title="async/await + runInAction" %}


```
Любые шаги после await не находятся в одном такте, 
поэтому они требуют обёртки действий. 
Здесь мы можем использовать runInAction:

import { runInAction, makeAutoObservable } from "mobx"

class Store {
    githubProjects = []
    state = "pending" // "pending", "done" or "error"

    constructor() {
        makeAutoObservable(this)
    }

    async fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        try {
            const projects = await fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            runInAction(() => {
                this.githubProjects = filteredProjects
                this.state = "done"
            })
        } catch (e) {
            runInAction(() => {
                this.state = "error"
            })
        }
    }
}
```
{% endtab %}

{% tab title="`flow` + функция-генератор" %}
```
import { flow, makeAutoObservable, flowResult } from "mobx"

class Store {
    githubProjects = []
    state = "pending"

    constructor() {
        makeAutoObservable(this, {
            fetchProjects: flow
        })
    }

    // Заметьте! звездочка (*) означает функцию-генератор!
    *fetchProjects() {
        this.githubProjects = []
        this.state = "pending"
        try {
            // Yield вместо await.
            const projects = yield fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            this.state = "done"
            this.githubProjects = filteredProjects
        } catch (error) {
            this.state = "error"
        }
    }
}

const store = new Store()
const projects = await flowResult(store.fetchProjects())
```
{% endtab %}
{% endtabs %}

### Использование flow вместо async / await {🚀}

Использование:

* `flow` _(annotation)_
* `flow(function* (args) { })`

_`flow`_ обёртка - это дополнительная альтернатива async / await, упрощающая работу с действиями MobX. Поток принимает функцию-генератор в качестве единственного аргумента. Внутри генератора вы можете сделать цепочку промисов, выталкивая их (вместо await somePromise вы пишете yield somePromise). Затем механизм потока будет следить за тем, чтобы генератор либо продолжал работу, либо срабатывал при разрешении выполненного промиса.

Таким образом, _`flow`_ - это альтернатива _`async`_ / _`await`_, не требующая дополнительных действий. Применять _`flow`_ можно следующим образом:

1. Обернуть асинхронные функции во _`flow`_
2. Вместо _`async`_ использовать _`function*`_
3. Вместо _`await`_ использовать _`yield`_

Приведенный выше пример flow + функция-генератор показывает, как это будет выглядеть на практике.

Обратите внимание, что функция flowResult необходима только при использовании TypeScript. Поскольку у метода есть декоратор _`flow`_, возвращаемый генератор будет заключен в промис. Однако TypeScript не знает об этом преобразовании, поэтому flowResult будет следить за тем, чтобы TypeScript знал об этом изменении типа.

_`makeAutoObservable`_  автоматически сделает генераторы потоками. Аннотированные элементы потока будут неперечисляемыми.

<details>

<summary>{🚀} Примечание: использование потока на объектных полях</summary>

_`flow`_ как и _`action`_ может использовать для непосредственного оборачивания функций. Приведенный выше пример также мог быть записан следующим образом:

```
import { flow } from "mobx"

class Store {
    githubProjects = []
    state = "pending"

    fetchProjects = flow(function* (this: Store) {
        this.githubProjects = []
        this.state = "pending"
        try {
            // yield instead of await.
            const projects = yield fetchGithubProjectsSomehow()
            const filteredProjects = somePreprocessing(projects)
            this.state = "done"
            this.githubProjects = filteredProjects
        } catch (error) {
            this.state = "error"
        }
    })
}

const store = new Store()
const projects = await store.fetchProjects()
```

Плюсом является то, что нам больше не нужен flowResult, но недостатком является то, что _`this`_ необходимо строго типизировать, чтобы убедиться, что его тип выводится правильно.

</details>

### _`flow.bound`_

Использование:

* `flow.bound` _(annotation)_

Аннотация _`flow.bound`_ может быть использована для автоматического связывания метода с соответсвующей сущностью так, что this будет всегда правильно привязан внутри функции. Подобно действиям, потоки могут быть привязаны по умолчанию с помощью опции _`autoBind`_.

### Отмена потоков  {🚀}

Еще одно приятное преимущество потоков - их отменяемость. Возвращаемое значение потока - это промис, который разрешается со значением, возвращаемым функцией-генератором в конце. У возвращенного промиса есть дополнительный метод _`cancel()`_, который прерывает работающий генератор и отменяет его. Любые конструкции _`try`_ / _`finally`_ все равно будут выполняться.

### Отключение обязательных действий {🚀}

По умолчанию, MobX версии 6 и выше требует, чтобы вы использовали действия для изменения состояния. Однако вы можете настроить MobX так, чтобы это поведение было отключено. Ознакомьтесь с разделом [_`enforceActions`_](../tonkaya-nastroika/konfigurirovanie.md). Например, это может быть весьма полезно при настройке модульного теста, когда предупреждения не всегда имеют большое значение.
