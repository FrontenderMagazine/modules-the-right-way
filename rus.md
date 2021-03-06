# Правильные модули

Самое главное в «правильно» реализованных модулях это то, что они работают. Они 
не отваливаются и не ведут себя самым неожиданным образом во время деплоя, когда 
с вашим кодом работают другие разработчики или когда вы возвращаетесь к ним, 
что бы обновить. По сравнению с этим простота использования модуля или его вес 
глубоко вторичны.

Я использую понятие «модуль» в очень широком смысле. Концепция модульности 
применима как к структуре проекта, или менеджеру пакетов, так и к различным 
способам написания программного кода. В статье я попробую объяснить идею 
модульности на реальных примерах.


## Глобальные зависимости

Классические глобальные системы подразумевают, что модули, от которых 
зависят все ваши проекты, хранятся в одном месте:

    | - modules
    | --- bear extends MODULES/animal
    | --- animal
    | - apps
    | --- grizzly extends MODULES/bear
    | --- koala extends MODULES/bear
    | --- panda extends MODULES/bear

Поначалу, такая структура может показаться идеальной. Во-первых, все ваши модули
хранятся в одной директории. Во-вторых, нет дублирования кода: с одной стороны, 
это экономия места на диске, с другой, обновление любого модуля моментально 
попадет во все проекты, которые этот модуль используют.

Из-за этих очевидных плюсов такой подход используют очень часто. Но, 
к сожалению, он **потрясающе порочен**.


### Технические проблемы

Обновление модулей происходит часто. Но об обратной совместимости обновлений или
о хорошем версионировании нельзя говорить с уверенностью. Не будет ничего 
удивительного, если ваше приложение сломается после очередного обновления 
модулей. Глобальные зависимости подразумевают, что вы обновляете **все** свои 
приложения каждый раз, когда обновляется код любого глобального модуля. 

Если у вас много проектов, то обновлять их все при каждом обновлении 
зависимостей — это ужасно. Более того, порой это физически невозможно.


### Социальные проблемы

Глобальные зависимости сложно использовать, так как они основаны на окружении, 
настраиваемом за пределами проекта. Что, как правило, приводит к необходимости для 
пользователя воссоздавать это окружение у себя (например, следуя пунктам указанным 
в readme).

Для многих команд, особенно тех, которые занимаются разработкой программного 
обеспечения с открытым кодом, это не практично.

### Пример

В течение года, вы собираетесь писать сайты для большого количества клиентов. 
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
сгенерировала ваша функция `utils.slug`, клиент пожаловался вам. И вы исправили
вашу функцию:

    utils.slug = function(str) {
      return str.toLowerCase().replace(/[^\w ]+/g, '').replace(/ +/g, '-');
    };

Теперь результат выглядит более прилично: `blogs-how-do-they-work`.

Ну вот, теперь сломан каждый проект, который использовал старый формат функции 
`utils.slug`. Так что, кроме обновления `utils.slug`, вы должны обновить и 
все свои проекты. Кроме того, после обновления, некоторые ссылки на страницы 
ваших клиентов сломаются. Неприятно, не правда ли?

Уверен, что вы можете продумать механизм обратной совместимости для этой 
простой, однострочной функции. Но нужно быть очень осторожным. Изменяя код любой 
глобальной зависимости, нужно помнить о каждом проекте, который эту зависимость 
использует. Это тяжело для для одного разработчика и практически невозможно для 
команд.

**Использование глобальных зависимостей со временем приводит к ошибкам.**

Скрипт с утилитами стоило вынести в отдельный модуль, и установить его локально
в каждый проект. Рабочий код намного важнее, чем пара сэкономленных 
килобайт.


## Одноуровневые зависимости

Одноуровневые или плоские зависимости — это шаг в верном направлении. В такой
структуре, у каждого проекта есть своя директория `modules`. Это гарантия того, 
что зависимости проекта обновятся только в том случае, если это действительно 
необходимо. Начать работу над таким проектом не сложно. Окружение, необходимое
для запуска проекта, воссоздается прямо в его директории.

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

При небольшом количестве зависимостей одноуровневая структура работает отлично. 
Модули в ней легкодоступны, так как все необходимое находятся во вложенном 
директории.

