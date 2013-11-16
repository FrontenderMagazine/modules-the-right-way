Модули, сделанные по «правильному пути»<!-- («правильно»???) -->, обычно подразумевается — они 
работают. Они не падают с ошибками. Они не ведут себя странно, когда вы 
выкладываете их в продакшн. Они должны вести себя странно и когда с ними 
работают другие разработчики. Они не должны вести себя странно даже если через 
когда-нибудь кто-то решит обновить проект. Остальные проблемы, такие как 
количество файлов(???)<!-- Working code is far more important than saving a few kilobytes of
file space. --> и легкость поддержки(??) вторичны. 

<!--
A module done "the right way" primarily means: it works. It doesn't error or run
code in an unexpected way when you're deploying, have other developers running 
your code or come back to update a project some time in the future. Other 
problems like files space and ease of use should be secondary concerns.
-->


Я использую понятие «модули» в очень общем значении этого слова. Концепция
модульности применима и к структуре проекта, и к менеджеру пакетов, и к написанию
кода в целом(???). Я попытаюсь объяснить основную идею модульности на реальных 
примерах.

<!--
I am using the term "modules" in the most generic sense of the word. These
concepts can apply to your project structure, package manager or the very way 
you write code. I'll attempt to explain these concepts a generic way then 
provide real world examples.
-->


## [Глобальные зависимости][1]
<!-- ## [Global Dependencies][1] -->


Глобальные системы являются классическими, если все зависимости находятся 
в одном месте. Из этого пулла различные приложения могут подключать необходимые 
зависимости. 

<!-- Global systems are classic where all your dependencies reside in a single
location. Then multiple applications consume those dependencies from the same 
pool. -->


    | - modules
    | --- bear extends MODULES/animal
    | --- animal
    | - apps
    | --- grizzly extends MODULES/bear
    | --- koala extends MODULES/bear
    | --- panda extends MODULES/bear

Сначала такая структура кажется идеальной. It takes up the least amount of 
file space(???) и все ваши модули расположены в подходящем месте. Все ваши 
приложения, ссылаются на один модуль, так что если вы обновите такой модуль,
то он обновится во всех прилоджениях сразу.

<!-- Upon initial consideration this structure seems ideal. It takes up the least
amount of file space and all of your modules are located in one convenient place.
All of your applications point to the same module so it is really easy to update
that module across all of your projects. -->


Именно по этому такую конструкцию часто используют... Но **в таком подходе есть 
огромная брешь** 

<!-- For these reasons this structure is commonly used... but **it is incredibly
flawed**. -->


### ТЕХНИЧЕСКИЕ ПРОБЛЕМЫ
<!-- ### TECHNICAL ISSUES -->


В большинстве систем обновления модулей происходят часто. Но обратная совместимость
и хорошее версионирование не гарантируется. Обновление модулей вполне может 
сломать все ваши приложения. Глобальные зависимости подразумевают, что вы 
автоматически обновляете свои приложения каждый раз, когда обновляется код любого 
глобального модуля.

<!-- Updating modules, for most systems, happens frequently. Backwards compatibility
and good versioning are not a guarantee. Updating modules will eventually break 
your apps. Global dependencies mean you are forced to update**ALL** of your
apps each time you update a global dependency. -->


Чтож, если у вас много проектов, поддерживать их при каждом обновлении зависимостей
— это не просто кошмар. Часто, это еще и невозможно физически. 

<!-- So unless you only have a couple of projects, keeping every project in step
with dependency updates is a nightmare and most of the time not even physically 
possible. -->


### ПРОБЛЕМЫ КОМУНИКАЦИИ(?)
<!-- ### SOCIAL ISSUES -->


Глобальными зависимостями сложно делиться потому что они зависят от окружения, 
которое находится за пределами проекта. Обычно проблема с воссозданием окружения 
решается с помощью списка шагов в readme, или с помощью механизма, который 
делает это автоматически. Без этого проект не запустить.

<!-- Global dependencies are not easily shareable as they rely on an environment
setup outside of the project. Which usually requires a project to request users
(usually via a bulleted list of steps in a readme) or provide a switching 
mechanism to recreate the environment in order to run the project. -->


Еще одна проблема в том, что разработчикам в таких систамах часто приходится 
поддерживать сразу несколько проектов. Особенно это актуально для open source
проектов. Это не практично.

<!-- For diverse teams, especially open source software teams with developers
contributing to a diverse array of projects,**this is not practical**. -->


### ПРИМЕР
<!-- ### EXAMPLE -->


В течении года вы собираетесь разрабатывать сайты для различных клиентов. Вы
создаете папку для каждого клиента. В каждом проекте вы используете одинаковые
скрипты с утититами. В конце концов, вместо того, чтобы хранить эти утилиты
внутри каждого проекта, вы вынесли их в отдельную директорию:

