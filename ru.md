Самое главное в правильно сделанных модуля в том, что они работают. Они 
не падают с ошибками. Они не ведут себя странно когда вы или кто-то еще 
используете их в другом окружении. Они не будут вести себя странно даже если 
кто-то когда-нибудь решит обновить ваш код. Остальные проблемы вторичны. 
К примеру, не важно сколько килобайт места на диске вы сэкономили, или 
насколько просто использовать ваш модуль.

Я использую понятие «модуль» в самом общем значении. Концепция модульности 
применима как к структуре проекта, или менеджеру пакетов, так и к различным 
способам написания программного кода. В статье я попробую объяснить идею 
модульности на реальных примерах.


## [Глобальные зависимости][1]

Классические глобальные системы подразумевают, что модули, от которых 
зависимосят всех ваши проекты, хранятся в одном месте:

    | - modules
    | --- bear extends MODULES/animal
    | --- animal
    | - apps
    | --- grizzly extends MODULES/bear
    | --- koala extends MODULES/bear
    | --- panda extends MODULES/bear

По-началу, такая стуктура может показаться идеальной. Во-первых, все ваши модули
хранятся в одной директории. Во-вторых, нет дублирования кода: с одной стороны, 
это экономия места на диске, с другой, обновление любого модуля моменатально 
попадет во все проекты, которые этот модуль используют.

Из-за этих очевидных плюсов такой подход используют очень часто. Но такой 
подход **потрясающе порочен**.


### ТЕХНИЧЕСКИЕ ПРОБЛЕМЫ

Обновление модулей происходит часто. Но об обратной совместимости обновлений или
о хорошем версионировании нельзя говорить с уверенностью. Не будет ничего 
удивительного, если ваше приложение сломается после очередного обновления 
модулей. Глобальные зависимости подразумевают, что вы обновляете **все** свои 
приложения каждый раз, когда обновляется код любого глобального модуля. 

Если у вас много проектов, то обновлять их все при каждом обновлении 
зависимостей — это ужасно. Более того, порой это физически невозможно.


### СОЦИАЛЬНЫЕ ПРОБЛЕМЫ

Проектами основанными на глобальных зависимостях сложно делиться. Зависимости 
относятся не к самому проекту, а хранятся за его пределами, в окружении. 
Обычно проблема с воссозданием такого окружения решается с помощью перечисления 
действий, в readme, **or provide a switching 
mechanism to recreate the environment in order to run the project**.

Есть еще одна проблема. Из-за высокой связанности проектов, разработчикам 
приходится поддерживать большое количество проектов одновременно. Особенно 
актуально это в мире открытого ПО.


### ПРИМЕР

В течение года, вы собираетесь писать сайты для большого количества клиетов. 
Для каждого нового проекта вы создаете отдельную директорию. И в каждой такой
директории вы храните одинаковый скрипт с утилитами. В конце концов, вы 
решаетесь вынести этот скрипт из каждого проекта в отдельную директорию:

    // /Users/dude/scripts/utils.js
    var utils = module.exports = {};
    utils.slug = function(str) {
      return str.toLowerCase().replace(/ /g, '-').replace(/[^\w-]+/g, '');
    };


А затем, вы подключаете его в каждом вашем проекте:

    // /Users/dude/projects/acme/blog.js
    var utils = require('/Users/dude/scripts/utils.js');
    var title = utils.slug('Acme Blog Post'); // acme-blog-post


Ваше дело выгорело — проектов с каждым днем становится все больше. Однажды,
один из ваших клиентов, написал заголовок в своем блоге: «Blogs - How do they 
work?». Увидев получившуюся ссылку `blogs---how-do-they-work`, которую 
сгенерировала ваша функция `utils.slug`, клиент пожалуется вам. И вы исправите
вашу функцию:

    utils.slug = function(str) {
      return str.toLowerCase().replace(/[^\w ]+/g, '').replace(/ +/g, '-');
    };

Теперь результат выглядит более прилично: `blogs-how-do-they-work`.

