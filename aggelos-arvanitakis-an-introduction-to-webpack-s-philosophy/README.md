# Введение в философию Webpack

*Перевод статьи [Aggelos Arvanitakis](https://www.facebook.com/aggelos.arvanitakis): [An introduction to webpack’s philosophy](https://medium.com/webmonkeys/an-introduction-to-webpacks-philosophy-78a02461c17f).*

Много было сказано о Webpack, но я постоянно сталкиваюсь с людьми, не знающими не только о его существовании, но также о его фактическом предназначении. В этой статье я попытаюсь объяснить вам почему Webpack существует, почему он нужен и какие преимущества вы можете получить от его внедрения в ваш проект.

## Введение

В последние несколько лет, в индустрии web-разработки наблюдается постоянный рост одностраничных приложений (SPA), много автономных приложений, которые можно установить на компьютер, предлагающих полноценный функционал для пользователя. К сожалению, как известно, чем больше функций вы добавляете, тем больше кода приходится писать и, учитывая среднюю сложность SPA, в конце концов вы достигните точки, где ваш код может состоять из сотен, если не тысяч `.js`, `.css` и многих других файлов.

Поэтому возникает следующая проблема: как вам справиться с кодовой базой такого размера? Как управлять порядком загрузки файлов/модулей, делающих ваше приложение работоспособным, когда у вас их так много? В старые времена это было значительно легче. Вы бы вставили теги `script` в определенном порядке в тело вашего html. Так, например, если в файл `B` должен быть загружен после файла `A` и файл `C` должен быть загружен после `A` и `B`, вероятно, вы делали бы что-то подобное:

```html
<html>
    <body>
        …
        …
        <script type='text/javascript' src='path/to/A.js'></script>
        <script type='text/javascript' src='path/to/B.js'></script>
        <script type='text/javascript' src='path/to/C.js'></script>
    </body>
</html>
```

По мере усложнения приложения, количество этих файлов будет расти и управление зависимостями будет постоянно усложняться. Что если ваш файл `B` нужно заргузить после другого файла `D`, который, в свою очередь, должен быть загружен после другого файла `E`, и файл `A` также должен быть после загрузки файла `D`? Вы, наверное, уже немного запутались, а у нас до сих пор только 5 файлов.

Попробуйте представить отношения зависимостей между этими файлами. Эта конструкция называется **граф зависимостей**, по сути, это дерево, описывающее порядок загрузки файлов.

Как вы уже поняли, ведение этого графика вручную - трудная задача. Когда вы создаете новый файл, вам нужно знать, где должна происходить загрузка в цепочке загрузки скриптов, а также от каких внешних файлов он зависит, и в конечном итоге, где должен находиться `<script ...> </ script>` в `.html`.

Кроме того, хорошие практики говорят нам, что мы должны создать единственный `.js` файл (или как можно меньше) из всех этих отдельно взятых `A.js`, `B.js`, `C.js` и так далее, чтобы избежать затрат на несколько HTTP запросов загрузки для каждого `.js` файла. Этот процесс называется **сборка**, он объеденяет разные файлы в один, сохраняя порядок их зависимостей. Делая это вручную, требуется много копировать и вставлять, каждый раз когда вы создаете или меняете файл, одновременно рискуя допустить ошибку.

## Что такое Webpack и зачем он мне нужен

Webpack - это **инструмент**, основная цель которого состоит в том, чтобы собрать все ваши `.js` файлы в любое нужное вам количество пакетов (*bundles*), а также убедиться, что в собранном пакете есть все `.js` файлы вашего проекта в правильном порядке.

## Как это работает и как я могу этим воспользоваться

Для того, чтобы использовать Webpack есть одно большое требование: ваш **JavaScript код должен быть организован в формате модуля**. Это означает, что каждый файл должен явно требовать все остальные `.js` файлы, от которых он зависит, и потенциально сделать некоторые элементы доступными для других `.js` файлов. С олдскульными техниками нет способа декларировать зависимости, кроме как сказать: «Эй, я сделаю так, чтобы тэг `script` для `A.js` будет загружен перед тэгом `script` `B.js`».

В современном мире существует 3 различных способа сделать это:

1. CommonJS
2. AMD modules
3. ES6 modules

Независимо от того, создаете ли вы проект с нуля или у вас уже есть текущий проект, вам придется структурировать его таким образом, чтобы соответствовать одному из вышеперечисленных. На данный момент я должен упомянуть, что с выпуском ES6 (или ES2015 или «новой версии JavaScript») шаблоны СommonJS и AMD считаются в основном устаревшими, поэтому, если у вас нет веской причины выбрать любой из них, я бы настоятельно рекомендовал бы вам придерживаться модулей ES6 при создании нового проекта, так как именно в эту сторону двигается рынок.

Итак, как мы говорили ранее, каждый файл должен явно описывать то, что ему нужно в качестве входных данных, и то, что он сделает доступным для других файлов. Все, что не сделано доступным для других файлов, остается в пределах файла и никогда не будет раскрыто. В псевдоязыке это будет выглядеть примерно так:

```js
// Убедиться что функция add из файла add.js была загружена
// Убедиться что переменные "number1" и "number2" из файла "numbers.js" были загружены
// variable X = add(number1, number2)
// Сделать X доступной для других JS файлов
```

Итак, как бы вы требовали другие файлы? Ну, прежде всего, вам нужно будет отказаться от всех ваших тегов `<script ...> </ script>` в `.html` файлах. После этого вам нужно будет сообщить всем вашим `.js` файлам о приоритете их загрузки, который переводится как «Эй, вы должны быть загружены, когда другой файл загрузится первым». В мире модулей требования файлов преобразуются в требования переменных, функций, компонентов, объектов и так далее. И это напоминает что-то близкое к следующему:

В ES6:

```js
// add.js
export default function add (a,b) {
    return a + b
};

// numbers.js
export const number1 = 3;
export const number2 = 5;

// whatever.js
import add from './path/to/add.js';
import { number1, number2 } from './path/to/numbers.js';

const X = add(number1, number2);

export const X;
```

В CommonJS
```js
// add.js
module.exports = {
    add: function(a,b) {
        return a + b
    }
}

// numbers.js
module.exports = {
    number1: 3,
    number2: 5
};

// whatever.js
var math = require('./path/to/add.js');
var numbers = require('./path/to/numbers.js')

module.exports = {
    X: math.add(numbers.number1, numbers.number2);
};
```

В AMD:

```js
// add.js
define([] , function () {
   return {
       add: function(a, b) {
          return a + b
   }
});

// numbers.js
define([] , function () {
   return {
       number1: 3,
       number2: 5
});

// whatever.js
define([
   './path/to/add',
   './path/to/numbers'
] , function (math, numbers) {
    var X = math.add(numbers.number1, numbers.number2);
    return X;
});
```

Вышеуказанные инструкции определяют `add.js` и `variables.js` в качестве зависимости для нашего `whatever.js`. Имея это в виду, файлы `.js` имеют зависимости, полностью описывающие, что нужно загружать, вычислять или запускать, прежде чем они сами могут быть загружены или запущены. Webpack теперь принимает эти инструкции и пытается создать направленный ациклический граф (*acyclic graph*), на простом русском языке это означает, что он пытается подтянуть все зависимости без каких-либо циклов. Наличие цикла означало бы, что файл `A` зависит от файла `B`, а файл `B` зависит от файла `A`, что делает связывание невозможным. Если же циклов не обнаружено, Webpack может взять содержимое всех этих файлов и вставить их в один `.js` файл в правильном порядке.

Но как Webpack узнает, какие файлы добавлять в финальный файл (*bundle*), ведь нигде нет тегов `<script>`? Ответ заключается в том, что просто так это узнать невозможно. Вот почему нам нужно объявить файл, который будет работать как точка входа. Webpack описал бы точку входа так: «Направьте меня в файл, чтобы при рекурсивном обходе его требований, все, что может понадобиться вашему проекту, было бы импортировано». В современном случае это будет файл `.js`, содержащий основной/базовый компонент вашего приложения. В более старом сценарии: файл, который должен быть загружен после загрузки всех остальных `js` файлов. В нашем конкретном примере сверху точкой входа будет файл `whatever.js`.

Последний шаг - вам нужно добавить только один тег `<script>`, который будет указывать на собранный `.js` файл, созданный для нас Webpack.

## В заключение

Хотя существуют альтернативы Webpack (такие как Browserify), Webpack в настоящее время является самым популярным инструментом не только из-за большого числа разработчиков, активно поддерживающих его, но и из-за количества дополнительных функциональных возможностей, доступных для него, таких как загрузчики (*loaders*) и плагины. С их помощью вы можете легко запускать различные процессы до и после того, как Webpack собрал ваши файлы. Это позволяет легко построить статический конвейер активов (*assets*), подобный следующему:

1. Переведет ваши файлы из es6 в es5
2. Переведет ваши `.less` или `.scss` файлы в обычный `.css`
3. Минифицирует ваши файлы (уменьшает размер файла)
4. Удаляет все комментарии из всех файлов (для дальнейшего уменьшения размера файла)
5. Добавит все пользовательские шрифты и иконки в выходной файл (чтобы они были доступны сразу, без задержки, связанной с дополнительными HTTP запросами)
6. Разместит весь ваш код в один `js` файл, а все библиотеки в другие `js` файлы (чтобы их можно было кэшировать в браузере, так как вряд ли они когда-нибудь изменятся в стабильном проекте).
7. Соберет все ваши `.css` сборки (*bundles*) в один файл.
8. Автоматически изменит версию выходного файла (чтобы избежать проблем с кэшированием браузера при изменении содержимого).
9. Автоматически вставит файлы как теги `<link>` или `<script>` в ваш `.html` файл (вам не нужно вручную менять теги каждый раз, когда изменяются версии файлов).

Спасибо за чтение! :)

- - - -

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*