<!-- You plan on building websites for various clients throughout the year. You
create a project folder for each client and use the same utility scripts for 
every project. So lazily you place those utilities in a single folder and then 
consume it with each project: -->

    // /Users/dude/scripts/utils.js
    var utils = module.exports = {};
    utils.slug = function(str) {
      return str.toLowerCase().replace(/ /g, '-').replace(/[^\w-]+/g, '');
    };


И, затем, подключили их в каждом вашем проекте:

<!-- and then within each of your projects: -->

    // /Users/dude/projects/acme/blog.js
    var utils = require('/Users/dude/scripts/utils.js');
    var title = utils.slug('Acme Blog Post'); // acme-blog-post


Бизнес со временем разрастается, и вот, у вас все больше и больше проектов. 
Однажды, один из ваших клиентов напишет заголовок «Blogs - How do they work?» в своем
блоге. После обработки slug вернет строку `blogs---how-do-they-work`. Клиент 
пожалуется вам, и вы обновите вашу функцию:

<!-- Over time business is doing well and you add more and more projects. One day a
client enters the blog title "Blogs - How do they work?" and the slug produced 
is`blogs- - -how-do-they-work`. They complain and you update your slug utility: -->

    utils.slug = function(str) {
      return str.toLowerCase().replace(/[^\w ]+/g, '').replace(/ +/g, '-');
    };


Теперь результат выглядит так: `blogs-how-do-they-work`.

<!-- Which will produce the more desired `blogs-how-do-they-work`. -->



Чтож, теперь каждый проект, который использовал старый формат slug'а сломан. 
Вы не поймете этого, пока не возьметесь обновить какой-нибудь старый проект.
Вероятно, вы обнаружите, что на страницах ваших клиентов теперь ошибка `404`, 
потому что их url'ы изменились при обновлении. Забавный момент.

<!-- Now every single past project that used to rely on the previous slug format is
broken. You won't realize this until you're doing updates on a past project. You
'll likely find out your client's pages are now 404ing because they're URLs are 
changing as they update. Fun times. -->


Я уверен, что вы породумаете механизм обратной совместимости для этой 
маленькой функции. Но нужно быть очень осторожным, изменяя любую глобальную
зависимость. Нужно помнить о каждом проекте, который использует эту зависимость.
Это тяжело и не практично для разработчиков, и это практически невозможно для 
команд. 

<!-- I'm sure you can think of a way to retain backwards compatibility for this
simple one line function. But the point is, when editing a global dependency you
must be mindful of every single project that uses it. That is difficult and 
unnecessary for a developer and nearly impossible for a development team. -->


**Глобальные зависимости приводят к сломанному коду**

<!-- **Global dependencies will eventually produce broken code.** -->


Вместо этого, вы должны вынести эту утилиты в отдельный модуль и установить его 
локально в каждый проект. Рабочий код намного важнее, чем несколько 
сэкономленных килобайт.

Instead you should turn that utility into a module and install it locally into
every project. Working code is far more important than saving a few kilobytes of
file space.


## [Flat Dependencies][2]

A flat structure or peer dependencies is a step towards the right way. Each
project has its own`modules` location so each project can be updated only as
needed. Each project is also shareable as the user can recreate the environment 
quickly within the project folder to run the app.

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

Flat dependency structures only work well for projects with a small amount of
dependencies. They are also easily accessible since every module is located just
one folder down.

This structure begins to break down when the module tree becomes diverse. Our
`grizzly` needs to eat let's give him some `fish` which extends the `animal`
module:

    | - grizzly@0.1.0
    | --- modules
    | ----- fish@0.1.0 extends animal@0.1.0
    | ----- bear@0.1.0 extends animal@0.1.0
    | ----- animal@0.1.0

### TECHNICAL ISSUES

Everything is golden until `animal` updates to `0.2.0`. The maintainer of 
`bear` is active and updates to `animal@0.2.0`. You're working on 
`grizzly@0.2.0` which now relies on `bear@0.2.0`. But unfortunately the
maintainer of`fish` doesn't have time to update.

What do you do? Keep `bear@0.1.0` until the maintainer of `fish` gets time to
update? You have a deadline that requires those features in`bear@0.2.0`! Likely
at this point you'll be writing hacks to get`fish` to work *good enough*.

### SOCIAL ISSUES

Flat modules can break when another unanticipated module updates. This puts
pressure on the developer ecosystem to couple their modules together to avoid 
these potential conflicts. Which is why small, tight knit teams with minimal 
dependencies**get by** with this approach.

Open source software progresses through diversity. I believe a module structure
should encourage module decoupling. Developers shouldn't have to think and keep 
up with an entire ecosystem just to build a single module.

### EXAMPLE