Такая структура однако перестает работать с усложнением дерева зависимостей. 
`grizzly` голоден, дайте ему `fish`, которая наследует от модуля `animal`:


    | - grizzly@0.1.0
    | --- modules
    | ----- fish@0.1.0 extends animal@0.1.0
    | ----- bear@0.1.0 extends animal@0.1.0
    | ----- animal@0.1.0


### Технические проблемы

Все будет хорошо ровно до того момента, пока модуль `animal` не обновится 
до версии `0.2.0`. Разработчик `bear` активно работает над своим модулем, 
он сразу же возьмется за обновление. Вы будете погружены в работу над своим 
проектом `grizzly@0.2.0` когда обновление `bear@0.2.0` выйдет в свет. Но 
у разработчика `fish` свободного времени, к сожалению, не оказалось. 

Как вы поступите? Продолжите использовать `bear@0.1.0`, пока не обновится 
`fish`? Но у вас есть дедлайн. За определенный срок вы должны подключить 
к своему модулю новые возможности, которые реализованы в `bear@0.2.0`. 
Возможно, вы сможете написать хак, с которым модуль `fish` будет работать 
*достаточно хорошо*.


### Социальные проблемы

Одноуровневая структура может ломается, если какой-нибудь из модулей неожиданно 
обновится. Это создает давление на экосистему разработчиков, заставляя их всех
отслеживать потенциальные конфликты. Такой подход *работает* если разработчиков
немного, и они тесно связаны между собой.

Открытое программное обеспечение развивается благодаря многообразию решений. 
Я верю, что модульная структура должна поощрять разработку независимых модулей. 
Разработчики не должны быть вплетены в упряжку, которая тянет вперед огромную 
экосистему. Работать над своими собственными модулями вполне достаточно.


### Примеры

Фреймворки — замечательный источник проблем с одноуровневыми зависимостями.

К примеру, создадим фреймворк Acme:

    var acme = module.exports = {
      config: {
        user: 'Dude'
      },
      announce: function() {
        console.log('Hi! My name is ' + this.config.user);
      },
    };


Каждый плагин требует инстанс нашего фреймворка. Создадим плагин:

    module.exports = function(acme) {
      if (acme.config.user) acme.announce.call(acme);
      else console.log('User not found');
    };

Плагин не указывает Acme как собственную зависимость, но инстанс Acme необходим
ему для работы. Это и есть плоская зависимость. 

Такая архитектура выглядит удобно с точки зрения автора, но она имеет ряд 
проблем:

*   В будущем выйдет новая версия фреймворка. Но, для того, чтобы перейти 
    на нее, пользователю придется обновить каждый плагин, который он использует.

*   В будущем выйдет новая версия фреймворка. Каждый проект может иметь только 
    одну версию Acme. Пользователю придется обновить каждый плагин для того, 
    чтобы использовать новую версию фреймворка.

*   Ваш плагин работает только с фреймворком Acme. Вам следовало бы сделать
    код более общим. Тогда пользователи других фреймворков, или пользователи, 
    которые вообще не используют фреймворки смогут воспользоваться вашим кодом. 
    Нет нужды переписывать один и тот же код снова и снова для каждого нового 
    фреймворка. 

**Фреймворкам следует поощрять создание независимых плагинов, работа которых 
возможна не только с конкретным фреймворком.**

Вот пример более общего кода, который не требует инстанса Acme, избавляясь таким
образом от плоской зависимости:

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

Теперь наш плагин доступен для всех.


## Многоуровневые зависимости

Многоуровневые (вложенные) зависимости не имеют проблем, присущих глобальным и 
одноуровневым системам. В такой системе каждый модуль представляет собой 
отдельный проект со своими собственными зависимостями. Благодаря инкапсуляции, 
такие модули легко использовать повторно в других проектах.

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

Вложенные зависимости полностью избавляют от конфликтов различных версий 
модулей, присущих одноуровневым системам:

    | - grizzly@0.2.0
    | --- modules
    | ----- fish@0.1.0 extends animal@0.1.0
    | ------- animal@0.1.0
    | ----- bear@0.2.0 extends animal@0.2.0
    | ------- animal@0.2.0

