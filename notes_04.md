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

Переключиться на ветку 4 главы
`git checkout -b 04_factories origin/04_factories`
* * *
## **4 глава - Генерация тестовых данных с помощью фабрик** 

Будем тестировать более сложные сценарии, для упрощения этого  процесса и чтобы сосредоточиться на тестах, а не на заполнении данных, в этой главе рассмотрим "Factory Girl" ('[factory_boot_rails](https://github.com/thoughtbot/factory_bot_rails#factory_bot_rails---)') - выбор многих разработчиков.

**А именно:**
- Поговорим о преимуществах и недостатках использования фабрик по сравнению с
другими методами
- Создадим базовую фабрику и применим ее к существующим `spec'ам`
- Отредактируем фабрики, чтобы сделать их еще более удобными в использовании
- Сгенерируем тестовые данные с помощью гема [Faker](https://github.com/faker-ruby/faker).
- Рассмотрим более продвинутые фабрики, с ассоциациями Active Record
- В конце, поговорим о рисках, связанных со слишком глубоким внедрением фабрик в
ваших приложениях
* * *
## **Фабрики против фикстур**
Из коробки Rails предоставляет средства для быстрой генерации образцов данных, называемые
фикстуры. Фикстура - это, по сути, файл в формате YAML, который помогает создать образец
данных. Например, фикстура для нашей модели Contact может выглядеть следующим образом:

`contacts.yml`
```yml
aaron:
  firstname: "Aaron"
  lastname: "Sumner"
  email: "aaron@everydayrails.com"

john:
  firstname: "John"
  lastname: "Doe"
  email: "johndoe@nobody.org"
```
Затем, вызвав `contacts(:aaron)` в тесте, получаем контакт со всеми установленными атрибутами. У фикстур есть преимущества, но есть и недостатки. Не будем тратить много времени и говорить плохо о фикстурах - это уже сделали многие люди в сообществе тестировщиков Rails. 

>Есть две проблемы которые надо избегать: 
>1. Данные фикстуры хрупкие и их легко сломать (это означает, что вы тратите примерно столько же времени на тестовые данные, сколько и на реальный код)
>2. Rails не использует Active Record при загрузке данных из фикстур в тестовую базу данных. Это значит, что такие важные вещи, как валидация моделей, игнорируются. Это плохо!

Фабрики: Простые, гибкие, генераторы тестовых данных. `Factory Girl` - простой в использовании и легко настраиваемый гем для создания фабрик.

В сообществе Ruby всегда ведется дискуссия - что лучше? И у `Factory Girl` тоже есть свои недоброжелатели. Летом 2012 года в сети разгорелись дебаты по поводу достоинства фабрик. Ряд ярых противников, в том числе создатель Rails `Дэвид Хайнемайер Ханссон`, указывали, что фабрики являются основной причиной медленных тестов и что фабрики могут быть особенно громоздкими при сложных ассоциациях.

Хотя их точка зрения имеет право быть, что за простоту использования фабрик приходиться платить скоростью, все же медленный тест лучше, чем отсутствие теста и подход основанный на фабриках, упрощает работу для людей, которые только учатся тестировать. Всегда можно поменять фабрики на более эффективные методы позже, когда будете более комфортно себя чувствовать в тестировании.

А пока применим фабрики в нашем приложении.
* * *
## **Добавление фабрик в приложение**
В каталоге `spec`, добавим еще один подкаталог `factories`, внутри него файл `contacts.rb` со следующим содержанием:

`spec/factories/contacts.rb`
```ruby
FactoryGirl.define do
  factory :contact do
    firstname "John"
    lastname "Doe"
    sequence(:email) { |n| "johndoe#{n}@example.com"}
  end
end
```
Этот фрагмент кода создает фабрику, которую можно использовать во всех  `spec'ах`. Всякий раз, когда создаются тестовые данные с помощью `FactoryGirl.create(:contact)`, имя контакта будет `John Doe`. 

Адрес электронной почты: используется функция, предоставляемая `Factory Girl`, которая называется `sequence`. `Sequence` будет автоматически увеличивать `n` внутри блока, выдавая `johndoe1@example.com` , `johndoe2@example.com` и так далее, по мере того, как фабрика будет генерировать новые контакты. `Sequence` необходимы для любой модели, в которой есть проверка уникальности. (Позже в этой главе рассматривается хорошая альтернатива для генерации таких вещей как адреса электронной почты и имена, под названием [Faker](https://github.com/faker-ruby/faker)).
Хотя все атрибуты в этом примере являются строками, можно передавать и другие атрибуты, включая целые числа, булевы выражения и даты. Можно передать код `Ruby` для динамического присвоения значений, только внутри блока (как показано в примере `sequence` выше). 
Например, если бы нужны были дни рождения контактов, можно было легко сгенерировать эти даты фабриками, используя методы `Ruby datetime`, такие как: `33.years.ago` или `Date.parse('1980-05-13')`.
>Имена файлов для фабрик не столь специфичны, как имена файлов для `spec`. На самом деле можно включить все фабрики в один файл. Однако, генератор `Factory Girl` хранит их в `spec/factories`, с именем файла, которое является множественным числом модели, которой он соответствует (например: `spec/factories/contacts.rb` для модели `Contact`). Надо придерживаться этого подхода. 

>Итог: Пока ваши фабрики синтаксически корректны и расположены в `spec/factories/`, вы на правильном пути.

Вернемся к файлу `contact_spec.rb`, который создали в предыдущей главе и добавим в него запись:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do
  it "has a valid factory" do
    expect(FactoryGirl.build(:contact)).to be_valid
  end

  ## more specs
end
```
При этом создается (но не сохраняется) новый контакт с атрибутами, создаными фабрикой. Затем проверяется валидность этого нового контакта. Сравним это с `spec'ой` из
предыдущей главы, где для прохождения проверки требовалось прописать все необходимые атрибуты:

```ruby
it "is valid with a firstname, lastname and email" do
  contact = Contact.new(
    firstname: 'Aaron',
    lastname: 'Sumner',
    email: 'tester@example.com')
  expect(contact).to be_valid
end
```
Вернемся к существующим `spec'ам`, теперь используя `Factory Girl` упростим создание данных. Переопределим один или несколько атрибутов для генерации данных из фабрики:

`spec/models/contact_spec.rb`
```ruby
it "is invalid without a firstname" do
  contact = FactoryGirl.build(:contact, firstname: nil)
  contact.valid?
  expect(contact.errors[:firstname]).to include("can't be blank")
end

it "is invalid without a lastname" do
  contact = FactoryGirl.build(:contact, lastname: nil)
  contact.valid?
  expect(contact.errors[:lastname]).to include("can't be blank")
end

it "is invalid without an email address" do
  contact = FactoryGirl.build(:contact, email: nil)
  contact.valid?
  expect(contact.errors[:email]).to include("can't be blank")
end

it "returns a contact's full name as a string" do
  contact = FactoryGirl.build(:contact,
    firstname: "Jane",
    lastname: "Smith"
  )
  expect(contact.name).to eq 'Jane Smith'
end
```
Эти проверки довольно просты. Как и в предыдущем проверке, они используют метод `Factory Girl's build` метод для создания нового, пока еще записанного в БД `Contact'а`. 
- В первой проверке генерируется контакт, которому не присвоено имя. 
- Во второй проверке не присвоена фамилия. Поскольку наша модель контакта проверяет наличие как имени так и фамилии, в обоих примерах ожидаем увидеть ошибки. 
- Следуя той же схеме, проверяем email. 
- Четвертая проверка немного отличается, она не использует фабрику. Создаем новый контакт с определенными значениями для `firstname` и `lastname` . Потом убеждаемся, что метод `name` возвращает значение строки, которое ожидаем.

В следующей `spec'е` незначительные изменения:

`spec/models/contact_spec.rb`
```ruby
it "is invalid with a duplicate email address" do
  FactoryGirl.create(:contact, email: 'aaron@example.com')
  contact = FactoryGirl.build(:contact, email: 'aaron@example.com')
  contact.valid?
  expect(contact.errors[:email]).to include('has already been taken')
end
```
В этой проверке убеждаемся, что атрибут `email` тестового контакта не является дубликатом существующего контакта. Для этого нам нужно создать контакт в БД, используем `FactoryGirl.create`, чтобы сохранить контакт в БД с тем же адресом электронной почты.
>Памятка: Используйте `FactoryGirl.build` для создания нового тестового объекта в памяти, а `FactoryGirl.create` для сохранения тестового объекта в тестовой базе.
* * *
## **Упрощение синтаксиса**

Приходиться писать `FactoryGirl.build(:contact)` каждый раз, когда нужен новый контакт - это громоздко. К счастью, `Factory Girl версии 3.0 и новее` упрощает жизнь Rails-программиста с помощью небольшой конфигурации. Добавьте ее в любое место внутри
блока `RSpec.configure`, расположенного в `rails_helper.rb`:

`spec/rails_helper.rb`
```ruby
RSpec.configure do |config|
  # Include Factory Girl syntax to simplify calls to factories
  config.include FactoryGirl::Syntax::Methods

  # other configurations omitted ...
end
```
Так же можно удалить или закомментировать конфигурацию `config.fixture_path`  - фикстуры не будут использоваться!
Теперь в `spec'ах` можно использовать более короткий синтаксис `build(:contact)`,  `create(:contact)` , также `attributes_for(:contact)` и `build_stubbed(:contact)`, которые будут использоваться в следующих главах.

Пример обновленной и более компактной `spec'и` модели:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do
  it "has a valid factory" do
    expect(build(:contact)).to be_valid
  end

  it "is invalid without a firstname" do
    contact = build(:contact, firstname: nil)
    contact.valid?
    expect(contact.errors[:firstname]).to include("can't be blank")
  end

  it "is invalid without a lastname" do
    contact = build(:contact, lastname: nil)
    contact.valid?
    expect(contact.errors[:lastname]).to include("can't be blank")
  end

  # remaining examples omitted ...
end
```
Более читабельный, но совершенно необязательный синтаксис.
* * *
## **Ассоциации и наследование в фабриках**

Если создать фабрику для модели `Phone`, учитывая приобретеные знания, она будет выглядеть примерно так:

`spec/factories/phones.rb`
```ruby
FactoryGirl.define do
  factory :phone do
    association :contact
    phone '123-555-1234'
    phone_type 'home'
  end
end
```
Новым здесь является вызов `:association`, который указывает `Factory Girl` создать новый `Contact` для этого телефона, если он не был передан в методами `build` или `create`.

Но у контакта может быть три типа телефонов - домашний, рабочий и мобильный. До сих пор,
если надо было указать в `spec'е` не домашний телефон, делали это следующим образом:

`spec/models/phone_spec.rb`
```ruby
it "allows two contacts to share a phone number" do
  create(:phone,
    phone_type: 'home',
    phone: "785-555-1234")
  expect(build(:phone,
    phone_type: 'home',
    phone: "785-555-1234")).to be_valid
end
```
Проведем некоторый рефакторинг. `Factory Girl` предоставляет возможность создавать наследуемые фабрики, переопределяя атрибуты по мере необходимости. Другими словами, если нужен офисный телефон в `spec'е`, можно создать его с помощью `build(:office_phone)` (или более длинным выражением `FactoryGirl.build(:office_phone)`, если так удобнее).
Вот как это выглядит:

`spec/factories/phones.rb`
```ruby
FactoryGirl.define do
  factory :phone do
    association :contact
    phone '123-555-1234'

    factory :home_phone do
      phone_type 'home'
    end

    factory :work_phone do
      phone_type 'work'
    end

    factory :mobile_phone do
      phone_type 'mobile'
    end
  end
end
```
`spec'у` можно упростить:

`spec/models/phone_spec.rb`
```ruby
require 'rails_helper'

describe Phone do
  it "does not allow duplicate phone numbers per contact" do
    contact = create(:contact)
    create(:home_phone,
      contact: contact,
      phone: '785-555-1234'
    )
    mobile_phone = build(:mobile_phone,
      contact: contact,
      phone: '785-555-1234'
    )

    mobile_phone.valid?
    expect(mobile_phone.errors[:phone]).to include('has already been taken')
  end

  it "allows two contacts to share a phone number" do
    create(:home_phone,
      phone: '785-555-1234'
    )
    expect(build(:home_phone, phone: "785-555-1234")).to be_valid
  end
end
```
Этот синтаксис пригодится в последующих главах, когда нужно будет создавать различные типы пользователей (администраторов и обычных юзеров) для тестирования механизмов аутентификации и авторизации.
* * *
## **Генерация реалистичных вымышленных данных**
Ранее использовалась `sequence`, чтобы сгенерировать уникальные адреса электронной почты. Можно сгенерировать более реалистичные тестовые данные, используя генератор вымышленных данных, который называется - `Faker` - это `gem Ruby` для генерации вымышленных имен, адресов, предложений, отлично подходит для тестирования.

Добавим вымышленные данные в наши фабрики:

`spec/factories/contacts.rb`
```ruby
FactoryGirl.define do
  factory :contact do
    firstname { Faker::Name.first_name }
    lastname { Faker::Name.last_name }
    email { Faker::Internet.email }
  end
end
```
Теперь `spec'а` будет использовать случайный адрес электронной почты каждый раз, когда используется фабрика генерации телефонов. (Можно посмотреть `log/test.log` после запуска `spec'и`, чтобы увидеть адреса электронной почты которые были добавлены в базу данных файлом `contact_spec.rb`). 

>Следует обратить внимание на два важных момента: 
>1. Загрузка библиотеки `Faker` происходит в первой строке фабрики
>2. Передается метод `Faker::Internet.email` внутри блока - `Factory Girl`  - это назвается "ленивым атрибутом" в отличие от статически добавляемой строки, которая была ранее.

Вернемся к фабрике генерации телефонов. Вместо того чтобы давать каждому
новому телефону номер по умолчанию, создадим уникальные, случайные, реалистичные номера:

`spec/factories/phones.rb`
```ruby
FactoryGirl.define do
  factory :phone do
    association :contact
    phone { Faker::PhoneNumber.phone_number }

    # child factories omitted ...
  end
end
```
Этот подход не является необходимым. Можно продолжать использовать `sequences` и `spec'и` все равно пройдут. Но `Faker` дает больше реалистичных данных для тестирования.
`Faker` может генерировать разные типы случайных данных, такие как адреса, вымышленые названия и слоганы и даже вымышленые слова. Также текст-заполнитель [lorem](https://www.lipsum.com/).

>Можно ознакомиться с [Forgery](https://github.com/sevenwire/forgery) как альтернативой `Faker`. `Forgery` аналогичен, но имеет немного другой синтаксис. Существует также [ffaker](https://github.com/ffaker/ffaker), доработаный вариант `Faker`, работающий в 20 раз быстрее оригинала.
* * *
## **Расширенные ассоциации**

`spec'и` для проверки валидаций, были относительно просты. Они не проверяли, создаются ли при создании контакта, три телефонных номера. Как можно это проверить? Как создать фабрику, чтобы убедиться, что тестовые контакты состоят из реалистичных данных?

Ответ заключается в использовании `callback'ов`, встроенных в `Factory Girl`, для добавления дополнительного кода в данную фабрику. Эти `callback'и` особенно полезны при тестировании вложенных атрибутов, например позволяют добавлять номера телефонов при создании или редактировании контакта. 

Эта модификация фабрики для генерации контактов использует `callback` `after`, чтобы убедиться, что у нового контакта будут все три типа телефонов:

`spec/factories/contacts.rb`
```ruby

FactoryGirl.define do
  factory :contact do
    firstname { Faker::Name.first_name }
    lastname { Faker::Name.last_name }
    email { Faker::Internet.email }

    after(:build) do |contact|
      [:home_phone, :work_phone, :mobile_phone].each do |phone|
        contact.phones << FactoryGirl.build(:phone,
          phone_type: phone, contact: contact)
      end
    end
  end
end
```
`after(:build)` принимает блок, внутри блока, массив из трех телефонных типов, которые  используется для создания телефонных номеров контакта. 

Пример, что это работает:

`spec/models/contact_spec.rb`
```ruby
it "has three phone numbers" do
  expect(create(:contact).phones.count).to eq 3
end
```
Можно добавить еще валидацию в модель `Contact`:

`app/models/contact.rb`
```ruby
validates :phones, length: { is: 3 }
```

В качестве эксперимента можно изменить значение в валидации на какое-нибудь другое число,
и запустить тесты снова. Все проверки, которые ожидали валидный контакт упадут. Второй эксперимент - закомментировать блок `after` в фабрике для генерации контактов, запустить тесты, также много проверок не пройдут.

>Хотя пример специфичен, `callback'и` в  `Factory Girl` не так ограничены. Пост [Get Your Callbacks On with Factory Girl 3.3](https://thoughtbot.com/blog/get-your-callbacks-on-with-factory-bot-3-3) на Thoughtbot о том, как использовать преимущества `collback'ов`.

`after(:build)`, `before(:build)`, `before(:create)` и `after(:create)` - это `callback'и`. Все они работают аналогично.
* * *
## **Как злоупотребляют фабриками**

Фабрики - это прекрасно, за исключением тех случаев, когда неконтролируемое использование фабрик может привести к низкой скорости тестов - особенно когда появляются сложные ассоциации. Создание последней фабрикой трех дополнительных объектов при каждом вызове - это перебор, но на данный момент, удобство генерации этих данных одним методом вместо нескольких, перевешивает все недостатки.

Хотя генерирование ассоциаций фабриками - это простой способ ускорить написание тестов, этим также легко злоупотребить, и это часто является причиной того, что время выполнения тестов очень увеличивается. В таком случае лучше удалить ассоциации из фабрик и заполнить тестовые данные вручную. Также можно вернуться к подходу `Plain Old Ruby Objects`
который использовался в `главе 3`  или к гибридному подходу.

Если изучать другие подходы тестирования в целом или RSpec в частности, несомненно сталкнетесь с терминами `mocks` и `stubs`. Если у вас уже есть некоторый опыт в тестировании возникнет вопрос: почему постоянно используются фабрики, а не `mocks` и `stubs`. Ответ заключается в том, что базовые объекты и фабрики проще, для того чтобы почувствовать себя комфортно в тестировании. И чрезмерное использование `mocks` и `stubs` может привести к другим проблемам.

Поскольку на данном этапе приложение довольно маленькое, любое увеличение скорости, при использовании более сложного подхода будет незначительным. Тем не менее, `mocks` и `stubs` играют свою роль в тестировании. О них подробнее в главах 9 и 10.
* * *
## **Резюмируя**

В этой главе рассмотрели `Factory Girl`. Теперь меньше синтаксиса в `spec'ах`, гибкий способ создания определенных типов данных, более реалистичные вымышленные данные и варианты создания более сложных ассоциаций по мере необходимости. Эти знания помогут справиться с большинством задач тестирования, но в [документации Factory Girl](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md) есть дополнительные примеры. `Factory Girl` заслуживает отдельной небольшой книги.