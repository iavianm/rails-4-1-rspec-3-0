# **Everyday Rails Testing with RSpec**

>автор Aaron Sumner
*опубликована 20-12-2014*

Почитать отзывы о книге можно в твиттере по этому [адресу](https://twitter.com/search?q=#everydayrailsrspec)
* * *
## Настройка учебного материала

Оригинальный репозиторий материала книги находится [здесь](https://github.com/everydayrails/rails-4-1-rspec-3-0)

Локально склонировать оригинал
`git clone git@github.com:everydayrails/rails-4-1-rspec-3-0.git`

Настройка для push'а в свой репозиторий
`git remote set-url --push origin git@github.com:iavianm/rails-4-1-rspec-3-0.git`

Переключиться на ветку второй главы
`git checkout -b 02_setup origin/02_setup`
* * *
## **2 глава - Настройка RSpec**
## Gemfile
>Поскольку ***RSpec*** не включен в приложение Rails по умолчанию, надо установить его и вспомогательные GEM'ы.
- Gemfile
```ruby
group:development, :test do
	gem"rspec-rails","~> 3.1.0"
	gem"factory_girl_rails","~> 4.4.1"
end

group:test do
	gem"faker","~> 1.4.3"
	gem"capybara","~> 2.4.3"
	gem"database_cleaner","~> 1.3.0"
	gem"launchy","~> 2.4.2"
	gem"selenium-webdriver","~> 2.43.0"
end
```

>Gem `factory_girl_rails` переименован в [Factory_bot_rails](https://github.com/thoughtbot/factory_bot_rails)

## Зачем устанавливать в две отдельные группы?
`rspec-rails` и `factory_girl_rails` используются в двух средах: `development` и `test`
Остальные гемы для тестирования нашего приложения. только в среде: `test`
>Если устанавливать gem'ы из командной строки, они сами установятся в нужные группы
>*Пример:* `$ gem install factory_bot_rails`

1. **[Rspec-rails](https://github.com/rspec/rspec-rails)** включает сам RSpec в приложение.
2. **[Factory_girl_rails](https://github.com/thoughtbot/factory_bot_rails)** заменяет стандартные Rails [fixtures](https://api.rubyonrails.org/v3.1/classes/ActiveRecord/Fixtures.html) на [фабрики](https://github.com/thoughtbot/factory_bot_rails#factory_bot_rails---) для заполнения тестовых данных.
3. **[Faker](https://github.com/faker-ruby/faker)** генерирует имена, адреса электронной почты и другие данные для фабрик.
4. **[Capybara](http://teamcapybara.github.io/capybara/)** позволяет легко программно моделировать взаимодействие ваших пользователей
5. **[Database_cleaner](https://github.com/DatabaseCleaner/database_cleaner)** очищает данные из тестовой базы.
6. **[Launchy](https://github.com/copiousfreetime/launchy)** открывает веб-браузер по умолчанию по требованию, для отображения, что выводит на экран приложение.
7. **[Selenium-webdriver](https://github.com/SeleniumHQ/selenium/)** позволяет протестировать взаимодействие браузера на основе JavaScript с **Capybara**
* * *
## **Тестовая база данных**
>Перед тестированием надо убедиться, что есть тестовая база данных

Откроем `config/database.yml`, должны увидеть примерно такую картину:

**Для SQLite:**
`config/database.yml`
```yml
test:
  adapter: sqlite3
  database: db/test.sqlite3
  pool: 5
  timeout: 5000
```

**Для MySQL:**
`config/database.yml`
```yml
test:
  adapter: mysql2
  encoding: utf8
  reconnect: false
  database: contacts_test
  pool: 5
  username: root
  password:
  socket: /tmp/mysql.sock
```

**Для PostgreSQL:**
`config/database.yml`
```yml
test:
  adapter: postgresql
  encoding: utf8
  database: contacts_test
  pool: 5
  username: root # or your system username
  password:
```

Запускаем `$bin/rake db:create:all` - создадуться базы окружений
* * *
## **Конфигурация RSpec**
> После создания базы данных, можно приступать к настройке RSpec

Запустим - `$ bin/rails generate rspec:install`
>Должны получить ответ
```
create .rspec
create spec
create spec/spec_helper.rb
create spec/rails_helper.rb
```

- `.rspec` — Конфигурационный файл RSpec
- `spec` — Директория для тестов
- `spec/spec_helper.rb` и `spec/rails_helper.rb` — хелперы для взаимодействия с кодом. В этих файлах много закоменченного кода с обьяснением возможных настроек. Сейчас сильно вчитыватся в них ненадо, ознакомимся с ними позже.

>Для вывода результатов тестов в удобном читаемом формате добавим в файл `.rspec`

1. `--format documentation` - форматирование вывода
2. `--warning` - все предупреждения
* * *
## **Генераторы**
>Необязательный шаг настройки: 
>При использовании `scaffold`  сгенерируется много тестов, которые нам не будут нужны, например `views/spec`.
Можно вручную указать для каких частей приложения генерировать тесты

`config/application.rb`
 ```ruby
config.generators do |g|
  g.test_framework :rspec,
    fixtures: true,
    view_specs: false,
    helper_specs: false,
    routing_specs: false,
    controller_specs: false,
    request_specs: false
  g.fixture_replacement :factory_bot, dir: "spec/factories"
end
```

- `fixtures: true` - создание фикстур для каждой модели
- `view_specs: false` - пропускать генерирование тестов для вьюх. Мы не будем их тестировать в этой книге, вместо этого мы будем писать тесты для элементов интерфейса.
- `helper_specs: false` - пропускать генерирование тестов для хелперов. По мере роста уровня вашего мастерства в работе с RSpec поменйте значение на `true`, что бы тестировать эти файлы.
- `routing_specs: false` - пропускает тесты для _config/routes.rb_. Учебное приложение достаточно простое и можно не писать тесты на раутинг. Но по мере роста приложения и появления большого количества путей рекомендуется включать тестирование маршрутизации.
- `request_specs: false` - пропускает создание дефолтных RSpec тестов интеграцонного уровня в _spec/request_. Мы займемся ручным написанием таких тестов в 8 главе.
- `g.fixture_replacement :factory_girl` - говорит "рельсам" генерить фабрики вместо фикстур, и сохранять их в _spec/factories directory_.

>Не забывайте! Если RSpec не генерит вам нужные файлы, вы всегда можете их написать руками. Необходимо следовать [соглашению](https://relishapp.com/rspec/rspec-rails/docs/directory-structure) для расположения и названия файлов.
* * *
## **Применение вашей схемы базы данных для тестирования**
> Для версии Rails до версии 4.1 необходимо в ручную создать базу данных для тестов `rake db:test:prepare` или `rake db:test:clone`
- Первый запуск

```
$ bin/rspec`

No examples found.

Finished in 0.00027 seconds(files took 0.06527 seconds to load)
0 examples, 0 failures
```