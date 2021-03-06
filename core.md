#Описание ядра LiveStreet

*[Эту статью](http://livestreet.ru/blog/dev_documentation/113.html) написал Максим Мжельский в далеком 2008 году, она морально устрарела и требует переработки, но общее представление всё же дает.*

##Общее представление

Движок LiveStreet построен на базе собственного фреймворка с использованием модульности и модели MVC.
Фреймворк представляет из себя каркас из абстрактных классов(абстракции `module, action, block, mapper, entity), ядро(engine), роутер(route) и набор системных модулей(модули с префиксом sys_).
Обшая картина выглядит так:

![Структура](http://livestreet.ru/uploads/images/1/ab5cb2212d.png)

* Роутер(Route.class.php), он же контроллер, производит разбор запрашиваемого URL и определяет какой экшен(action) необходимо запустить, определяет метод экшена евент(event) и параметры(params). Также инициализирует ядро. Запуск роутера происходит автоматически при каждом запросе к сайту в файле index.php.

* Ядро(Engine.class.php) это сердце движка, в нем происходит инициализация всех модулей и реализован механизм доступа к методам модулей через $this->ModuleName_ModuleMethod(params) Ядро инициализируется в роутере, но так же можно сделать это в другом месте, например, в обработчиках Аякса.

* Модуль(Module.class.php) абстракция модуля, от неё наследуются все модули в движке. Предоставляет возможность доступа к модулям, методы инициализации и завершения модуля.

* Экшен(Action.class.php) абстракци экшена, от неё наследуются все экшены движка. Предоставляет возможность доступа к модулям и параметрам переданым в URL

* Блок(Block.class.php) абстракция обработчика блока в шаблонах. Также предоставляет доступ к модулям. Например, облако тегов обрабатывает отдельный обработчки блока.

* Маппер(Mapper.class.php) абстракция мапперов — классов для работы с базой данных, содержащих SQL запросы.

* Сущность(Entity.class.php) абстракция сущности, например, сущность user(пользователь). Позволяет получать/устанавливать свойства сущности, и автоматическую их загрузку, например из SQL запроса.

Описание конфигов:

* config.php — главный системный конфиг, содержит много разных настроект, настоятельно рекомендуется заглянуть в него и настроить движок «под себя»

* config.db.php — содержит дефолтные настройки к базе данных

* config.table.php — содержит описание таблиц используемых в БД

* config.route.php — содержит настройки роутинга страниц. Соответствия URL и классы экшена

* config.ajax.php — содержит настройки необходимые для аякс обработчиков

* config.module.php — содержит конфигурацию модулей, сейчас там только список модулей запускающихся автоматически при запросе к сайту

* config.memcache.php — содержит настройки системы кеширования memcached

Всё это работает по следующей схеме.
Пользователь запрашивает URL вида livestreet.ru/page/about/, роутер разбирает этот URL и получает на выходе action=page, event=about, params[](в данном запросе параметры отсутствуют).
Долее он пытается найти этот экшен в конфиге config.route.php, если находит то запускает соответствующий класс экшена и передает ему евент+параметры. Экшен при запуске проверяет зарегистрирован ли переданный ему евент, если да то запускает необходимый метод, который привязан к евенту, если нет то срабатывает метод EventNotFound(). После выполения метода евента экшен завершает свою работу запуском метода EventShutdown().
Стоит обратить внимание на то, что экшен может делать внутренний редирект внутри метода евента. Это достигается таким вызовом в методе евента: return Router::Action('error'); Тогда произойдет передача управление на другой экшен(error). После того как экшен отработает(или цепочка экшенов) происходит завершение модулей(вызов методов Shutdown()), загрузка системных переменных в шаблон(пути и другие константы) и вывод шаблона на экран. Т.е. как видим основная логика сайта реализуется в экшенах. В них можно обращаться к различным модулям, в том числе и системным.

Модули служат для реализации какого либо функционала, без внешней логики, т.е. некие кирпичики, из которых в экшенах можно построить логику.

Файловая структура LiveStreet:

![Файловая структура](http://livestreet.ru/uploads/images/1/e847c5257f.gif)

* /classes/actions/ — содержит классы экшенов, т.е. логика поведения сайта лежит здесь

* /classes/blocks/ — содержит обработчики блоков в шаблонах. например, обработчик облака тегов

* /classes/engine/ — содержит системные классы движка, в них лучше ничего не менять

* /classes/lib/ — содержит используемые внешние библиотеки

* /classes/modules/ — содержит модули, каждый модуль в отдельном каталоге, имена каталогов и модулей должны совпадать

* /config/ — содержит конфиги для настройки движка

* /include/ — содержит дополнительные какие либо средства для работы двига, например, function.php где определены дополнительные функции

* /include/ajax/ — содержит обработчики аякса, например, обработчик голосования за топик

* /logs/ — содержит логи движка

* /templates/cache/ — кеш файлы шаблонов Smarty

* /tamplates/compiled/ — компилированные файлы шаблонов Smarty

* /tamplates/skin/ — содержит темы оформления движка в виде шаблонов Smarty

* /tamplates/skin/habra/actions/ — содержит шаблоны к экшенам, для каждого экшена отдельный каталог с шаблонами, имя должно совпадать с именем класса экшена

* /uploads/ — содержит загруженные пользователями файлы

Описание стандартных методов Action

* AddEvent($sEventName,$sEventFunction) — добавляет новый евент в экшен, т.е. новое событие. sEventName — название евента, то что передаётся в URL'е. sEventFunction — название метода который будет вызван для обработки этого евента

* SetDefaultEvent($sEvent) — устанавливает какой эвент будет запускаться по дефолту, когда в URL не передан евент. sEvent — название евента. Должен быть вызван при инициализации экшена

* GetDefaultEvent() — возвращает название дефолтного евента

* GetParam($iOffset) — возвращает параметр по его смещению в URL, если параметра нет, то возвращает null

* SetParam($iOffset,$value) — подменяет параметр из URL по его смещению. iOffset — смещение, начинается с нуля. value — новое значение параметра

* SetTemplate($sTemplate) — устанавливает шаблона Smarty, который будет использован для вывода. Путь до шаблона относительно каталога с темой. По умолчанию шаблон совпадает с названием евента

* SetTemplateAction($sTemplate) — устанавливает шаблона Smarty, который будет использован для вывода. Путь до шаблона относительно каталога с шаблонами экшена

* GetTemplate() — возвращает используемый шаблон для вывода. Путь до шаблона относительно каталога с темой

* GetActionClass() — возвращает имя класса экшена, которое совпадает с каталогом шаблонов экшена

* EventNotFound() — вызывается если не найден переданный в URL евент. Как же этот метод можно использовать для перенаправления на страницу ошибки 404: return EventNotFound();

* EventShutdown() — автоматически вызывается при завершении работы экшена

* Init() — это метод должен всегда присутствовать в экшене, обычно в нем происходит какая то инициализация экшена

* RegisterEvent() — также обязательный метод, в нём должны быть добавлены евенты через метод AddEvent()