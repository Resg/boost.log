# boost.log
[Boost.Log](http://www.boost.org/doc/libs/1_62_0/libs/log/doc/html/index.html) - библиотека логирования в Boost. Она поддерживает многочисленные бэкэнды к логируемым данным в различных форматах.
К бэкэндам получают доступ через фронтэнды, которые связывают службы и различными путями передают логируемые данные.
Например, есть фронтэнд, который использует поток, чтобы передать запись в лог асинхронно. У фронтэндов могут быть фильтры, 
чтобы проигнорировать определенные данные. И они определяют, записи отформатированными как строки. Все эти функции
расширяемы, что делает Boost.Log мощной библиотекой.

`Пример 62.1. Бэкэнд, фронтэнд, ядро и логгер`
<a name="example621"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/logger.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>

using namespace boost::log;

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);

  core::get()->add_sink(sink);

  sources::logger lg;

  BOOST_LOG(lg) << "note";
  sink->flush();
}
```

[Пример 62.1](#example621) представляет важные составляющие Boost.Log. Boost.Log предоставляет Вам доступ к бэкэндам, фронтэндам, ядру и логгерам:

* Бэкэнды решают, где данные будут записаны. **boost::log::sinks::text_ostream_backend*** инициализирован с потоком типа
  **std::ostream** и делает записи к нему.

* Фронтэнды - соединение между ядром и бэкэндом. Они реализуют различные функции, которые не должны быть реализованы каждым отдельным бэкэндом. Например, могут быть добавлены фильтры к фронтэнду, чтобы выбрать, какие записи в логе будут переданы бэкэнду,а какие нет.

* [Пример 62.1](#example621) использует фронтэнд **boost::log::sinks::asynchronous_sink**. Вы должны использовать фронтэнд, даже если
  вы не используете фильтры. **boost::log::sinks::asynchronous_sink** использует поток, который добавляет записи к бэкэнду асинхронно.
  Это может улучшить производительность, но задерживает операции записи.
  
* Ядро - центральный компонент, через который производятся все записи. Оно реализовано как синглтон. Чтобы получить указатель на ядро, 
  вызовите **boost::log::core::get()**.

  Фронтэнды должны быть добавлены к ядру, чтобы получить записи в логе. Передача записей фронтэндам зависит от фильтров в ядре.
  Фильтры могут быть зарегистрированы или во фронтэндах или в ядре. Фильтры, зарегистрированные в ядре, являются глобальной переменной,
  а фильтры, зарегистрированные во фронтэндах, локальны. Если запись в логе отфильтрована ядром, она не будет передана никакому
  фронтэнду. Если она фильтруется по фронтэнду, то эта запись все еще может быть обработана другими фронтэндами и передана их бэкэндам.

* Логгер - компонент в Boost.Log, который вы будете использовать чаще всего. В то время как вы получаете доступ к бэкэндам,
  фронтэндам и ядру только, когда вы инициализируете библиотеку логирования, вы используете логгер каждый раз, когда вы делаете
  запись в журнале. Логгер делает запись в ядро.

  Логгер в [Примере 62.1](#example621) есть тип **boost::log::sources::logger**. Это - самый простой логгер. Когда вы захотите
  записать запись в логе, используйте макро **BOOST_LOG** и передайте логгер в качестве параметра. Запись в логе создается при
  записи данных в макрос, как будто это - поток типа **std::ostream**.

Бэкэнд, фронтэнд, ядро и логгер работают вместе. **boost::log::sinks::asynchronous_sink**, фронтэнд, является шаблоном,
который получает бекенд **boost::log::sinks::text_ostream_backend** в качестве параметра. Впоследствии, фронтэнд инстанцируют
с **boost::shared_ptr**. Требуется умный указатель, чтобы зарегистрировать фронтэнд в ядре: вызов **boost::log::core::add_sink()** требует
**boost::shared_ptr**.

Поскольку бэкэнд - шаблонный параметр фронтэнда, он может быть сконфигурирован только после того, как фронтэнд инстанцировали. Бэкэнд
определяет, как это сделано. Член-функция **add_stream ()** обеспечена бэкэндом **boost::log::sinks::text_ostream_backend**,
чтобы добавлять потоки. Вы можете добавить больше чем один поток в повысить **boost::log::sinks::text_ostream_backend**. Другие
бэкэнды обеспечивают различные член-функции для конфигурации. Консультируйтесь с документацией для деталей.

Чтобы получить доступ к бэкэнду, все фронтэнды обеспечивают член-функцию **locked_backend ()**. Эту член-функцию вызывают
**locked_backend ()**, потому что это возвращает указатель, который обеспечивает, синхронизировал доступ к бэкэнду, пока указатель
существует. Вы можете получить доступ к бэкэнду через указатели, возвращенные **locked_backend ()** от многократных потоков, без
необходимости синхронизировать доступ самому.

Вы можете инстанцировать логгер как **boost::log::sources::logger** с конструктором по умолчанию. логгер
автоматически вызывает **boost::log::core::get()**, чтобы передать записи в журнале ядру.

Вы можете получить доступ к логгерам без макросов. логгеры - объекты с член-функциями, которые Вы можете вызвать. Однако
макросы как BOOST_LOG упрощают писать записи в логе. Без макросов не было бы возможно записать запись в логе в одной строке кода.

[Пример 62.1](#example621) вызовов **boost::log::sinks::asynchronous_sink::flush()** в конце **main()**. Этот вызов требуется, потому что
фронтэнд асинхронный и использует поток, чтобы передать записи в журнале. Вызов подтверждает, что все буферизированные записи в
журнале переданы к бэкэнду и записаны. Без вызова **flush()**, пример мог завершиться, не выводя на экран `````note`````.

`Пример 62.2. boost::sources::severity_logger с фильтром`
<a name="example622"></a>
``` c++
#include <boost log="" common.hpp="">
#include <boost log="" sinks.hpp="">
#include <boost log="" sources="" severity_logger.hpp="">
#include <boost utility="" empty_deleter.hpp="">
#include <boost shared_ptr.hpp="">
#include <iostream>

