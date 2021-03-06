# Домашнее заание №4. Sedna.
## История развития СУБД.
Sedna была разработана в институте системного программирования(ИСП). Последний релиз был выпущен в 2012 году, а ссылку на более ранние версии недоступны. Судя по гиту проект начал развиваться в 2007 году под руководством Ivan Shcheklein. Коммитов не было с 2018 года. А как уже упомянал выше, последний релиз был в 2012 году.

## Инструменты для взаимодействия с СУБД.
У Sedna есть драйверы для языков C, C++, Java, Python, Scheme, C#, PHP, Ruby, Haskell, Catalyst, Perl, Delphi and Pascal. Также иммется XQJ and XML:DB drivers для Java. Некоторые из них уже не поддерживаются.

## Какой database engine используется в вашей СУБД?
В Sedna используется Native XML DBMS. В отличие от СУБД, которые поддерживают XML формат, но используют реляционную модель данных, они могут представлять иерархические данные, понимают встроенные объявления PCDATA в элементах XML и поддерживают специфические для XML языки запросов, такие как XPath, XQuery или XSLT.

## Как устроен язык запросов в вашей СУБД? Разверните БД с данными и выполните ряд запросов. 
Для установки Sedna достаточно скачать и разархивировать архив.  
Чтобы запустить Sedna сервер нужно зайти в папку INSTALL_DIR/bin и выполнить:  
``.\se_gov.exe``  
Для остановки:  
``.\se_stop.exe``  
Чтобы создать базу данных нужно выполнить:  
``.\se_cdb.exe testdb``  
Чтобы запустить базу данных нужно выполнить команду:  
``.\se_sm testdb``  
Для остановки:  
``.\se_smsd testdb``  
Язык запросов СУБД - XQuery 1.0(w3.org/TR/2010/REC-xquery-20101214/). Давайте заполним датасет данными. Чтобы добавить существующий заполненный документ используется следующая команда:  
``LOAD "path_to_file" "document_name"``  
Я буду использовать готовую [демобазу](auction.xml), которая есть в примерах. 
Если же мы хотим создать пустой документ и заполнить его и обновлять данные, то используется следующий синтаксис:  
``CREATE DOCUMENT "hospital"& // создать документ hospital``  
``UPDATE INSERT <root><peace/></root> INTO fn:doc("hospital")& \\ кладём выражение ``  
``UPDATE INSERT <warning>High Blood Pressure!</warning> INTO fn:doc("hospital")/root/peace``  
Давайте попробуем сделать запрос к базе. Например, давайте посчитаем количество товаров, проданных по цене больше 39.  
``count(for $i in doc("auction")/site/closed_auctions/closed_auction``  
``        where  $i/price/text() >= 40``   
``        return $i/price)``  
Как мы видим, мы проходим по всем проданным товарам в первой строчке, во второй отбираем сколько стоят больше 39 и в третьей строчке возвращаем нужное.
Давайте теперь посчитаем, сколько товаров представлено в данном документе?  
``for    $b in doc("auction")/site/regions``  
``return count ($b//item)``

## Распределение файлов БД по разным носителям?
На официальном сайте информации по этому вопросу не нашёл, но нашёл презентацию ИСПа по memory managment in Sedna.(https://www.slideshare.net/shcheklein/sedna-xml-database-memory-management). Сайт лагучий, презентация иногда не подгружается.
![mem man](https://i.imgur.com/sMDrYA8.png)
![mem man2](https://i.imgur.com/z1lpW8Z.png)

Из презентации ясно, что DAS указатели ускорили Sedna на 1,5%.  
Также интерсено обратить внимание на то, как хранится документ, благодаря ообой структуре хранения, переход между элементами осуществляется инкрементом указателя.

![storage example](https://i.imgur.com/lwMQqco.png)  
А ещё общие идеи управлением памятью заключаются в том, что, во-первых, содержимое блоков базы данных не изменяется при перемещении блока из внешней в основную память. Во-вторых, для представления связей между узлами XML -документа почти всегда используются прямые указатели, которые можно использовать для перехода от одного узла к другому сразу после перемещения блока базы данных в основную память. При этом потенциальный размер базы данных не ограничивается.  
Сами файлы баз данных хранятся в *SEDNA_INSTALL/data* данные базы данных, хранящиеся в подкаталогах с именем ``<db_name>_files``, где ``<db_name>`` это имя соответствующей базы данных  

## На каком языке/ах программирования написана СУБД?
Судя по гиту большая часть проекта написана на **C++/C**, парсер написан на **Yacc**. Также немного, но использовался **Scheme** и **Java**.
![languages](https://i.imgur.com/HdNpoH8.png)

## Какие типы индексов поддерживаются в БД? Приведите пример создания индексов.
Sedna поддерживает value indexes для индексации содержимого элементов XML и значений атрибутов. Реализуются на B+ tree или BST(Block String Trie).  
Чтобы создать индекс используется команда:  
``CREATE INDEX title-expr``  
``ON Path1 BY Path2``  
``AS type``
``[USING tree-type]``  
Создаёт индекс под названием ``title-expr`` на ноде ``Path1`` по ключу ``Path2``. type - это атомарный тип, к которому должно быть приведено значение ключей. Для B-дерева поддерживаются следующие типы: xs:string, xs:integer, xs:float, xs:double, xs:date, xs:time, xs:dateTime, xs_yearMonthDuration, xs_dateTimeDuration. BST поддерживает только xs:string.
Пример создания:  
``CREATE INDEX "people"``  
``ON doc("auction")/site//person BY address/city``  
``AS xs:string``  
Но в Sedna индексы не используются по умолчанию, роэтому нужно писать явно, если хотим ими воспользоваться:  
``index-scan("people", "London", "EQ")/name``   
Выбирем имена людей, которые живут в городе Лондон, используя индекс "people", который был создан выше.  
Также поддерживаются full-text indexes:  
``CREATE FULL-TEXT INDEX title-expr``  
``ON path``
``TYPE type``  
``[``  
``WITH OPTIONS options``  
``]``  
Пример создания full-text index.  
``CREATE FULL-TEXT INDEX "articles"``  
``ON doc("foo")/library//article``  
``TYPE "xml"``

## Как строится процесс выполнения запросов в вашей СУБД?
Парсер преобразует запрос в логическое представление, представляющее собой дерево операций, близких к тем, которые специфицированы в ядре XQuery (XQuery Core). Оптимизатор получает логическое представление запроса и производит план выполнения запроса, дерево низкоуровневых операций над физическими структурами данных. План выполнения интерпретируется исполнителем, который взаимодействует с соответствующим экземпляром менеджера базы данных.

## Есть ли для вашей СУБД понятие «план запросов»? Если да, объясните, как работает данный этап.
Из предыдщего пункта ясно, что оптимизатор производит план запросов. Давайте поговорим об этой стадии. Известные методы оптимизации запроса:  
* Метод **открытой подстановки тел функций** позволяет заменить вызов определенной пользователем функции на ее тело. Открытая подстановка устраняет накладные расходы, порождаемые при вызове функции, и дает возможность совместной статической оптимизации тела функции совместно с общим кодом запроса. Реализованный алгоритм открытой подстановки корректно обрабатывает вызовы не рекурсивных и структурно рекурсивных функций и разумным образом завершается при обработке вызовов произвольных рекурсивных функций.
* Метод **выдавливания** предикатов ниже конструкторов элементов XML позволяет изменить порядок операций таким образом, чтобы предикаты применялись до выполнения конструкторов XML-элементов. Это позволяет уменьшить размер промежуточных результатов, к которым применяются конструкторы.
* При применении метода **проекции преобразования** выражение XPath статически применяется к конструкторам элементов. Это позволяет избежать избыточных вычислений в конструкторах XML-элементов на стадии выполнения запроса.
* Метод **упрощения запроса** на основе использования схемы оказывается полезным в тех случаях, когда схема XML-документа доступна оптимизатору, а запрос сформулирован пользователем, не имеющим отчетливого представления об этой схеме. Получение более точной формулировки запроса позволяет избежать избыточного сканирования данных, свойственного запросам, при формулировке которых не учитывалась схема. Этот оптимизационный метод базируется на статическом выводе типов данных в языке XQuery.
* Метод **повышения уровня декларативности формулировки запроса** дает возможность расширить пространство поиска оптимизатора при выборе оптимального плана выполнения запроса. Метод является адаптацией аналогичного SQL-ориентированного метода, позволяющего преобразовывать SQL-запросы со вложенными подзапросами в эквивалентные запросы с соединениями.
* **Нормализация предикатов соединения** состоит в их приведении в конъюнктивную нормальную форму, чтобы можно было использовать различные алгоритмы выполнения операции соединения, а не только алгоритм вложенных циклов. Для достижения этой цели в оптимизаторе СУБД Sedna из предикатов соединения извлекаются подвыражения, подобные операторам XPath, и помещаются вне операций соединения, где они вычисляются только один раз.
* **Метод распознавания инвариантных подвыражений** в теле итерационных операций и вынесения их за пределы этих операций позволяет уменьшить вычислительную сложность запроса.
* Для обеспечения сериализуемости транзакций применяется вариант известного строгого двухфазного протокола синхронизационных блокировок (2 PL). В текущей версии системы единицей блокировки является XML-документ целиком. Однако во многих случаях блокировка всего XML-документа не требуется и приводит к уменьшению уровня параллельности. Поэтому разрабатывается новый метод *гранулированных* блокировок, позволяющий поддерживать высокий уровень параллельности.

## Поддерживаются ли транзакции в вашей СУБД? Если да, то расскажите о нем. Если нет, то существует ли альтернатива?
Режим фиксации соединения по умолчанию - auto-commit, что означает, что все обновления будут фиксироваться автоматически. Если такое поведение нежелательно, мы можем передать параметр manual-commit в sql:connect при создании хэндла соединения.  
В режиме ручной фиксации мы можем указать, когда обновления будут закомичены или произойдёт откат:
![transaction](https://i.imgur.com/fxh91fF.png)
Транзакции запускаются автоматически запросами, существуют две функции для указания границ транзакций - sql:commit и sql:rollback:
![commit](https://i.imgur.com/7DIBGFJ.png)
Функция sql:commit фиксирует все изменения, сделанные во время последней транзакции в соединении базы данных, указанном хэндлом соединения $connection, и закрывает транзакцию. Если операция не может быть завершена, выдается ошибка.
![rollback](https://i.imgur.com/6sAe4aK.png)
Функция sql:rollback откатывает все изменения, сделанные во время последней транзакции в соединении базы данных, указанном хэндлом соединения $connection, и закрывает транзакцию. Если операция не может быть завершена, возникает ошибка.

## Какие методы восстановления поддерживаются в вашей СУБД. Расскажите о них.
* Как мы заметили выше, поддерживается процесс восстановления отката данных в транзакции.  
* Также существует утилита ``se_exp``, назначение которой обеспечить функциональность экспорта/импорта данных. Идея метода se_exp заключается в генерации набора XML-файлов и сценариев XQuery для восстановления базы данных в том же состоянии, в котором она находилась на момент экспорта.
* Альтернативной стратегией резервного копирования базы данных является прямое копирование каталогов, которые Sedna использует для хранения данных базы данных(*SEDNA_INSTALL/data*). Чтобы восстановить базу данных, скопируйте соответствующий каталог резервного копирования в место, где Sedna хранит базы данных.
* Другой альтернативой является резервное копирование базы данных во время ее работы. Такая процедура называется горячим резервным копированием. Ее цель - создать согласованную резервную копию, пока пользователи еще выполняют некоторые запросы. Затем эту копию можно восстановить, скопировав соответствующий каталог резервной копии в каталог, где Sedna хранит базы данных. Такое горячее резервное копирование может выполняться в инкрементном режиме, что позволяет более эффективно архивировать изменения баз данных.
В двух словах, при горячем резервном копировании Sedna создает копии всех файлов базы данных, необходимых для восстановления стабильного состояния базы данных в случае сбоя. Основное различие между резервным копированием на уровне файловой системы выше и горячим резервным копированием заключается в том, что целевая база данных не должна быть остановлена. В качестве компромисса, восстановление из горячей резервной копии может быть более медленным процессом, в зависимости от давности копии. Используется утилита ``se_hb``.

## Расскажите про шардинг в вашей конкретной СУБД. Какие типы используются? Принцип работы.
Как видно из картинки, партицирование не реализовано в Sedna.
![sharding](https://i.imgur.com/1eL9wM1.png)

## Возможно ли применить термины Data Mining, Data Warehousing и OLAP в вашей СУБД?
Для Data Mining нет никаких инструментов. 

Data Warehousing - корпоративное хранилище данных отличается от обычных БД, используемых в бизнесе, по нескольким параметрам:

* Тип и источник данных. DWH консолидирует в себе информацию от всех департаментов компании — от статистики продаж до сведений о сотрудниках.
* Объем данных. В Data Warehouse стекаются исторические данные и архивные сведения.
* Роль в бизнес-процессах. Data Warehouse всегда содержит последние версии данных.

Sedna вполне удолетворяет всем трём параметрам, так что да, можно применить это слово к ней.

Также и OLAP можно применить, так как XQuery обладает достаточной возможностью для аналитической обработки в реальном времени.

## Какие методы защиты поддерживаются вашей СУБД? Шифрование трафика, модели авторизации и т.п.
Для начала СУБД поддерживает понятие users and roles.
Юзеры могут быть двух типов:
* Администратор базы данных (пользователь DBA);
* Обычный пользователь - пользователь.  

Возможнсти администратора:
* имеет все возможные привилегии на любой объект в базе данных;
* может удалить любой объект в базе данных;
* может удалить любого пользователя базы данных;
* может предоставлять/отменять любые привилегии любому пользователю базы данных;
* может предоставить пользователю роль "DBA", тем самым делая этого пользователя также пользователем DBA. Любой пользователь DBA также может отозвать роль "DBA" у любого пользователя DBA.

Возможности пользователя:
* может действовать в соответствии с привилегиями, которые у него есть;
* может предоставлять и отзывать любые привилегии на объекте базы данных, которым он владеет, любому пользователю;
* может удалять объекты базы данных, которыми он владеет, и удалять пользователей, которых он создал.

Каждый пользователь имеет свое имя и пароль. Дальше прелставлены таблицы привелегий и к чему они могут быть применены:
![privaliges](https://i.imgur.com/pc3Zjd7.png)
![usage](https://i.imgur.com/pYYxlbO.png)

Когда клиентское приложение подключается к серверу базы данных, оно указывает, под каким именем пользователя базы данных Sedna оно хочет подключиться. Имя пользователя определяет привилегии доступа к объектам базы данных, поэтому аутентификация клиента используется для ограничения того, какие пользователи базы данных могут подключаться.  
При создании с опцией -db-security authorization или -db-security authentication база данных будет проверять пароль пользователя при открытии сессии.
В настоящее время Sedna использует парольную аутентификацию: клиентское приложение, подключающееся к базе данных, должно указать имя пользователя и пароль пользователя, т.е. в утилите se_term используйте опции -pswd и -name.  
Остальной защиты нет.

## Какие сообщества развивают данную СУБД? Кто в проекте имеет права на коммит и создание дистрибутива версий? Расскажите об этих людей и/или компаниях.
В настоящее время последний релиз датируется 2012 годом, а последний коммит датируется 2018 годом. Проект больше не развивается судя по этим данным. Также проверил форки репозитория. По ним никаких подвижек нет, только один парень добавил свой пример(https://github.com/ChaosReadman/sedna). Больше всех в проект коммитил Ivan Shcheklein. А также Ilya Taranov, Alexander Kalinin, Oleg Borisenko. Под дистрибутивом на sourceforge отмечены Ivan Shcheklein и Ilya Taranov.

### Ivan Shcheklein.

Работал в ИСПе, сейчас переехал в Сан Франциско. Co-founder and CTO at Iterative - Версионирование дата-сетов и моделей машинного обучения.

### Ilya Taranov.

В 2009 году закончил аспирантуру на кафедре ИСПа. Информации больше о нём найти не смог.

### Alexander Kalinin.

Также работал на ИСП. Перебрался в Америку. Работает в Brown University.

### Oleg Borisenk.

Выпускник МГУ, с 2011 года работает в ИСПе.

## Как продолжить самостоятельное изучение языка запросов с помощью демобазы. Если демобазы нет, то создайте ее.
[Демобаза есть](auction.xml). Находся в разархивированном архиве в кладке **SEDNA_INSTALL/examples/commandline**. Для дальнейшего изучения там есть примеры разных запросов. Также следует в деталях изучить запросов XQuery, изучить full-text запросы, external functions, разобраться как работать с sql. Почитать более детально документацию, попробовать сделать свои триггеры. Попробовать воспользоваться API для какого-то языка программирования. Ну и в конечном счёте, написать какое-нибудь приложение на этой СУБД.

## Где найти документацию и пройти обучение.
Докумаентация для [программистов](https://www.sedna.org/progguide/ProgGuide.html) и для [администрирования](https://www.sedna.org/adminguide/AdminGuide.html). Пройти [обучение](https://www.w3schools.com/xml/xquery_intro.asp) языка XQuerry можно по гиперссылки, посмотреть работы с СУБД с помощью командной строки можно в директории *SEDNA_INSTALL/examples/commandline*, с помощью API в *SEDNA_INSTALL/examples/api/language_name*.

## Как быть в курсе происходящего?
Никак, проект выглядит мёртвым.
