# Интеграция с React

{% hint style="info" %}
Оригинал этой страницы доступен [здесь](https://mobx.js.org/react-integration.html)
{% endhint %}

Использование:

```
import { observer } from "mobx-react-lite" // или "mobx-react".

const MyComponent = observer(props => ReactElement)
```

Хотя MobX работает независимо от React, чаще всего они используются вместе. В разделе «[Суть MobX](../vvedenie/sut-mobx.md)» вы уже видели самую важную часть этой интеграции: HoC _`observer`_, которым  "оборачивается" компонент React.

_`observer`_ предоставляется отдельным пакетом привязок React, который вы выбираете во время установки. В этом примере мы собираемся использовать более легкий [пакет _`mobx-react-lite`_](https://github.com/mobxjs/mobx/tree/main/packages/mobx-react-lite).

```
import React from "react"
import ReactDOM from "react-dom"
import { makeAutoObservable } from "mobx"
import { observer } from "mobx-react-lite"

class Timer {
    secondsPassed = 0

    constructor() {
        makeAutoObservable(this)
    }

    increaseTimer() {
        this.secondsPassed += 1
    }
}

const myTimer = new Timer()

// Функциональный компонент обернутый в `observer` будет реагировать
// на любое будущее изменение наблюдаемого объекта, используемое ранее.
const TimerView = observer(({ timer }) => <span>Seconds passed: {timer.secondsPassed}</span>)

ReactDOM.render(<TimerView timer={myTimer} />, document.body)

setInterval(() => {
    myTimer.increaseTimer()
}, 1000)
```

**Совет**: вы можете поиграть с примером выше в [CodeSandbox](https://codesandbox.io/s/minimal-observer-p9ti4?file=/src/index.tsx).

HoC _`observer`_ автоматически подписывает компоненты React на любые _наблюдаемые объекты_, используемые во время рендеринга. В результате компоненты будут автоматически повторно рендериться при изменении соответствующих наблюдаемых объектов. Это также гарантирует, что компоненты не будут повторно рендериться, если нет соответствующих изменений. Таким образом, наблюдаемые объекты, доступные для компонента, но фактически не считываемые им, никогда не вызовут повторного рендеринга.

На практике, это делает приложения MobX очень хорошо оптимизированными уже из коробки, и им обычно не требуется дополнительный код для предотвращения чрезмерного рендеринга.

Для работы наблюдателя (_`observer`_) не имеет значения, _как_ наблюдаемые объекты достигают компонента, важно только то, что к ним есть обращение и они считываются. Глубокое чтение наблюдаемых объектов - это нормально и сложные выражения вроде _`todos[0].author.displayName`_ работают из коробки. Это делает механизм подписки намного более точным и эффективным по сравнению с другими фреймворками, в которых зависимости данных должны быть объявлены явно или предварительно вычислены (например, селекторы).

### Локальное и внешнее состояние

MobX предоставляет большую гибкость в организации состояния, поскольку для него не имеет значения (технически), какие наблюдаемые объекты мы читаем или откуда наблюдаемые объекты возникли. Приведенные ниже примеры демонстрируют различные шаблоны того, как внешнее и локальное наблюдаемое состояние может использоваться в компонентах, обернутых в _`observer`_.

### Использование внешнего состояния в _`observer`_ компонентах

{% tabs %}
{% tab title="используя props" %}
Наблюдаемые объекты могут быть переданы в компоненты как свойства (как в примере выше):

```
import { observer } from "mobx-react-lite"

const myTimer = new Timer() // см. объявление Timer выше.

const TimerView = observer(({ timer }) => <span>Seconds passed: {timer.secondsPassed}</span>)

// Передаем myTimer как пропс.
ReactDOM.render(<TimerView timer={myTimer} />, document.body)
```
{% endtab %}

{% tab title="используя глобальные переменные" %}
```
/*
Поскольку не имеет значения, как мы получили ссылку на наблюдаемый объект, 
мы можем напрямую использовать наблюдаемые объекты из внешних областей 
(в том числе из импорта и т.д.):
*/

const myTimer = new Timer() // см. объявление Timer выше.

// Пропсов нет, `myTimer` используется непосредственно из замыкания.
const TimerView = observer(() => <span>Seconds passed: {myTimer.secondsPassed}</span>)

ReactDOM.render(<TimerView />, document.body)

/*
Прямое использование наблюдаемых объектов работает очень хорошо, 
но поскольку это обычно вводит состояние модуля, 
этот шаблон может усложнить модульное тестирование. 
Вместо этого мы рекомендуем использовать React Context.
*/
```
{% endtab %}

{% tab title="используя React context" %}
[React Context](https://reactjs.org/docs/context.html) - отличный механизм для обмена наблюдаемыми объектами со всем поддеревом:

```
import {observer} from 'mobx-react-lite'
import {createContext, useContext} from "react"

const TimerContext = createContext<Timer>()

const TimerView = observer(() => {
    // Берем таймер из контекста
    const timer = useContext(TimerContext) // см. объявление Timer выше.
    return (
        <span>Seconds passed: {timer.secondsPassed}</span>
    )
})

ReactDOM.render(
    <TimerContext.Provider value={new Timer()}>
        <TimerView />
    </TimerContext.Provider>,
    document.body
)
```

Обратите внимание, что мы не рекомендуем заменять значение _`value`_ _`Provider`_'a другим. При использовании MobX в этом не должно возникать необходимости, поскольку наблюдаемый объект, используемый совместно, может быть обновлен самостоятельно.
{% endtab %}
{% endtabs %}

### Использование локального наблюдаемого состояния в _`observer`_ компонентах

Поскольку наблюдаемые объекты, используемые наблюдателем, могут поступать откуда угодно, то они также могут быть локальным состоянием. Опять же, доступны разные варианты.

{% tabs %}
{% tab title="`useState` с наблюдаемым классом" %}
Самый простой способ использовать локальное наблюдаемое состояние - сохранить ссылку на наблюдаемый класс с помощью _`useState`_. Обратите внимание: поскольку мы не хотим заменять ссылку, мы полностью игнорируем функцию обновления состояния, возвращаемую _`useState`_:Как указывалось ранее, вместо использования классов можно напрямую создавать наблюдаемые объекты. Для этого мы можем использовать наблюдаемое:

```
import { observer } from "mobx-react-lite"
import { useState } from "react"

const TimerView = observer(() => {
    const [timer] = useState(() => new Timer()) //см. объявление Timer выше.
    return <span>Seconds passed: {timer.secondsPassed}</span>
})

ReactDOM.render(<TimerView />, document.body)

/*
Если вы хотите автоматически обновлять таймер, 
как мы это делали в исходном примере, 
useEffect можно использовать обычным для React способом:
*/

useEffect(() => {
    const handle = setInterval(() => {
        timer.increaseTimer()
    }, 1000)
    return () => {
        clearInterval(handle)
    }
}, [timer])
```
{% endtab %}

{% tab title="`useState` с локальным наблюдаемым объектом" %}
Как указывалось ранее, вместо использования классов можно напрямую создавать наблюдаемые объекты. Для этого мы можем использовать _`observable`_:

```
import { observer } from "mobx-react-lite"
import { observable } from "mobx"
import { useState } from "react"

const TimerView = observer(() => {
    const [timer] = useState(() =>
        observable({
            secondsPassed: 0,
            increaseTimer() {
                this.secondsPassed++
            }
        })
    )
    return <span>Seconds passed: {timer.secondsPassed}</span>
})

ReactDOM.render(<TimerView />, document.body)
```
{% endtab %}

{% tab title="хук `useLocalObservable`" %}
Комбинация _`const [store] = useState(() => observable ({/ * наблюдаемое что-то * /}))`_ довольно распространена. Чтобы упростить этот шаблон, в пакете _`mobx-react-lite`_ предоставляется хук _`useLocalObservable`_, позволяющий упростить предыдущий пример до:

```
import { observer, useLocalObservable } from "mobx-react-lite"
import { useState } from "react"

const TimerView = observer(() => {
    const timer = useLocalObservable(() => ({
        secondsPassed: 0,
        increaseTimer() {
            this.secondsPassed++
        }
    }))
    return <span>Seconds passed: {timer.secondsPassed}</span>
})

ReactDOM.render(<TimerView />, document.body)
```
{% endtab %}
{% endtabs %}

#### Возможно, вам не понадобится локальное наблюдаемое состояние

Обычно мы не рекомендуем сразу использовать _`observable`_ для локального состояния компонента, поскольку теоретически это может заблокировать вас от некоторых функций механизма [React Suspense](https://reactjs.org/docs/concurrent-mode-suspense.html). Используйте _`observable`_, когда состояние захватывает данные домена, совместно используемые компонентами (включая дочерние). Например, задачи, пользователи, заказы и т. д.

Состояние, которое фиксирует только состояние пользовательского интерфейса (состояние загрузки, выбор и т. д.), может быть лучше обслужено [хуком _`useState`_](https://reactjs.org/docs/hooks-state.html), поскольку это позволит вам использовать функции React Suspense в будущем.

Использование наблюдаемых объектов внутри компонентов React становится ценнее, если они 1) глубокие; 2) имеют вычисляемые значения или 3) используются совместно с другими _`observer`_ компонентами.

### Всегда считывайте наблюдаемые объекты внутри _`observer`_ компонетах

Вам может быть интересно, когда же применять _`observer`_? Практическое правило: применяйте _`observer`_ ко всем компонентам, которые читают наблюдаемые данные.

_`observer`_ усиливает только украшаемый вами компонент, но не вызываемые им компоненты. Так что обычно, все ваши компоненты должны быть обернуты в  _`observer`_. Не волнуйтесь, это не снижает эффективность. Напротив, большее количество компонентов наблюдателя делает рендеринг более эффективным, поскольку обновления становятся более детализированными.

#### Совет: извлекайте значения из объектов как можно позже

_`observer`_ работает лучше всего, если вы передаете ссылки на объекты как можно дольше и читаете их свойства только внутри компонентов на основе _`observer`_'а, которые собираются отображать их в DOM / низкоуровневых компонентах. Другими словами, _`observer`_ реагирует на то, что вы «разыменовываете» значение объекта.

В приведенном выше примере компонент _`TimerView`_ не реагировал бы на будущие изменения, если бы он был определен следующим образом, потому что _`.secondsPassed`_ читается не внутри компонента-наблюдателя, а снаружи и, следовательно, не отслеживается:

```
const TimerView = observer(({ secondsPassed }) => <span>Seconds passed: {secondsPassed}</span>)

React.render(<TimerView secondsPassed={myTimer.secondsPassed} />, document.body)
```

Обратите внимание, что это отличается от других библиотек, таких как _`react-redux`_, где хорошей практикой является раннее разыменование и передача примитивов для лучшего использования мемоизации. Если проблема не совсем ясна, обязательно ознакомьтесь с разделом «[Разбираем реактивность](../sovety-i-rekomendacii/razbiraem-reaktivnost.md)».

#### Не передавайте наблюдаемые в компоненты в не _`observer`_ компоненты.

Компоненты, обернутые в _`observer`_, подписываются только на наблюдаемые объекты, используемые во время их собственного рендеринга компонента. Поэтому, если наблюдаемые объекты / массивы / коллекции map передаются дочерним компонентам, они также должны быть обернуты в _`observer`_. Это также верно для любых компонентов, основанных на callback.

Если вы хотите передать наблюдаемые объекты компоненту, который не является _`observer`_'ом (либо потому, что это сторонний компонент, либо потому, что вы хотите, чтобы этот компонент был агностиком MobX), то вам придется преобразовать наблюдаемые объекты в обычные структуры или значения JavaScript, прежде чем передать их дальше.

Чтобы подробнее рассказать о вышеизложенном, возьмем следующий пример наблюдаемого объекта _`todo`_, компонента _`TodoView`_ (наблюдателя) и воображаемого компонента _`GridRow`_, принимающим пару столбец/значение, но не являющегося _`observer`_'ом:

```
class Todo {
    title = "test"
    done = true

    constructor() {
        makeAutoObservable(this)
    }
}

const TodoView = observer(({ todo }: { todo: Todo }) =>
   // НЕПРАВИЛЬНО: GridRow не отобразит изменения в todo.title / todo.done
   // поскольку он не является наблюдателем (observer).
   return <GridRow data={todo} />

   // ПРАВИЛЬНО: позволяем `TodoView` обнаружить соответствующие изменения в `todo`,
   // и передать обычные JS данные ниже.
   return <GridRow data={{
       title: todo.title,
       done: todo.done
   }} />

   // ПРАВИЛЬНО: использование `toJS` тоже работает, но лучше все указывать явно.
   return <GridRow data={toJS(todo)} />
)
```

#### Callback-компоненты могут потребовать \<Observer>

Представьте тот же пример, где _`GridRow`_ вместо этого принимает callback _`onRender`_. Поскольку _`onRender`_ является частью цикла рендеринга _`GridRow`_, а не рендеринга _`TodoView`_ (хотя он синтаксически появляется именно там), мы должны убедиться, что callback-компонент использует _`observer`_-компонент. Или мы можем создать встроенного анонимного наблюдателя, используя [`<Observer />`](https://github.com/mobxjs/mobx-react#observer)

```
const TodoView = observer(({ todo }: { todo: Todo }) => {
    // НЕПРАВИЛЬНО: GridRow.onRender не подхватит изменения в todo.title/todo.done
    // поскольку он не является наблюдателем (observer).
    return <GridRow onRender={() => <td>{todo.title}</td>} />

    // ПРАВИЛЬНО: оберните callback-рендеринг в Observer, чтобы иметь возможность обнаруживать изменения.
    return <GridRow onRender={() => <Observer>{() => <td>{todo.title}</td>}</Observer>} />
})
```

### Советы

<details>

<summary>Рендеринг на стороне сервера (SSR)</summary>

Если _`observer`_ используется в контексте рендеринга на стороне сервера, то обязательно вызовите метод _`enableStaticRendering(true)`_, чтобы _`observer`_ не подписывался на какие-либо используемые наблюдаемые объекты, и чтобы не возникало проблем со сборщиком мусора.

</details>

<details>

<summary>Примечание: mobx-react или mobx-react-lite?</summary>

По умолчанию, в этой документации мы использовали _`mobx-react-lite`_. Пакет _`mobx-react`_ - это старший брат, который использует _`mobx-react-lite`_ под капотом. Он предлагает еще несколько функций, которые обычно более не нужны в новых проектах. Дополнительные возможности _`mobx-react`_:

1. Подддержка классовых компонентов React.
2. _`Provider`_ и _`Inject`_. Собственный предшественник MobX React.createContext больше не нужен.
3. Специфические наблюдаемые _`propTypes`_.

</details>

<details>

<summary>Примечание: <em><code>observer</code></em> или <em><code>React.memo</code></em>?</summary>

_`observer`_ автоматически применяет _`memo`_, поэтому нет необходимости оборачивать _`observer`_-компоненты в _`memo`_. _`memo`_ можно безопасно применять к компонентам наблюдателя, т.к. мутации (глубоко) внутри пропса в любом случае будут улавливаться наблюдателем, если это необходимо.

</details>

<details>

<summary>Совет: <em><code>observer</code></em> для классовых компонентов React</summary>

Как указано выше, компоненты на основе классов поддерживаются только через пакет _`mobx-react`_, а не через _`mobx-react-lite`_. Вкратце, вы можете обернуть классовые компоненты в _`observer`_ так же, как это ранее делалось с функциональными компонентами:

```
import React from "React"

const TimerView = observer(
    class TimerView extends React.Component {
        render() {
            const { timer } = this.props
            return <span>Seconds passed: {timer.secondsPassed} </span>
        }
    }
)
```

Ознакомьтесь с документацией по [mobx-react ](https://github.com/mobxjs/mobx-react#api-documentation)для получения дополнительной информации.

</details>

<details>

<summary>Совет: красивые названия компонентов в React DevTools</summary>

[React DevTools](https://reactjs.org/blog/2019/08/15/new-react-devtools.html) использует информацию об отображаемых именах компонентов для правильного отображения иерархии компонентов.

Если вы используете:

```
export const MyComponent = observer(props => <div>hi</div>)
```

то тогда в DevTools не будет доступно отображаемое имя.

&#x20;![](https://mobx.js.org/assets/devtools-noDisplayName.png)

Следующие подходы могут это исправить:

* используйте _`function`_ вместо стрелочной функции. _`mobx-react`_ выводит имя компонента из имени функции:

```
export const MyComponent = observer(function MyComponent(props) {
    return <div>hi</div>
})
```

* Транспиляторы (например, Babel или TypeScript) выводят имя компонента из имени переменной:

```
const _MyComponent = props => <div>hi</div>
export const MyComponent = observer(_MyComponent)
```

* Снова сделайте вывод из имени переменной, используя экспорт по умолчанию:

```
const MyComponent = props => <div>hi</div>
export default observer(MyComponent)
```

* \[**Не работает**] Явно укажите _`displayName`_:

```
export const MyComponent = observer(props => <div>hi</div>)
MyComponent.displayName = "MyComponent"
```

Это не работает в React 16 на момент написания раздела. _`observer`_ из пакета mobx-react использует React.memo и сталкивается с этой ошибкой: [https://github.com/facebook/react/issues/18026](https://github.com/facebook/react/issues/18026), но она будет исправлена в React 17.

Теперь вы можете видеть имена компонентов:

![](https://mobx.js.org/assets/devtools-withDisplayName.png)

</details>

<details>

<summary>{🚀} <strong>Совет:</strong> при объединении <em><code>observer</code></em>'а с другими компонентами более высокого порядка, сначала примените <em><code>observer</code></em></summary>

Когда _`observer`_ должен быть объединен с другими декораторами или компонентами более высокого порядка, убедитесь, что _`observer`_ является самым внутренним (применяемым первым) декоратором; в противном случае он может не работать вообще.

</details>

<details>

<summary>{🚀} <strong>Совет: получение вычисляемых значений из пропсов</strong></summary>

В некоторых случаях, вычисляемые значения ваших локальных наблюдаемых объектов могут зависеть от некоторых пропсов, которые получает ваш компонент. Однако набор пропсов, получаемых компонентом React, сам по себе не является наблюдаемым, поэтому изменения в пропсах не будут отражены ни в каких вычисляемых значениях. Вы должны вручную обновить локальное наблюдаемое состояние, чтобы правильно получить вычисленные значения из последних данных.

```
import { observer, useLocalObservable } from "mobx-react-lite"
import { useEffect } from "react"

const TimerView = observer(({ offset = 0 }) => {
    const timer = useLocalObservable(() => ({
        offset, // Изначальноез значение offset
        secondsPassed: 0,
        increaseTimer() {
            this.secondsPassed++
        },
        get offsetTime() {
            return this.secondsPassed - this.offset // Not 'offset' from 'props'!
        }
    }))

    useEffect(() => {
        // Синхронизация значения offset из пропсов в наблюдаемом 'timer'
        timer.offset = offset
    }, [offset])

    // Эффект для установки таймера, только для демонстрационных целей.
    useEffect(() => {
        const handle = setInterval(timer.increaseTimer, 1000)
        return () => {
            clearInterval(handle)
        }
    }, [])

    return <span>Seconds passed: {timer.offsetTime}</span>
})

ReactDOM.render(<TimerView />, document.body)
```

На практике вам редко понадобится этот шаблон, так как return \<span>Seconds passed: {timer.secondsPassed - offset}\</span> - гораздо более простое, хотя и немного менее эффективное решение.

</details>

<details>

<summary>{🚀} <strong>Совет: useEffect и observables</strong></summary>

_`useEffect`_ можно использовать для настройки побочных эффектов, которые должны произойти и которые привязаны к жизненному циклу компонента React. Использование _`useEffect`_ требует указания зависимостей. Но в MobX это не нужно, поскольку в MobX уже есть способ автоматического определения зависимостей эффекта - _`autorun`_. К счастью, объединить _`autorun`_ и привязать его к жизненному циклу компонента с помощью _`useEffect`_ просто:

```
import { observer, useLocalObservable, useAsObservableSource } from "mobx-react-lite"
import { useState } from "react"

const TimerView = observer(() => {
    const timer = useLocalObservable(() => ({
        secondsPassed: 0,
        increaseTimer() {
            this.secondsPassed++
        }
    }))

    // Эффект, который срабатывает при изменениях наблюдаемых значений
    useEffect(
        () =>
            autorun(() => {
                if (timer.secondsPassed > 60) alert("Still there. It's a minute already?!!")
            }),
        []
    )

    // Эффект для установки таймера, только для демонстрационных целей.
    useEffect(() => {
        const handle = setInterval(timer.increaseTimer, 1000)
        return () => {
            clearInterval(handle)
        }
    }, [])

    return <span>Seconds passed: {timer.secondsPassed}</span>
})

ReactDOM.render(<TimerView />, document.body)
```

Обратите внимание, что мы возвращаем средство удаления, созданное _`autorun`_'ом, из нашей функции-эффекта. Это важно, поскольку гарантирует, что автозапуск будет очищен после размонтирования компонента!

Массив зависимостей обычно можно оставить пустым, если только ненаблюдаемое значение не должно вызывать повторный запуск _`autorun`_. Только в этом случае вам нужно будет добавить его в массив зависимостей. Чтобы ваш линтер был доволен, вы можете определить _`timer`_ (в приведенном выше примере) как зависимость. Это безопасно и не имеет никакого дальнейшего эффекта, поскольку ссылка на самом деле никогда не изменится.

Если вы предпочитаете явно указать, какие наблюдаемые объекты должны запускать эффект, используйте _`reaction`_ вместо _`autorun`_ (кроме того, шаблон остается идентичным).

</details>

#### Как я могу еще оптимизировать компоненты React?

Для дополнительной информации ознакомьтесь с разделом ["Оптимизация React" {🚀}](optimizacii-react.md).

### Исправление проблем

Помогите! Мой компонент не перерисовывается...

1. Убедитесь, что вы не забыли про _`observer`_ (да, это самая частая ошибка).
2. Убедитесь, что объект, на который вы собираетесь реагировать, действительно наблюдаем. При необходимости используйте такие утилиты, как _`isObservable`_, _`isObservableProp`_, если необходимо проверить это во время выполнения.
3. Проверьте журналы консоли в браузерах на наличие предупреждений или ошибок.
4. Обязательно ознакомьтесь с принципами работы трекинга в целом. Ознакомьтесь с разделом «[Разбираем реактивность](../sovety-i-rekomendacii/razbiraem-reaktivnost.md)».
5. Ознакомьтесь с общими ошибками, описанными вышеу.
6. [Настройте](../tonkaya-nastroika/konfigurirovanie.md) MobX, чтобы он предупреждал вас о ненадлежащем использовании механизмов и проверял журналы консоли.
7. Используйте [трассировку](../sovety-i-rekomendacii/analiziruem-reaktivnost.md), чтобы убедиться, что вы подписываетесь на нужные вещи. Также проверьте, что вообще делает MobX с помощью пакета [spy](../sovety-i-rekomendacii/analiziruem-reaktivnost.md) / [mobx-logger](https://github.com/winterbe/mobx-logger).