Чтож, теперь сломан каждый проект, который использовал старый формат функции 
`utils.slug`. Так что, кроме обновления `utils.slug`, вы так же должны обновить
все свои проекты. Кроме того, после обновления, некоторые ссылки на страницы 
ваших клиентов сломаются. Неприятно, не правда ли?




Я уверен, что вы породумаете механизм обратной совместимости для этой 
крошечной функции. Но нужно быть очень осторожным. Изменяя код любой глобальной
зависимости, нужно помнить о каждом проекте, который эту зависимость использует.
Это совершенно лишний и тяжелый для разработчиков. И это практически невозможно
для команд.

**Глобальные зависимости приводят к постоянно ломающемуся коду.**

Скрипт с утилитами стоило вынести в отдельный модуль, и установить его локально
в каждый проект. Рабочий код намного важнее, чем пара сэкономленных 
килобайт.


## [Плоские зависимости][2]
<!-- ## [Flat Dependencies][2] -->


Плоская структура, или равные зависимости — это шаг в правильную сторону. Каждый
проект имеет свои собственную директорию `modules`, так что каждый проект 
будет обновлен только тогда, когда это будет необходимо. Проекты теперь проще
разворачивать на других машинах, потому что пользователь может быстрее воссоздать
окружение внутри директории проекта и запустить приложение (WAT?). 

<!-- A flat structure or peer dependencies is a step towards the right way. Each
project has its own`modules` location so each project can be updated only as
needed. Each project is also shareable as the user can recreate the environment 
quickly within the project folder to run the app. -->

    | - apps
    | --- grizzly extends modules/bear
    | ----- modules
    | ------- bear extends animal
    | ------- animal
    | --- koala extends modules/bear
    | ----- modules
    | ------- bear extends animal
    | ------- animal
    | --- panda extends modules/bear
    | ----- modules
    | ------- bear extends animal
    | ------- animal


Простая структура зависимостей хорошо работают только для проектов с небольшим 
количеством зависимостей. Такие структуры также легко реализуемы, потому как 
каждый модуль расположен всего лишь на директорию глубже.

<!-- Flat dependency structures only work well for projects with a small amount of
dependencies. They are also easily accessible since every module is located just
one folder down. -->


Такая структура начинает разрушаться если ваше дерево зависимостей становится
разнообразнее. Наш `grizzly` хочет есть, поэтому добавим модуль `fish`:

<!-- This structure begins to break down when the module tree becomes diverse. Our
`grizzly` needs to eat let's give him some `fish` which extends the `animal`
module: -->

    | - grizzly@0.1.0
    | --- modules
    | ----- fish@0.1.0 extends animal@0.1.0
    | ----- bear@0.1.0 extends animal@0.1.0
    | ----- animal@0.1.0


### ТЕХНИЧЕСКИЕ ПРОБЛЕМЫ
### TECHNICAL ISSUES

Все замечательно пока `animal` не обновится до версии `0.2.0`. Разработчик `bear`
быстро обновил свой модуль под версию `animal@0.2.0`. Вы работаете над 
`gruzzly@0.2.0`, который теперь поддерживает `bear@0.2.0`. Но, к сожалению, 
у разработчика `fish` нет времени на обновления.

<!-- Everything is golden until `animal` updates to `0.2.0`. The maintainer of 
`bear` is active and updates to `animal@0.2.0`. You're working on 
`grizzly@0.2.0` which now relies on `bear@0.2.0`. But unfortunately the
maintainer of`fish` doesn't have time to update. -->


Что делать? Оставить `bear@0.1.0` пока не обновится `fish`? У вас есть дэдлайн,
к которому нужно подключить новые фичи из `bear@0.2.0`! Вероятно, в этом случае
вы напишете хак, для того, чтобы заставить `fish` *приемлемо* работать.

<!-- What do you do? Keep `bear@0.1.0` until the maintainer of `fish` gets time to
update? You have a deadline that requires those features in`bear@0.2.0`! Likely
at this point you'll be writing hacks to get`fish` to work *good enough*. -->


### ПРОБЛЕМЫ КОММУНИКАЦИИ(?)
<!-- ### SOCIAL ISSUES -->


