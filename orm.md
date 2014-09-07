#ORM в LiveStreet
##Что такое ORM?
ORM или Object-relational mapping (рус. Объектно-реляционное отображение) — это технология программирования, которая позволяет преобразовывать несовместимые типы моделей в ООП, в частности, между хранилищем данных и объектами программирования.

Ключевой особенностью ORM является отображение, которое используется для привязки объекта к его данным в хранилище данных (БД). ORM как бы создает «виртуальную» схему базы данных в памяти и позволяет манипулировать данными уже на уровне объектов. Использование ORM избавляет разработчика от необходимости работы с SQL и написания большого количества кода, часто однообразного и подверженного ошибкам. Весь генерируемый ORM код предположительно валиден, поэтому не нужно задумываться о его тестировании. Основной минус ORM — некоторое снижение производительности по сравнению с обычными SQL-запросами.
Реализация ORM в LiveStreet
---------------------------
Чтобы начать использовать стандартные методы ORM LiveStreet, необходимо использовать наследование. Модули наследуются от  `ModuleORM`, сущности от `EntityORM`, мапперы от `MapperORM`:
~~~
class ModuleTest extends ModuleORM {}
~~~
~~~
class ModuleTest_EntityTest extends EntityORM {}
~~~
~~~
class ModuleTest_MapperTest extends MapperORM {}
~~~
При этом они сразу становятся обладателями стандартных методов ORM: `get*()`, `Add()`, `Save()`, `GetBy*()`, `GetItemsBy*()`, `Delete()` и т.д. Например:
~~~
// Создание пустой сущности
$oTest = Engine::GetEntity('ModuleTest_EntityTestEntity');
// Можно короче
$oTest = LS::Ent('Test_TestEntity');

// Задаем свойство
$oTest->setTitle('Тестовый заголовок');

// Сохранение сущности в таблице `prefix_test`
$oTest->Add();
// или
$oTest->Save();

// Поиск сущности по ключу
$oTest = Engine::GetInstance()->Test_GetTestByTitle('Тестовый заголовок');
// Или short-alias
$oTest = LS::E()->Test_GetTestByTitle('Тестовый заголовок');
~~~
Таблицы в базе данных должны называться следующим образом: `prefix_<module-name>_<entity-name>` (или можно прописать их соответствия в конфиге), а если название сущности совпадает с названием модуля, то достаточно назвать таблицу так: `prefix_<module-name>`. Поля в таблицах именуются аналогично: `<entity-name>_<field-name>`, или просто `<field-name>`.