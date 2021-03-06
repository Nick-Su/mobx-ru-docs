# Оптимизации React {🚀}

{% hint style="info" %}
Оригинал этой страницы доступен [здесь](https://mobx.js.org/react-optimizations.html)
{% endhint %}

### Оптимизация рендеринга компонентов React

MobX очень быстрый, [зачатую даже быстрее, чем Redux](https://twitter.com/mweststrate/status/718444275239882753), но вот несколько советов, которые помогут вам максимально эффективно использовать React и MobX. В основном, большинство из них относятся к React, а не к MobX. Имейте в виду, что знать об этих шаблонах полезно, но обычно ваше приложение будет достаточно быстрым, даже если вы совсем не будете уделять внимание оптимизации.

Вопрос оптимизации становится приоритетным только тогда, когда это является настоящей проблемой!

### Используйте больше мелких компонентов

_`observer`_ компоненты будут отслеживать все используемые ими значения и запускать ре-рендер компонента при изменении этих значений. Таким образом, чем меньше ваши компоненты, тем меньше изменений они должны отрендерить. Это значит, что вы получаете больше независимых друг от друга частей пользовательского интерфейса.

### Списки должны рендериться в отдельных компонентах

Вышесказанное особенно актуально при рендеринге больших коллекций. React, как известно, плохо справляется с рендерингом больших коллекций, поскольку React должен перерасчитывать созданные коллекцией компоненты, при каждом изменении коллекции. Поэтому рекомендуется иметь компоненты, которые просто отображают коллекцию и рендерят ее и не рендерят ничего другого.

Плохой подход:

```
const MyComponent = observer(({ todos, user }) => (
    <div>
        {user.name}
        <ul>
            {todos.map(todo => (
                <TodoView todo={todo} key={todo.id} />
            ))}
        </ul>
    </div>
))
```

В приведенном выше примере, React будет делать избыточный перерассчет всех компонентов _`TodoView`_ при изменении _`user.name`_. Они не будут повторно отрендерены, но процесс перерасчета компонент затратен сам по себе.

Хороший подход:

```
const MyComponent = observer(({ todos, user }) => (
    <div>
        {user.name}
        <TodosView todos={todos} />
    </div>
))

const TodosView = observer(({ todos }) => (
    <ul>
        {todos.map(todo => (
            <TodoView todo={todo} key={todo.id} />
        ))}
    </ul>
))
```

### Не используйте индексы массива в качестве ключей

Не используйте в качестве ключа индексы массива или любое значение, которое может измениться в будущем. При необходимости сгенерируйте идентификаторы для ваших объектов. Прочтите[ это сообщение в блоге](https://robinpokorny.medium.com/index-as-a-key-is-an-anti-pattern-e0349aece318).

### Разыменовывайте значения как можно позже

При использовании _`mobx-react`_ рекомендуется разыменовать значения как можно позже. Это связано с тем, что MobX будет повторно рендерить компоненты, которые автоматически разыменовывают наблюдаемые значения. Если это будет происходить глубже в вашем дереве компонентов, то потребуется меньше компонентов для повторного рендеринга.

Медленнее:

```
<DisplayName name={person.name} />
```

Быстрее:

```
<DisplayName person={person} />
```

В более быстром примере изменение свойства _`name`_ запускает повторный рендер только компонента _`DisplayName`_. В более медленном примере, не только компонент _`DisplayName`_, но и его родительский компонент также будет повторно рендерться при изменении свойства _`name`_. В этом нет ничего плохого и если рендеринг компонента-родителя выполняется достаточно быстро (обычно это так!), то этот подход работает как надо.

### Функциональные свойства {🚀}

Вы можете заметить, что для более позднего разыменования значений нужно создать множество небольших компонентов-наблюдателей, каждый из которых будет настроен для отображения другой части данных, например:

```
const PersonNameDisplayer = observer(({ person }) => <DisplayName name={person.name} />)

const CarNameDisplayer = observer(({ car }) => <DisplayName name={car.model} />)

const ManufacturerNameDisplayer = observer(({ car }) => 
    <DisplayName name={car.manufacturer.name} />
)
```

Но это быстро становится утомительным, если у вас много данных из разных источников. Альтернативой является использование функции, возвращающей данные, которые вы хотите, чтобы рендерил ваш \*Displayer:

```
const GenericNameDisplayer = observer(({ getName }) => <DisplayName name={getName()} />)
```

Затем вы можете использовать компонент следующим образом:

```
const MyComponent = ({ person, car }) => (
    <>
        <GenericNameDisplayer getName={() => person.name} />
        <GenericNameDisplayer getName={() => car.model} />
        <GenericNameDisplayer getName={() => car.manufacturer.name} />
    </>
)
```

Такой подход позволит повторно использовать _`GenericNameDisplayer`_ во всем приложении для рендеринга любого имени, при этом вы по-прежнему будете сводить повторный рендеринг компонента к минимуму.