Плоские модули могут сломаться если какой-нибудь из модулей неодиданно обновится.
Для того, чтобы избежать потенциальных конфликтов в системе, опирающейся на модули,
приходиться прилогать много усилий.(WAT?) Именно по этому, в небольших сильно связанных 
командах, такой подход *проходит*.

<!-- Flat modules can break when another unanticipated module updates. This puts
pressure on the developer ecosystem to couple their modules together to avoid 
these potential conflicts. Which is why small, tight knit teams with minimal 
dependencies**get by** with this approach. -->


Открытое ПО развивается через многообразие. Я верю, что модульная структура должна
поощрять развязывание(??) модулей. Разработчики не должны задумываться о целой 
экосистеме [ окружении модуля ], они должны просто создать один модуль.


<!-- Open source software progresses through diversity. I believe a module structure
should encourage module decoupling. Developers shouldn't have to think and keep 
up with an entire ecosystem just to build a single module. -->


### ПРИМЕР
<!-- ### EXAMPLE -->


Фреймворки — это замечательный пример модулей, которые создают ситуацию плоской 
зависимости(WAT?).

<!-- Frameworks are a great example of modules that create peer dependent situations -->
<!-- .   -->

Создадим Acme framework: (??? Я не создавал никогда xD)

<!-- Let's create an Acme framework: -->

    var acme = module.exports = {
      config: {
        user: 'Dude'
      },
      announce: function() {
        console.log('Hi! My name is ' + this.config.user);
      },
    };


Сейчас каждый плагин для Acme требует инстанса объекта Acme(WAT???).  Создадим 
плагин:

<!-- Now each Acme plugin requires an instance of the Acme object. Let's create a
plugin: -->


    module.exports = function(acme) {
      if (acme.config.user) acme.announce.call(acme);
      else console.log('User not found');
    };


Плагин не указывает Acme как зависимость, но инстанс Acme необходим для работы
плагина. Это плоская зависимость. 

<!-- The plugin doesn't consume Acme as a dependency but the instance of Acme is
required for the plugin to run. Therefore it is a peer dependency. -->


Эта архитектура выглядит удобно с точки зрения автора. Но она имеет ряд проблем:

<!-- This architecture seems convenient from a plugin author perspective but it has
a couple of problems: -->


*   В будущем выйдет новая версия фреймворка. Каждый проект может иметь только 
    одну версию Acme. Пользователю придется обновить каждый плагин для того, 
    чтобы использовать новую версию фреймворка.

<!-- *   Down the road new versions of the Acme framework are released. Each project
    can only have one version of your Acme framework installed. The user is forced 
    to upgrade every single plugin they use in order to use the new version of your 
    framework. -->


*   Ваш плагин работает только с фреймворком Acme. Вы должни быть хорошим 
    сторонником открытого ПО, и сделать ваш плагин общим(?). Тогда пользователи
    других фреймворков, или пользователи, которые вообще не используют фреймворки
    смогут воспользоваться вашим кодом. Нет нужды переписывать один и тот же код
    снова и снова для каждого нового фреймворка. 

<!-- *   Your plugin will only work with the Acme framework. You should be a good
    open source citizen and make your plugin generic. Then users of other frameworks
    or users who don't use a framework can consume your code. We don't need the same
    code written over and over custom tailored to each framework. -->


**Фреймворки должны поощрять общие плагины.**(WAT???)
<!-- **Frameworks should encourage generic plugins.** -->


Вот пример более общего кода, который не требует инстанса Acme, таким образом, 
избавляется от плоской зависимости. 

<!-- Here is a more generic approach that doesn't require an instance of acme thus
removing the peer dependency: -->

    // framework
    var acme = module.exports = {
      config: {},
      announce: function() {
        console.log('Hi! My name is ' + this.config.user);
      },
    };
    
    // plugin
    var acme = require('acme');
    module.exports = function(config) {
      acme.config = config;
      if (config.user) acme.announce.call(acme);
      else console.log('User not found');
    };


Теперь ваш плагин ориентирован на будущее и достубен для всех пользователей. 
<!-- Now your plugin is future proof and available to everyone. -->


## [Вложенные зависимости][3]
<!-- ## [Nested Dependencies][3] -->