using namespace boost::log;

bool only_warnings(const attribute_value_set &set)
{
  return set["Severity"].extract<int>() > 0;
}

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);
  sink->set_filter(&only_warnings);

  core::get()->add_sink(sink);

  sources::severity_logger<int> lg;

  BOOST_LOG(lg) << "note";
  BOOST_LOG_SEV(lg, 0) << "another note";
  BOOST_LOG_SEV(lg, 1) << "warning";
  sink->flush();
}
```
[Пример 62,2](#example622) основан на [примере 62.1](#example621), но он заменяет **boost::sources::logger** на **boost::sources::severity_logger**.
Этот логгер добавляет атрибут для уровня логировния для каждой записи лога. Чтобы задать уровень ведения логирования можно
использовать макрос **BOOST_LOG_SEV**.

Тип уровня лога зависит от шаблона параметра, передаваемого в **boost::sources::severity_logger**. [Пример 62,2](#example622) использует int.
Вот почему числа как 0 и 1 передаются **BOOST_LOG_SEV**. Если используется **BOOST_LOG**, уровень ведения журнала имеет значение 0.

[Пример 62,2](#example622) также вызывает **set_filter()** для регистрации фильтра в фронтэнде. Функция фильтра вызывается для каждой записи лога.
Если функция возвращает значение **true**, запись журнала направляется в back-end. [Пример 62,2](example622) определяет функцию **only_warnings()** с
возвращаемым значением типа **bool**.

**only_warnings()** ожидает параметр типа **boost::log::attribute_value_set**. Этот тип представляет записи лога в то время как они
передаются вокруг в рамках логирования. **Boost::log::Record** — еще один тип для записей лога, как оболочка для
**boost::log::attribute_value_set**. Этот тип предоставляет член-функцию **attribute_values()**, которая извлекает ссылку на
**boost::log::attribute_value_set**. Функции фильтра получают **boost::log::attribute_value_set** непосредственно, но не **boost::log::record**.
**Boost::log::attribute_value_set** хранит пары ключ/значение. Представте как **std::unordered_map**.

Записи лога состоят из атрибутов. Атрибуты имеют имя и значение. Можно создавать атрибуты самостоятельно. Они также могут
создаваться автоматически – например логгерами. В самом деле, именно поэтому Boost.Log предоставляет несколько средств ведения
лога. **Boost::log::Sources::severity_logger** добавляет атрибут, называющийся уровнем серьезности для каждой записи лога. Этот
атрибут сохраняет уровень лога. Таким образом фильтр может проверить, является ли уровень логирования записи лога больше 0.

**Boost::log::attribute_value_set** предоставляет несколько член-функций для доступа к атрибутам. Член-функции похожи на 
предоставляемые **std::unordered_map**. Например **boost::log::attribute_value_set** перегружает оператор **operator []**. Этот оператор возвращает
значение атрибута, имя которого передается в качестве параметра. Если атрибут не существует, он создается.

Тип имен атрибутов является **boost::log::attribute_name**. Этот класс предоставляет конструктор, который принимает строку, поэтому можно
передать строку непосредственно **operator []**, как и в [примере 62,2](#example622).

Тип значений атрибутов является **boost::log::attribute_value**. Этот класс предоставляет член-функции для получения значения атрибута
исходного типа. Поскольку уровень ведения журнала значение типа **int**, **int** передается как параметр шаблона **extract()**.

**Boost::log::attribute_value** также определяет член-функции **extract_or_default()** и **extract_or_throw()**.** Extract()** возвращает значение,
созданное с помощью конструктора по умолчанию, если тип преобразование завершается неудачей, – например 0 в случае **int**.
**extract_or_default()** возвращает значение по умолчанию, которое передается в качестве другого параметра этой член-функции.
**extract_or_throw()** создает исключение типа** boost::log::runtime_error** в случае ошибки.

Для безопасных типов преобразований, Boost.Log предоставляет посетителям функции **boost::log::visit()**, которую можно использовать вместо
**extract()**.

[Пример 62,2](#example622) выводит ```warning```. Эта запись лога имеет уровень логирования больше 0 и таким образом не фильтруется.

`Пример 62.3. Изменение формата записи журнала с set_formatter()`
<a name="example623"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>

using namespace boost::log;

void severity_and_message(const record_view &view, formatting_ostream &os)
{
  os << view.attribute_values()["Severity"].extract<int>() << ": " <<
    view.attribute_values()["Message"].extract<std::string>();
}

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);
  sink->set_formatter(&severity_and_message);

  core::get()->add_sink(sink);

  sources::severity_logger<int> lg;

  BOOST_LOG_SEV(lg, 0) << "note";
  BOOST_LOG_SEV(lg, 1) << "warning";
  sink->flush();
}
```
[Пример 62,3](#example623) основан на [примере 62,2](#example622). На этот раз показан уровень лога.

Фронтэнды предоставляют член-функции **set_formatter()**, которые могут быть переданы функции format. Если запись не фильтруется фронтэндом, она передается функции format. Эта функция форматирует запись лога как строку, которая затем передается от фронтэнда к
бэкенду. Если вы не вызываете **set_formatter()**, по умолчанию back-end получает только то, что находится на правой стороне макроса как
**BOOST_LOG**.

[Пример 62,3](#example623) передает функции **severity_and_message() set_formatter()**. **severity_and_message()** ожидает параметры типа
**boost::log::record_view** и **boost::log::formatting_ostream**. **Boost::log::record_view** — это представление на записи журнала. Это похоже на
**boost::log::record**. Однако **boost::log::record_view** является записью неизменяемого лога.

**Boost::log::record_view** предоставляет член-функцию **attribute_values()**, которая возвращает константную ссылку на
**boost::log::attribute_value_set**. **Boost::log::formatting_ostream** является потоком, используемым для создания строки, которая передается
бэкенду.

**severity_and_message()** обращается к атрибутам Severity  и Message. **extract()** вызывается для получения значений атрибутов, которые
затем записываются в поток. Severity возвращает уровень лога как значение типа **int**. Message предоставляет доступ к тому, что на
правой стороне макроса как **BOOST_LOG**. Обратитесь к документации для полного списка имен доступных атрибутов.

[Пример 62,3](#example623) использует фильтр. Пример записывает две записи лога: `0: note` и `1: warning`.

`Пример 62,4. Фильтрация записей лога и их форматирование лямбда-функциями`
<a name="example624"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/expressions.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>


using namespace boost::log;

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);
  sink->set_filter(expressions::attr<int>("Severity") > 0);
  sink->set_formatter(expressions::stream <<
    expressions::attr<int>("Severity") << ": " << expressions::smessage);

  core::get()->add_sink(sink);

  sources::severity_logger<int> lg;

  BOOST_LOG_SEV(lg, 0) << "note";
  BOOST_LOG_SEV(lg, 1) << "warning";
  BOOST_LOG_SEV(lg, 2) << "error";
  sink->flush();
}
```
[Пример 62.4](#example624) использует фильтр и функцию format. На этот раз функции реализованы как лямбда-функции – не как C ++ 11 лямбда-функции, а
как Boost.Phoenix лямбда-функции.

Boost.Log предоставляет хелперы для лямбда-функции в пространстве имен **boost::log::expressions**. Например *boost::log::expressions::stream*
представляет поток. *Boost::log::Expressions::smessage* предоставляет доступ ко всему, на правой стороне макроса как **BOOST_LOG**. **Boost::
log::expressions::attr()** можно использовать для доступа к любому атрибуту. Вместо **smessage** [пример 62.4](#example624) может использовать ` c++ attr<std::string> ("Message")`.

[пример 62.4](#example624) отображает `1: warning`  и `2: error`.

`Пример 62,5. Определение ключевых слов для атрибутов`
<a name="example625"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/expressions.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>

using namespace boost::log;

BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", int)

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);
  sink->set_filter(severity > 0);
  sink->set_formatter(expressions::stream << severity << ": " <<
    expressions::smessage);

  core::get()->add_sink(sink);

  sources::severity_logger<int> lg;

  BOOST_LOG_SEV(lg, 0) << "note";
  BOOST_LOG_SEV(lg, 1) << "warning";
  BOOST_LOG_SEV(lg, 2) << "error";
  sink->flush();
}
```
Boost.Log поддерживает определяемые пользователем ключевые слова. Можно использовать макрос  **BOOST_LOG_ATTRIBUTE_KEYWORD** чтобы определить
ключевые слова для доступа к атрибутам без необходимости повторно передавать имена атрибутов в виде строк
**boost::log::expressions::attr()**.

[пример 62.5](#example625) используется макрос **BOOST_LOG_ATTRIBUTE_KEYWORD** для определения *severity* ключевого слова. Макрос ожидает три параметра: имя
ключевого слова, имя атрибута как строка и тип атрибута. Ключевое слово new может использоваться в лямбда-функции фильтра и формата. Это
означает, что вы не ограничены использованием ключевых слов, таких как **boost::log::expressions::smessage**, которые предоставляются
Boost.Log-вы также можете определить новые ключевые слова.

Во всех примерах пока атрибуты, используемые являются те, которые определены в Boost.Log. Пример 62,6 показывает, как создать
пользовательские атрибуты.

`Пример 62.6. Определение атрибутов`
<a name="example626"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/expressions.hpp>
#include <boost/log/attributes.hpp>
#include <boost/log/support/date_time.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>

using namespace boost::log;

BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", int)
BOOST_LOG_ATTRIBUTE_KEYWORD(counter, "LineCounter", int)
BOOST_LOG_ATTRIBUTE_KEYWORD(timestamp, "Timestamp",
  boost::posix_time::ptime)

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);
  sink->set_filter(severity > 0);
  sink->set_formatter(expressions::stream << counter << " - " << severity <<
    ": " << expressions::smessage << " (" << timestamp << ")");

  core::get()->add_sink(sink);
  core::get()->add_global_attribute("LineCounter",
    attributes::counter<int>{});

  sources::severity_logger<int> lg;

  BOOST_LOG_SEV(lg, 0) << "note";
  BOOST_LOG_SEV(lg, 1) << "warning";
  {
    BOOST_LOG_SCOPED_LOGGER_ATTR(lg, "Timestamp", attributes::local_clock{})
    BOOST_LOG_SEV(lg, 2) << "error";
  }
  BOOST_LOG_SEV(lg, 2) << "another error";
  sink->flush();
}
```

Глобальный атрибут создается путем вызова **add_global_attribute()** ядра. Атрибут является глобальным, потому что он автоматически
добавляется в каждую запись журнала.

**add_global_attribute()** требует два параметра: имя и тип нового атрибута. Имя передается в виде строки. Для типа используется класс из
пространства имен **boost::log::attributes**, которое предоставляет классы для определения различных атрибутов.[пример 62.6](#example626) использует
**boost::log::attributes::counter** для определения атрибута LineCounter, который добавляет номер строки для каждой записи журнала. Этот
атрибут будет число записей журнала, начиная с 1.

**add_global_attribute()** не является шаблон функции. **Boost::log::Attributes::Counter** не передается в качестве параметра шаблона. Тип
атрибута должен быть создан и передан в качестве объекта.

[Пример 62.6](#example626) использует второй атрибут Timestamp. Это scoped атрибут, который создан с **BOOST_LOG_SCOPED_LOGGER_ATTR**. Этот макрос
добавляет атрибут средства ведения журнала. Первый параметр-это средство ведения журнала, второй — имя атрибута, а третий — объект
атрибута. Тип атрибута объекта является **boost::log::attribute::local_clock**. Атрибут имеет значение текущего времени для каждой записи
журнала.

Атрибут Timestamp добавляется запись лога “error”. Timestamp существует только в области, где используется
**BOOST_LOG_SCOPED_LOGGER_ATTR**. Когда область заканчивается, атрибут удаляется. **BOOST_LOG_SCOPED_LOGGER_ATTR** похож на вызов
**add_attribute()** и **remove_attribute()**.

Как и в [примеру 62.5](#example625), [Пример 62.6](#example626) использует макрос **BOOST_LOG_ATTRIBUTE_KEYWORD**, чтобы определить ключевые слова для новых атрибутов.
Функция format получает доступ ключевые слова, чтобы написать номер строки и текущее время. Значение timestamp будет пустой строкой для
этих записей журнала где атрибут Timestamp является неопределенным.

`Пример 62,7. Вспомогательные функции для фильтров и форматов`
<a name="example627"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/expressions.hpp>
#include <boost/log/attributes.hpp>
#include <boost/log/support/date_time.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <iomanip>

using namespace boost::log;

BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", int)
BOOST_LOG_ATTRIBUTE_KEYWORD(counter, "LineCounter", int)
BOOST_LOG_ATTRIBUTE_KEYWORD(timestamp, "Timestamp",
  boost::posix_time::ptime)

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);
  sink->set_filter(expressions::is_in_range(severity, 1, 3));
  sink->set_formatter(expressions::stream << std::setw(5) << counter <<
    " - " << severity << ": " << expressions::smessage << " (" <<
    expressions::format_date_time(timestamp, "%H:%M:%S") << ")");

  core::get()->add_sink(sink);
  core::get()->add_global_attribute("LineCounter",
    attributes::counter<int>{});

  sources::severity_logger<int> lg;

  BOOST_LOG_SEV(lg, 0) << "note";
  BOOST_LOG_SEV(lg, 1) << "warning";
  {
    BOOST_LOG_SCOPED_LOGGER_ATTR(lg, "Timestamp", attributes::local_clock{})
    BOOST_LOG_SEV(lg, 2) << "error";
  }
  BOOST_LOG_SEV(lg, 2) << "another error";
  sink->flush();
}
```
Boost.Log предоставляет многочисленные вспомогательные функции для фильтров и форматов. [Пример 62.7](#example627) вызывает помошника **boost::log::expressions::is_in_range()** для фильтрации записей журнала, чей уровень логирования находится вне диапазона. **Boost::
log::expressions::is_in_range()** ожидает атрибут в качестве первого параметра и нижней и верхней границы его второй и третий параметры
. Как и в случае с итераторами, верхняя граница является исключительным и не принадлежит к диапазону.

**boost:: log::expressions::format_date_time()** вызывается в функции format. Он используется для форматирования timepoint. [Пример 62.7](#example627)
использует **boost::log::expressions::is_in_range()** для записи времени без даты. Можно также использовать манипуляторы из
стандартной библиотеки в функции format. [Пример 62.7](#example627) использует **std::setw()** для задания ширины для счетчика.

`Пример 62.8. Несколько средств ведения журнала, фронт эндов и обратно концов`
<a name="example628"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/sources/channel_logger.hpp>
#include <boost/log/expressions.hpp>
#include <boost/log/attributes.hpp>
#include <boost/log/utility/string_literal.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <string>

using namespace boost::log;

BOOST_LOG_ATTRIBUTE_KEYWORD(severity, "Severity", int)
BOOST_LOG_ATTRIBUTE_KEYWORD(channel, "Channel", std::string)

int main()
{
  typedef sinks::asynchronous_sink<sinks::text_ostream_backend>
    ostream_sink;
  boost::shared_ptr<ostream_sink> ostream =
    boost::make_shared<ostream_sink>();
  boost::shared_ptr<std::ostream> clog{&std::clog,
    boost::empty_deleter{}};
  ostream->locked_backend()->add_stream(clog);
  core::get()->add_sink(ostream);

  typedef sinks::synchronous_sink<sinks::text_multifile_backend>
    multifile_sink;
  boost::shared_ptr<multifile_sink> multifile =
    boost::make_shared<multifile_sink>();
  multifile->locked_backend()->set_file_name_composer(
    sinks::file::as_file_name_composer(expressions::stream <<
    channel.or_default<std::string>("None") << "-" <<
    severity.or_default(0) << ".log"));
  core::get()->add_sink(multifile);

  sources::severity_logger<int> severity_lg;
  sources::channel_logger<> channel_lg{keywords::channel = "Main"};

  BOOST_LOG_SEV(severity_lg, 1) << "severity message";
  BOOST_LOG(channel_lg) << "channel message";
  ostream->flush();
}
```
[Пример 62.8](#example628) использует несколько средств ведения лога, фронтэндов и бекэндов. В дополнение к использованию классов
**boost::log::sinks::asynchronous_sink**, **boost::log::sinks::text_ostream_backend** и **boost::log::sources::severity_logge**, в примере также
используется фронтэнд **boost::log::sinks::synchronous_sink**, бекэнд **boost::log::sinks::text_multifile_backend**  и логгер
**boost::log::sources::channel_logger**.

Интерфейсный **boost::log::sinks::synchronous_sink** обеспечивает синхронный доступ к бекэнд, который позволяет использовать бекэнд в многопоточном приложении, даже если бекэнд не является потокобезопасным.

Разница между двумя фронтэндами **boost::log::sinks::asynchronous_sink** и **boost::log::sinks::synchronous_sink** заключается в том, последний не
основан на потоке. Записи лога передаются бекэнду в том же потоке.

[Пример 62.8](#example628) использует фронтэндов **boost::log::sinks::synchronous_sink**, бекэнд**boost::log::sinks::text_multifile_backend**. Бекэнд делает записи в логе и записывает их в один или несколько файлов. Имена файлов создаются в соответствии с правилом, принятым **set_file_name_composer()**
в бекэнде. Если вы используете свободно стоящие функции **boost::log::sinks::file::as_file_name_composer()**, как показано в примере, 
то правило может создаваться как лямбда-функция с той же структурой, используемой для функций форматирования. Однако атрибуты не используются
для создания строки, которая записывается в бекенж. Вместо этого строка будет именем файла, записи лога будут записаны.

[Пример 62.8](#example628) использует ключевые слова *channel* и *severity*, которые определяются с помощью макроса **BOOST_LOG_ATTRIBUTE_KEYWORD**. Они
ссылаются на атрибуты channel  и severity. Член-функцию **or_default()** вызывается на ключевые слова, чтобы передать значение по умолчанию,
если атрибут не задан. Если запись лога, channel  и severity не установлены, запись в файл `None-0.log`. Если запись журнала записывается
с уровнем журнала 1, он хранится в файле `None-1.log`. Если уровень записи лога 1 и канал называется Main, запись журнала сохраняется
в файле `Main-1.log`.

Атрибут канала определяется логгером **boost::log::sources::channel_logger**. Конструктор ожидает имя канала. Имя не может быть передано
непосредственно в виде строки. Вместо этого оно должно быть передано в качестве именованного параметра. Вот почему в примере используется
**keywords::channel = «Main»**, даже несмотря на то, что **boost::log::sources::channel_logger** не принимает каких-либо других параметров.

Пожалуйста, обратите внимание, что именованный параметр **boost::log::keywords::channel** не имеет ничего общего с ключевыми словами,
которые вы создаете с помощью макроса **BOOST_LOG_ATTRIBUTE_KEYWORD**.

**Boost::log::Sources::channel_logger** определяет записи журнала из различных компонентов программы. Компоненты могут использовать свои
собственные объекты типа **boost::log::sources::channel_logger**, давая им уникальные имена. Если компоненты только свои собственные
средства ведения журнала, это ясно, какой именно компонент записи конкретного журнала пришли из.

`Пример 62,9. Обработка исключений централизованно`
<a name="example629"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/logger.hpp>
#include <boost/log/utility/exception_handler.hpp>
#include <boost/log/exceptions.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <exception>

using namespace boost::log;

struct handler
{
  void operator()(const runtime_error &ex) const
  {
    std::cerr << "boost::log::runtime_error: " << ex.what() << '\n';
  }

  void operator()(const std::exception &ex) const
  {
    std::cerr << "std::exception: " << ex.what() << '\n';
  }
};

int main()
{
  typedef sinks::synchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);

  core::get()->add_sink(sink);
  core::get()->set_exception_handler(
    make_exception_handler<runtime_error, std::exception>(handler{}));

  sources::logger lg;

  BOOST_LOG(lg) << "note";
}
```
Boost.Log предоставляет возможность для обработки исключений в рамках записи лога централизованно. Это означает, что вам не нужно
обернуть каждый **BOOST_LOG** в блоке **try** для обработки исключений в улове.

[Пример 62.9](#example629) вызывает член-функцию **set_exception_handler()** . Ядро предоставляет эту лен-функцию для регистрации обработчика. Все
исключения в рамках записи лога будут передаваться в этот обработчик. Обработчик реализуется как объект функции. Он должен перегружать
**operator()** для каждого типа исключения, ожидаемого. Экземпляр этого объекта функции передается **set_exception_handler()** через
**boost::log::make_exception_handler()** шаблон функции. Все типы исключений, которые необходимо обработать должны передаваться в качестве
параметров шаблона для **boost::log::make_exception_handler()**.

Функция** boost::log::make_exception_suppressor()** сбрасывает все исключения в рамках ведения лога. Вы вызываете эту
функцию вместо **boost::log::make_exception_handler()**.

`Пример 62.10. Макрос, чтобы определить глобальные средства ведения журнала`
<a name="example6210"></a>
``` c++
#include <boost/log/common.hpp>
#include <boost/log/sinks.hpp>
#include <boost/log/sources/logger.hpp>
#include <boost/utility/empty_deleter.hpp>
#include <boost/shared_ptr.hpp>
#include <iostream>
#include <exception>

using namespace boost::log;

BOOST_LOG_INLINE_GLOBAL_LOGGER_DEFAULT(lg, sources::wlogger_mt)

int main()
{
  typedef sinks::synchronous_sink<sinks::text_ostream_backend> text_sink;
  boost::shared_ptr<text_sink> sink = boost::make_shared<text_sink>();

  boost::shared_ptr<std::ostream> stream{&std::clog,
    boost::empty_deleter{}};
  sink->locked_backend()->add_stream(stream);

  core::get()->add_sink(sink);

  BOOST_LOG(lg::get()) << L"note";
}
```
Все примеры в этой главе используют локальные логгеры. Если вы хотите определить глобальный логгер,
используйте макрос **BOOST_LOG_INLINE_GLOBAL_LOGGER_DEFAULT**, как в [примеру 62.10](#example6210). Передайте имя средства ведения журнала в качестве
первого параметра и тип в качестве второго. Средство ведения журнала не доступ через его имя. Вместо этого вы звоните **get()**, которая
возвращает указатель к singleton.

Boost.Log предоставляет дополнительные макросы, такие как **BOOST_LOG_INLINE_GLOBAL_LOGGER_CTOR_ARGS**. Они позволяют инициализировать
глобальные средства ведения журнала. **BOOST_LOG_INLINE_GLOBAL_LOGGER_CTOR_ARGS** позволяет передавать параметры конструктора глобального логгеры. Все эти макросы гарантируют, что будет правильно инициализированн глобальный логгер.

Boost.Log предоставляет много новых функций, которые стоит посмотреть. Например можно настроить инфраструктуру лога через
контейнер с парами ключ/значение в виде строк. Затем вам не нужно создавать экземпляры классов и вызова член-функции. Например можно
задать ключ назначения через консол, который автоматически создаст инфраструктуру лога используя **boost::log::sinks::text_ostream_backend** бекэнд. Бекэнд можно настроить через дополнительный ключ/значение пары. Поскольку
контейнер также может быть сериализован в INI-файле, можно сохранить конфигурации в текстовом файле и инициализировать инфраструктуру
лога из него.