Такой подход безопасен. Теперь автор каждого модуля заботится исключительно
о собственных зависимостях. Таким образом, экосистема развивается 
в геометрической прогрессии, при этом оставаясь стабильной.


### Технические проблемы

Все зависимости дублируются. Для корректной работы, каждый модуль изолируется 
от других. Каждый модуль должен самостоятельно заботиться о своих зависимостях. 
Для простых систем это неудобно — в конечном счете, одни и те же модули 
подключаются по нескольку раз.

Доступ к зависимостям вложенного модуля так же ограничен. Это правильно, потому 
как это не ваши зависимости; они принадлежат этому модулю. Ваши зависимости — 
это только те модули, которые находятся на первом уровне вложенности. Если вам 
нужен доступ к модулю `animal` — вы должны самостоятельно сдублировать этот 
модуль на первый уровень ваших зависимостей.


### Социальные проблемы

Ответственность. Когда возникает проблема, отследить ее причину и сообщить об 
этом разработчику модуля непросто. Тем более, учитывая, что каждая новая 
версия модуля может содержать абсолютно разные наборы зависимостей, и, 
соответственно, иметь совершенно разных авторов.


В больших корпорациях, отслеживание лицензий в модулях многоуровневых 
систем может быть тяжелым процессом. У каждого модуля обычно бывает несколько
разработчиков, которые могут либо быть чрезвычайно активны, либо без причины
пропадать на долгий срок.


### Примеры

Хороший пример многоуровневых зависимостей — [npm][1]. Его считают лучшим пакетным 
менеджером не без причины. В нем предложено решение всех упомянутых выше проблем.

**npm использует правильную реализацию модульности**, но кроме того этот пакетный 
менеджер дает вам возможность выбрать неверный путь.

*   **Глобальные зависимости?**
    По умолчанию, npm устанавливает модули локально, но при желании это можно 
    сделать глобально, используя ключ `-g` или `--global`.

*   **Одноуровневые зависимости?**
    Вы можете перечислить зависимости, которые нужно установить по соседству 
    с вашим модулем с помощью поля `peerDependencies` в `package.json`.

*   **Удобство разработки?**
    Вы можете в обход пакетного менеджера создать в вашем проекте ссылку 
    на модуль, который вы разрабатываете. Для этого существует команда 
    `npm link`.
    [https://npmjs.org/doc/cli/npm-link.html][2] 

*   **Дублирование модулей?**
    Для того, чтобы аккуратно вынести одинаковые версии пакетов на уровень выше,
    тем самым избавиться от дублирования в npm предусмотрен метод — `npm dedupe`.
    [https://npmjs.org/doc/cli/npm-dedupe.html][3] 

*   **Информация о лицензии и особенностях использования?**
    У каждого npm-модуля есть страница, на которой может быть указана лицензия, 
    перечислены ссылки на репозиторий, домашнюю страницу, и баг-трекер модуля. 
    Эту информацию автор модуля заполняет в файле `package.json`.

Кроме того, у npm есть отличный API. Если в npm вам чего-то не хватает, то 
совершенно не обязательно начинать проектировать свой собственный проектный 
менеджер. Вполне вероятно, что вы сможете добавить функции, которых вам 
не хватает, с помощью API.


## Заключение

Я верю, что модули должны быть маленькими и независимыми. Модули должны быть
инкапсулированы — разработчики модулей не должны выходить за рамки собственных 
модулей, работая на всю экосистему разом. Я рассуждаю так, отталкиваясь 
от того, насколько хорошо может развиваться проект, если создатели модулей 
на некоторое время перестанут над ними работать.

Я призываю вас к внимательности при создании структуры вашего кода. Помните
об экосистемах, когда вы создаете фреймворк, или планируете сделать ваш код 
открытым. И не забывайте о менеджерах пакетов, которые абсолютно открыты 
для ваших проектов.


[1]: http://dontkry.com/posts/code/npmjs.org
[2]: https://npmjs.org/doc/cli/npm-link.html
[3]: https://npmjs.org/doc/cli/npm-dedupe.html