Frameworks are a great example of modules that create peer dependent situations
.  
Let's create an Acme framework:

    var acme = module.exports = {
      config: {
        user: 'Dude'
      },
      announce: function() {
        console.log('Hi! My name is ' + this.config.user);
      },
    };

Now each Acme plugin requires an instance of the Acme object. Let's create a
plugin:

    module.exports = function(acme) {
      if (acme.config.user) acme.announce.call(acme);
      else console.log('User not found');
    };

The plugin doesn't consume Acme as a dependency but the instance of Acme is
required for the plugin to run. Therefore it is a peer dependency.

This architecture seems convenient from a plugin author perspective but it has
a couple of problems:

*   Down the road new versions of the Acme framework are released. Each project
    can only have one version of your Acme framework installed. The user is forced 
    to upgrade every single plugin they use in order to use the new version of your 
    framework.
   
*   Your plugin will only work with the Acme framework. You should be a good
    open source citizen and make your plugin generic. Then users of other frameworks
    or users who don't use a framework can consume your code. We don't need the same
    code written over and over custom tailored to each framework.
   

**Frameworks should encourage generic plugins.**

Here is a more generic approach that doesn't require an instance of acme thus
removing the peer dependency:

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

Now your plugin is future proof and available to everyone.

## [Nested Dependencies][3]

Nested dependencies solve the issues of global and flat systems. Each module is
its own project. These modules are portable and encapsulated.

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

Nested dependencies completely solve the versioning problem of the flat system
:

    | - grizzly@0.2.0
    | --- modules
    | ----- fish@0.1.0 extends animal@0.1.0
    | ------- animal@0.1.0
    | ----- bear@0.2.0 extends animal@0.2.0
    | ------- animal@0.2.0

This a safe approach. Each module author only has to worry about their own
dependencies. Thus allowing the ecosystem to thrive exponentially and operate 
with stability.

### TECHNICAL ISSUES

Duplication, everywhere. In order to ensure each module is protected, it needs
to carry a copy of all its dependencies. For naive systems, this is a problem, 
as you can end up bundling the same module more than once.

Access to a dependency of a nested module is limited as well but rightly so.
Those are not your dependencies. They belong to the module. First level modules 
are your dependencies. If you need to access`animal` then it should be
duplicated on the first level of your modules.

### SOCIAL ISSUES

Responsibility. When an issue does arise, tracking down the problem and
reporting to the appropriate maintainer is difficult. Even more so as each 
release of the module can switch to an entirely new set of modules and 
maintainers.

For copyright lawyers in a corporate environment staying on top of licensing
can be a chore. Each module usually consists of multiple disconnected 
maintainers that can either be extremely active or for some reason have 
disappeared from the face of the earth.

### EXAMPLE

[npm][4] is a fine example of nested dependencies and is hailed as the greatest
package manager for good a reason. It has thought about and solved all of the 
above issues.

**npm does modules the right way** but still gives you the option to do it the
wrong way.

*   **Globals Dependencies?**  
    npm defaults to local installs with an option to install globally with `-g`
   `--global`.
*   **Flat/Peer Dependencies?**  
    npm will read the `peerDependencies` key of your `package.json` as an
    option to install them as neighbors to your package.
   
*   **Developer Friendly?**  
    Use `npm link` in the project you're developing and 
    `npm link <package>` to link the development package into your
    project.
   [][5]<https://npmjs.org/doc/cli/npm-link.html> 
*   **Duplication?**  
    Use `npm dedupe` which will intelligently reduce the duplication in your
    package tree by moving common semver compatible dependencies up.
   [][6]<https://npmjs.org/doc/cli/npm-dedupe.html> 
*   **License/Issue Resolution?**  
    npm has a page for each package listing the license, repo, homepage and
    bugs (as configured by the author
    ).

The best part is if you still don't agree npm is the right for you; it has a
great API. Rather than starting from scratch just extend npm through it's API 
and add the features you need.

## [Conclusion][7]

I believe modules should try to be small and decoupled. No single or group of
maintainers should have control over any part of an open source ecosystem; only 
their own modules. I judge the success of an ecosystem based on how well it 
thrives outside the reach of it's creators.

I encourage you to be mindful when structuring your code, mindful of the
ecosystem when creating a framework or sharing code and mindful of the package 
managers your module is aimed towards.


 [1]: http://dontkry.com/posts/code/modules-the-right-way.html#global-dependencies

 [2]: http://dontkry.com/posts/code/modules-the-right-way.html#flat-dependencies

 [3]: http://dontkry.com/posts/code/modules-the-right-way.html#nested-dependencies
 [4]: http://dontkry.com/posts/code/npmjs.org
 [5]: https://npmjs.org/doc/cli/npm-link.html
 [6]: https://npmjs.org/doc/cli/npm-dedupe.html
 [7]: http://dontkry.com/posts/code/modules-the-right-way.html#conclusion