Вложенные зависимости решают проблемы глобальных и плоских систем. Каждый модуль 
в такой системе — это отдельный проект со своими зависимостями. Таким образом 
модули становятся легко переносимы и изолированы(инкапсулированы).

<!-- Nested dependencies solve the issues of global and flat systems. Each module is
its own project. These modules are portable and encapsulated. -->

    | - apps
    | --- grizzly extends modules/bear
    | ----- modules
    | ------- bear extends modules/animal
    | --------- modules
    | ----------- animal
    | --- koala extends modules/bear
    | ----- modules
    | ------- bear extends modules/animal
    | --------- modules
    | ----------- animal
    | --- panda extends modules/bear
    | ----- modules
    | ------- bear extends modules/animal
    | --------- modules
    | ----------- animal


Вложенные зависимости полностью решают проблему версионирования присущую плоским 
системам:

    | - grizzly@0.2.0
    | --- modules
    | ----- fish@0.1.0 extends animal@0.1.0
    | ------- animal@0.1.0
    | ----- bear@0.2.0 extends animal@0.2.0
    | ------- animal@0.2.0

Это безопасный подход. Теперь автор каждого модуля беспокоится исключительно 
о своих собственных зависимостях. Таким образом, это позволяет экосистеме 
развиваться в геометрической прогрессии, быть уверенным в стабильности работы. 

<!-- This a safe approach. Each module author only has to worry about their own
dependencies. Thus allowing the ecosystem to thrive exponentially and operate 
with stability. -->


### ТЕХНИЧЕСКИЕ ПРОБЛЕМЫ
<!-- ### TECHNICAL ISSUES -->

Все зависимости дублируются. Для корректной работы, каждый модуль изолирован от 
других. Каждый модуль должен самостоятельно заботиться о своих зависимостях. 
Для простых систем это проблема, потому что вы, в конечном счете, подключаете
одни и те же модули по нескольку раз.

<!-- Duplication, everywhere. In order to ensure each module is protected, it needs
to carry a copy of all its dependencies. For naive systems, this is a problem, 
as you can end up bundling the same module more than once. -->


Доступ к зависимостям вложенного модуля так же ограничен. Это правильно, потому
как эти зависимости уже не ваши. Они принадлежат модулю. Ваши зависимости — это
только те модули, которые находятся на первом уровне вложенности. Если вам нужен
доступ к модулю `animal` — вы должны самостоятельно скопировать этот модуль 
на первый уровень ваших завиисимостей.

<!-- Access to a dependency of a nested module is limited as well but rightly so.
Those are not your dependencies. They belong to the module. First level modules 
are your dependencies. If you need to access`animal` then it should be
duplicated on the first level of your modules. -->


### ПРОБЛЕМЫ КОММУНИКАЦИЙ(?)
<!-- ### SOCIAL ISSUES -->


Ответственность. Когда возникает проблема, отследить ее причину и сообщить об 
этом разработчику модуля не просто. Тем более, каждая новая версия модуля может
содержать абсолютно разные наборы модулей и, соответственно, иметь разных 
авторов.

<!-- Responsibility. When an issue does arise, tracking down the problem and
reporting to the appropriate maintainer is difficult. Even more so as each 
release of the module can switch to an entirely new set of modules and 
maintainers. -->


В больших корпорациях, отслеживание лицензий в модулях многоуровневых 
систем может обернуться тяжелым трудом. У каждого модуля обычно бывает несколько
разработчиков, которые могут либо быть черезвычайно активны, либо вообще исчезнуть
с лица земли по каким-то причинам.

<!-- For copyright lawyers in a corporate environment staying on top of licensing
can be a chore. Each module usually consists of multiple disconnected 
maintainers that can either be extremely active or for some reason have 
disappeared from the face of the earth. -->

### ПРИМЕР
<!-- ### EXAMPLE -->

Хороший пример многоуровневых зависимостей — [npm][4]. npm is hailed as the greatest
package manager for good a reason(???). Проблемы, о которых мы говорили учтены 
в npm.

<!-- [npm][4] is a fine example of nested dependencies and is hailed as the greatest
package manager for good a reason. It has thought about and solved all of the 
above issues. -->


