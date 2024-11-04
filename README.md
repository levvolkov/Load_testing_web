# Домашнее задание к лекции 3. «Проведение нагрузочного тестирования WEB» 

[Задание.](https://github.com/netology-code/loadqa-homeworks/blob/main/3.Load%20web/homework_lecture3.md)

### Сделано:

## Cреда нагрузочного тестирования:
1. Зарегистрирован на сайте [BlazeMeter](https://www.blazemeter.com/).
2. Установлено [расширение BlazeMeter](https://chromewebstore.google.com/detail/mbopgmdnpcbohhpnfglgohlbhfongabi?hl=ru&utm_source=ext_sidebar) для браузера.
3. Установлен [JMeter](https://github.com/netology-code/iqa-homeworks/blob/iqa-64/Instruction.md) для настройки и выполнения нагрузочных тестов.
  * Настроено автосохранение для автоматического сохранения тестового сценария:
    * Меню: `Option => Save Automatically before run => ✅`
  * Установлен [Plugins Manager](https://jmeter-plugins.org/wiki/PluginsManager/), переместив `.jar` файл в директорию Apache JMeter с помощью команды:
```bash
   mv ~/Downloads/jmeter-plugins-manager-1.10.jar /Applications/apache-jmeter-5.6.3/lib/ext/
```

-----

<br>

## Раунд тестирования добавления комментария на странице WordPress

#### 1. Подготовка расширения BlazeMeter
  * Для начала тестирования перешел на сайт [WordPress](https://cw24054-wordpress-zu0z0.tw1.ru/) и запустил расширение BlazeMeter. В процессе записи шагов нагрузочного тестирования были выполнены следующие настройки:
    * Назначено название теста.
    * Настройки в разделе Advanced options:
      * Filter Pattern:  `http://*/*, https://*/*` (выбор отслеживаемгог протокола).
      * ☑️ Disable Browser Cache (отключение кэша браузера для отправки всех запросов на удаленный сервер WordPress).
      * ☑️ Update Settings Before Running Test (настройка теста по количеству пользователей и виртуальных запросов).
      * 🔘 Only Top Level Request (запись основных запросов без дополнительных запросов к CSS и JavaScript).

#### 2. Запись тестового сценария
  * Запустил сценарий записи в расширении BlazeMeter и выполнил следующие действия:
    * Обновил главную страницу WordPress.
    * Перешел к тестовой записи "Привет, мир!".
    * Добавил комментарий с указанными данными:
      * Комментарий
      * Имя
      * Email
    * Остановил запись в BlazeMeter, тест был собран и перенаправлен на основную страницу сайта BlazeMeter.
  
#### 3. Настройка тестового сценария
  * На странице BlazeMeter были настроены основные параметры в области производительности (LOAD CONFIGURATION):
    * Total Users - количество виртуальных пользователей (количество виртуальных потоков, которые будут отправлять параллельные запросы).
    * Duration (min) ↔️ Iterations - продолжительность нагрузки системы (Duration) или количество итераций, сколько раз будет прогоняться тест (Iterations).
    * Ramp Up Time (min) - периодичность добавления пользователей (потоков) в систему (активно при количестве пользователей более одного).
    * Выбрано облако для выполнения нагрузки (LOAD DISTRIBUTION).
  * Запущен тест (Run Test).
    * Сценарий запуска теста завершился с открытием вкладки Summary, которая содержит основную сводку.
    * Также были сделаны [скриншоты](Screenshot-WordPress) результатов нагрузочного тестирования.
  * Для скачивания сценария в формате jmx:
    * Прешел на вкладку **Original Test Configuration => Files => название_теста.jmx ☁️**.

#### 4. Подготовка Jmeter
  * Скачанный файл `WordPress_COMMENT_TEST.jmx` был перемещен в директорию для хранения тестов производительности и запущен с помощью JMeter для нагрузки на веб-приложение.
```bash
   # Созданы папки хранения проекта:
   mkdir -p ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress

   # Перемещение файла .jmx в директорию:
   mv ~/Downloads/WordPress_COMMENT_TEST.jmx ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress

   # Переход в директорию:
   cd ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress

   #Открытие теста в JMeter:
   jmeter -t WordPress_COMMENT_TEST.jmx
```
  * Названия шагов в вкладке [Thread Group](https://jmeter.apache.org/usermanual/component_reference.html#Thread_Group) были изменены на более понятные.
  * Убрана галочка с Retrieve All Embedded Resources в разделе [HTTP Request Defaults](https://jmeter.apache.org/usermanual/component_reference.html#HTTP_Request_Defaults), чтобы исключить лишние запросы и ускорить поиск нужного запроса на комментарий.
  * В `Thread Group` был добавлен слушатель [View Results Tree](https://jmeter.apache.org/usermanual/component_reference.html#View_Results_Tree), что позволяет детально отслеживать и анализировать выполнение тестовых запросов:
    * По пути: Add => Listiner => View Results Tree
  * Чтобы не было дублировании коментария при повторном запуске теста, что приводит к ошибке в `Thread Group` добавлен еще oдин элемент [Random Variable](https://jmeter.apache.org/usermanual/component_reference.html#Random_Variable) спосбный автоматичеки создавать комментарий рандомно.
    * По пути: Add => Config Elements => Random Variable 
    * Создан элемент по маске расположенной в [документации](https://jmeter.apache.org/usermanual/component_reference.html#Random_Variable), который каждый раз при запуске сценария будет генерировать рандомное значение комментария (comment_0000). Cоздана и скопировано название переменной, после чего указана в сценарии тестового шага `${VARIABLE_COMMENT}`, вместо комментария.
    * Также рассмотрен более простой способ создания [рандомной строки](https://jmeter.apache.org/usermanual/functions.html#__RandomString) без необходимости создания Random Variable. Достаточно указать `${__RandomString(10,abcdefg)}` на месте комментария в тестовом сценарии, что генерирует случайное значение из 10 указанных букв.
  * В запущенном и выполненом тесте убедился в исполнении всех шагов, применено форматирование результатов тестирования с выбором формата HTML (Text ↔️ HTML).
    

#### 5. Генерация стандартного отчета о проведенном тестировани Apache JMeter Dashboard
  * Отчет Apache JMeter Dashboard представляет собой визуализированную сводку результатов тестирования производительности, включая графики нагрузки, статистику времени отклика и процентильные данные, что позволяет эффективно анализировать поведение системы под нагрузкой.
```bash
   # Сгенерирован стандартный JMeter отчет:
   jmeter -n -t ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress/WordPress_COMMENT_TEST.jmx -l ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress/test_results.jtl -e -o ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress/report_output

   # Переход в нужную директорию и открытие стандартного отчета jmeter о проведенном тестировании  (Apache JMeter Dashboard)
   cd ~/Documents/Performance_testingQA79/Load_testing_web/Test-WordPress && open report_output/index.html
```

-----

<br>

##  Раунд тестирования покупки билета и получение QR кода на сайте ИДЕМ В КИНО

#### 1. Запуск сайта кинотеатра "Идем в кино"
  * В процессе выполнения задач по тестированию производительности была проведена настройка проекта кинотеатра, которая включает в себя [клонирование репозитория с GitHub](https://github.com/mshegolev/congenial-potato/).
    * Команды для подготовки и запуска проекта:
```bash
   # Создана папка для хранения проекта для проведения нагрузочного тестирования:
   mkdir ~/Documents/Performance_testingQA79/Load_testing_web/Test-Cinema

   # Переход в директорию для хранения проекта:
   cd ~/Documents/Performance_testingQA79/Load_testing_web/Test-Cinema

   # Клонирование репозитория проекта:
   git clone https://github.com/mshegolev/congenial-potato.git

   # Переход в каталог кинотеатра и открытие папки cinema в Visual Studio Code:
   cd congenial-potato/cinema && code .
```

  * В файл `docker-compose.yml` добавлена строка `platform: linux/x86_64`, чтобы указать Docker на создание образов для архитектуры x86_64 (необходимо для работы на Apple M1 с архитектурой ARM).
  * Чтобы избежать предупреждения при запуске контейнеров в `docker-compose.yml` удалена строка `version: '3.7'`.
    * В версиях Docker Compose 2.0 и выше больше не требуется указывать версию файла, так как композитор теперь автоматически обрабатывает данные. Это упрощает работу и делает файлы конфигурации более понятными.
  * Запущены контейнеры с помощью Docker и проверено их состояние:
```bash
   # Запуск контейнеров Docker в фоновом режиме
   docker-compose up -d

   # Проверка запущенных контейнеров
   docker ps
```

#### 2. Подготовка расширения BlazeMeter
  * В браузере по адресу `localhost:8000` указанном в `docker-compose.uml` запущено расширение BlazeMater.
  * Для записи шагов нагрузочного тестирования:
    * Назначено название теста.
    * Настройки в разделе Advanced options:
      * Filter Pattern:  `http://*/*, https://*/*` (выбор отслеживаемгог протокола).
      * ☑️ Record Ajax Request (для захвата и записи AJAX-запросов, которые отправляются вашим веб-приложением во время взаимодействия с ним, особенно полезно при тестировании веб-приложений, использующих динамические загрузки данных через AJAX)
      * 🔘 Only Top Level Request (запись основных запросов без дополнительных запросов к CSS и JavaScript).

#### 3. Запись тестового сценария:
  
  * Запущен сценарий записи в расширении BlazeMeter для воспроизведения шагов:
    * Переход на страницу (обновлена главная страница)
    * Выбор сеанса
    * Выбор места
    * Получить код бронирования
    * Получен код бронирования
  * Запись в BlazeMeter остановлена и тест собран, перенаправлен на основную страницу сайта BlazeMeter.
   
#### 4. Настройка тестового сценария
  * На странице BlazeMeter настроены основные параметры в области производительности (LOAD CONFIGURATION):
    * Total Users - количество виртуальных пользователей (количество виртуальных потоков, которые будут отправлять параллельные запросы).
    * Duration (min) ↔️ Iterations - продолжительность нагрузки системы (Duration) или количество итераций, сколько раз будет прогоняться тест (Iterations).
    * Ramp Up Time (min) - с какой периодичностью пользователи (потоки) будут добавляться в систему (работает, если пользователей больше одного).
    * Выбрано облако для выполнения нагрузки (LOAD DISTRIBUTION).
  * Запущен тест (Run Test).
  * Сценарий запуска теста в расширении BlazeMeter завершился, открылась вкладка Summary, которая показывает основную сводку.
  * Для скачивания сценария в формате jmx:
    * Прешел на вкладку Original Test Configuration => Files => название_теста.jmx ☁️.

#### 5. Подготовка Apache JMeter
  * Скаченный файл `CinemaTest.jmx` перемещен в директорию для хранения тестов производительности и запущен с помощью JMeter для проведения нагрузки на веб-приложение.
```bash
   # Перемещение файла .jmx в директорию по тестированию произовдительности:
   mv ~/Downloads/CinemaTest.jmx ~/Documents/Performance_testingQA79/Load_testing_web/Test-Cinema

   # Переход в директорию:
   cd ~/Documents/Performance_testingQA79/Load_testing_web/Test-Cinema

   #Открытие теста в JMeter
   jmeter -t CinemaTest.jmx
```
  * В `Thread Group` добавлен слушатель [View Results Tree](https://jmeter.apache.org/usermanual/component_reference.html#View_Results_Tree) ,один из наиболее популярных и полезных инструментов, который позволяет пользователям детально отслеживать и анализировать выполнение тестовых запросов.
    * По пути: Add => Listiner => View Results Tree
  * Названия шагов расположенные во кладке [Thread Group](https://jmeter.apache.org/usermanual/component_reference.html#Thread_Group) изменены на более понятные.
    
    * В шаг который отвечает за выбор ряда и места добавлено случайное значение созданое  по маске расположенной в [документации](https://jmeter.apache.org/usermanual/functions.html#__Random), которое каждый раз при запуске сценария будет генерировать рандомное значение:
      * salesPlaces = `[{"row":${_Random(1, 10)}, "place":${__Random(1,10)}, "type":"standart"}]`
  * В запущенном и выполненом тесте убедился в исполнении всех шагов, применено форматирование результатов тестирования с выбором формата HTML(Download resources).
   
#### 6. Генерация стандартного отчета о проведенном тестировани Apache JMeter Dashboard
  * Отчет Apache JMeter Dashboard представляет собой визуализированную сводку результатов тестирования производительности, включая графики нагрузки, статистику времени отклика и процентильные данные, что позволяет эффективно анализировать поведение системы под нагрузкой.
```bash
   # Сгенерирован стандартный JMeter отчет:
   jmeter -n -t ~/Documents/Performance_testingQA79/Load_testing_web/CinemaTest.jmx -l ~/Documents/Performance_testingQA79/Load_testing_web/test_results.jtl -e -o ~/Documents/Performance_testingQA79/Load_testing_web/report_output

   # Переход в нужную директорию и открытие стандартного отчета jmeter о проведенном тестировании  (Apache JMeter Dashboard)
   cd ~/Documents/Performance_testingQA79/Load_testing_web && open report_output/index.html
```

#### 7. По результатам проведения нагрузочного тестировании сделаны [скришоты](Screenshot-Cinema).

<br>

---------------   

### Дополнительная информация
- https://www.blazemeter.com/ - инструкция по работе с `blazemeter`;
- [Blazemeter chrome extention](https://chrome.google.com/webstore/detail/blazemeter-the-continuous/mbopgmdnpcbohhpnfglgohlbhfongabi) - расширение Chrome browser для записи тестов c помощью `blazemeter`
- https://jmeter.apache.org/ - инструкция по работе с `jmeter`;
- [Jmeter Test Script Recorder](https://jmeter.apache.org/usermanual/jmeter_proxy_step_by_step.html) - инструкция по записи тестов с помощью `jmeter`
- [Download jmeter](https://jmeter.apache.org/download_jmeter.cgi) - дистрибутивы `jmeter`
- [Install plugin](https://jmeter-plugins.org/wiki/PluginsManager/) - установка плагинов в `jmeter`
