# Установка

{% hint style="info" %}
Оригинал этой страницы доступен [здесь](https://mobx.js.org/installation.html)
{% endhint %}

MobX работает в любой ES5 среде, включая браузеры и NodeJS.

Есть два пакета: _`mobx-react-lite`_ и _`mobx-react`_. _Mobx-react-lite_ поддерживает только функциональные компоненты. Mobx-react кроме функциональных компонентов, также поддерживает классовые компоненты. Выбирайте необходимый пакет исходя из ваших потребностей соответствующей командой _Yarn_ или _NPM:_

_**Yarn**: `yarn add mobx`_

_**NPM**: `npm install --save mobx`_

_**CDN**:_ [https://cdnjs.com/libraries/mobx](https://cdnjs.com/libraries/mobx) / [https://unpkg.com/mobx/dist/mobx.umd.production.min.js](https://unpkg.com/mobx/dist/mobx.umd.production.min.js)

### Используйте совместимую со спецификацией транспиляцию для свойств класса

{% hint style="danger" %}
Предупреждение: Когда вы используете MobX с TypeScript или Babel, и если вы планируете использовать классы, то обязательно обновите свою конфигурацию, чтобы использовать транспиляцию, совместимую со спецификацией TC-39. Без этого, поля класса нельзя сделать наблюдаемыми до их инициализации.
{% endhint %}

* **TypeScript**: установите параметр компилятора _`"useDefineForClassFields": true`_
* **Babel**: убедитесь, что вы используете по крайней мере версию 7.12, со следующей конфигурацией:

```
{
    "plugins": [["@babel/plugin-proposal-class-properties", { "loose": false }]],
    // Babel >= 7.13.0 (https://babeljs.io/docs/en/assumptions)
    "assumptions": {
        "setPublicClassFields": false
    }
}
```

Для проверки вставьте этот кусок кода в начало ваших файлов (например, в index.js)

```javascript
if (!new class { x }().hasOwnProperty('x')) throw new Error('Транспилятор настроен некорректно');
```

### MobX в старых средах JavaScript <a href="#mobx-on-older-javascript-environments" id="mobx-on-older-javascript-environments"></a>

По умолчанию MobX использует прокси для оптимальной производительности и совместимости. Однако в старых движках JavaScript прокси недоступен (проверьте поддержку прокси). Примеры этого: Internet Explorer (до Edge), Node.js <6, iOS <10, Android до RN 0.59 или Android на iOS.

В таких случаях MobX может вернуться к реализации, совместимой с ES5, которая работает почти идентично, хотя и есть несколько ограничений из-за отсутствия поддержки прокси. Вам нужно будет явно включить резервный режим, настроив useProxies:

```
import { configure } from "mobx"

configure({ useProxies: "never" }) // Или "ifavailable".
```

### MobX и Декораторы

Если вы раньше использовали MobX или следовали онлайн-руководствам, вы, вероятно, видели _MobX с декораторами_, такими как _`@observable`_. **В MobX 6 мы решили отказаться от декораторов.** По умолчанию, декораторы отключены для максимальной совместимости со стандартным JavaScript. Их все еще можно использовать, [если вы их включите](../tonkaya-nastroika/vklyuchenie-dekoratorov.md).а

### Development vs production

Если вы не используете предварительно собранный дистрибутив, заканчивающийся на .\[production|development].min.js, Mobx использует переменную process.env.NODE\_ENV для определения среды. Убедитесь, что при релизе проекта он установлен на «production». Обычно это делает ваш любимый сборщик: [webpack](https://reactjs.org/docs/optimizing-performance.html#webpack) [Rollup](https://reactjs.org/docs/optimizing-performance.html#rollup) [Browserify](https://reactjs.org/docs/optimizing-performance.html#browserify) [Brunch](https://reactjs.org/docs/optimizing-performance.html#brunch)

Большинство проверок безопасности, таких как _`enforceAction`_ и подобные, происходят только в режиме _development_.

### Использование MobX на других фреймворках / платформах

* [MobX.dart](https://mobx.netlify.app): MobX для Flutter / Dart
* [lit-mobx](https://github.com/adobe/lit-mobx): MobX для lit-element
* [mobx-angular](https://github.com/mobxjs/mobx-angular): MobX для angular
* [mobx-vue](https://github.com/mobxjs/mobx-vue): MobX для Vue