**npm использует правильную модульность**, но оставляет вам возможности для того,
чтобы сделать все неправильно..

<!-- **npm does modules the right way** but still gives you the option to do it the
wrong way. -->

*   **Глобальные зависимости?**
    По-умолчанию, npm устанавливает модули локально, но модуль можно установить
    глобально, используя ключ `-g` или `--global`

<!-- *   **Globals Dependencies?**  
    npm defaults to local installs with an option to install globally with `-g`
   `- -global`. -->


*   **Плоские/Одноуровневые зависимости?**
    Вы можете перечислить зависимости, которые нужно установить пососедству 
    с вашим модулем с помощью поля `peerDependencies` в `package.json`.

<!-- *   **Flat/Peer Dependencies?**  
    npm will read the `peerDependencies` key of your `package.json` as an
    option to install them as neighbors to your package.-->

*   **Удобство разработки**
    Для того, чтобы связать модуль, который вы разрабатываете с вашим проектом
    можно воспользоваться командами `npm link` в директории модуля, 
    и `npm link <module>` в вашем проекте.
    [][5]<https://npmjs.org/doc/cli/npm-link.html> 

<!-- *   **Developer Friendly?**  
    Use `npm link` in the project you're developing and 
    `npm link <package>` to link the development package into your
    project.
   [][5]<https://npmjs.org/doc/cli/npm-link.html> 
 -->


*   **Дубликация модулей?**
     Для того, чтобы аккуратно вынести одинаковые версии пакетов на уровень выше,
     тем самым избавиться от дублирования в npm предусмотрен метод — `npm dedupe`.
     [][6]<https://npmjs.org/doc/cli/npm-dedupe.html> 

<!-- *   **Duplication?**  
    Use `npm dedupe` which will intelligently reduce the duplication in your
    package tree by moving common semver compatible dependencies up.
   [][6]<https://npmjs.org/doc/cli/npm-dedupe.html>  -->


*   **Лицензии и баги?**
    У каждого npm-модуля есть страница, на которой указаны лицензия и ссылки
    на репозиторий, домашнюю страницу, и баг-трекер модуля. Эту информацию автор
    модуля заполняет в `package.json`.

<!-- *   **License/Issue Resolution?**  
    npm has a page for each package listing the license, repo, homepage and
    bugs (as configured by the author
    ). -->


У npm есть отличный API. Если вы по каким-то причинам считаете, что npm вам 
не подходит, то вместо того, чтобы начать писать свой менеджер пакетов с нуля,
вы можете добавить функции, которых вам не хватает, с помощью API.

<!-- The best part is if you still don't agree npm is the right for you; it has a
great API. Rather than starting from scratch just extend npm through it's API 
and add the features you need. -->


## [Заключение][7]
<!-- ## [Conclusion][7] -->


Я верю в то, что модули должны быть небольшими и несвязанными. Модули должны быть
инкапсулированы — разработчики модулей не должны выходить за рамки собственных 
модулей и влиять на экосистему open source. Я рассуждаю так, отталкиваясь 
от того, насколько хорошо может развиваться проект, если создатели модулей 
находятся вне досягаемости.

<!-- I believe modules should try to be small and decoupled. No single or group of
maintainers should have control over any part of an open source ecosystem; only 
their own modules. I judge the success of an ecosystem based on how well it 
thrives outside the reach of it's creators. -->


Я призываю вас быть внимательными при создании структуры вашего кода. Помните
об экосистемах когда вы создаете фреймворк, или планируете сделать ваш код 
открытым. Помните о пакетных менеджерах, куда вы можете легко выложить ваш модуль.

<!-- I encourage you to be mindful when structuring your code, mindful of the
ecosystem when creating a framework or sharing code and mindful of the package 
managers your module is aimed towards. -->


 [1]: #global-dependencies

 [2]: #flat-dependencies

 [3]: #nested-dependencies
 [4]: http://dontkry.com/posts/code/npmjs.org
 [5]: https://npmjs.org/doc/cli/npm-link.html
 [6]: https://npmjs.org/doc/cli/npm-dedupe.html
 [7]: #conclusion