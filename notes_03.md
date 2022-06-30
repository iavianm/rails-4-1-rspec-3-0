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

Переключиться на ветку 3 главы
`git checkout -b 03_models origin/03_models`
* * *
## **3 глава - Model specs**
В этой главе мы выполним следующие задачи:
>- Создадим спецификацию для существующей модели Contact. 
>- Затем мы напишем тесты для проверки модели, класса и instance методов
* * *
## **Анатомия тестирования модели**
>Я думаю, что проще всего научиться тестировать на уровне модели, потому что это позволяет вам изучить и протестировать основные строительные блоки приложения. Хорошо протестированный код на этом уровене является ключевым — прочная основа — это первый шаг к надежной базе приложения.

Для начала `spec'а` модели должна включать следующие тесты:
>- Метод `create` модели при правильных атрибутах должен выполняться.
>- Данные, не прошедшие проверку, не должны быть действительными.
>- Методы класса и экземпляра работают так, как ожидалось..

**Рассмотрим нашу модель Contact:**
```
describe Contact do
  it "is valid with a firstname, lastname and email"
  it "is invalid without a firstname"
  it "is invalid without a lastname"
  it "is invalid without an email address"
  it "is invalid with a duplicate email address"
  it "returns a contact's full name as a string"
end
```
Это простая `spec'а` для простой модели, но она описывает четыре лучших практики:
- Описывается набор ожиданий - как должна выглядеть модель Contact и как себя вести. 
- Каждый запрос, только для одной проверки. 
>Тестируются имя, фамилия и электронная почта отдельно. Таким образом, если тест потерпит неудачу, то сразу понятно в чем ошибка

- Каждый запрос является явным.
- Описание каждой проверки должно начинается с глагола (должен - не должен), для лучшего понимания происходящего.
* * *
## **Создание model spec**
>Откроем каталог spec, при необходимости создадим подкаталог с именем models. Внутри этого подкаталога создадим файл с именем contact_spec.rb и добавим следующее:

`spec/models/contact_spec.rb`
```
require 'rails_helper'

describe Contact do
  it "is valid with a firstname, lastname and email"
  it "is invalid without a firstname"
  it "is invalid without a lastname"
  it "is invalid without an email address"
  it "is invalid with a duplicate email address"
  it "returns a contact's full name as a string"
end
```
>Надо обратить внимание на запись `require 'rails_helper'` и не забывать ее при добавлении тестов вручную

>Месторасположение файлов RSpec очень важно, структура тестов должна повторять структуру вашего приложения

Запустив тестирование сейчас, введя в командной строке `bin/rspec` или просто `rspec`, получим вывод примерно такой:
```yml
Contact
  is valid with a firstname, lastname and email
    (PENDING: Not yet implemented)
  is invalid without a firstname
    (PENDING: Not yet implemented)
  is invalid without a lastname
    (PENDING: Not yet implemented)
  is invalid without an email address
    (PENDING: Not yet implemented)
  is invalid with a duplicate email address
    (PENDING: Not yet implemented)
  returns a contact's full name as a string
    (PENDING: Not yet implemented)

Pending:
  Contact is valid with a firstname, lastname and email
    # Not yet implemented
    # ./spec/models/contact_spec.rb:4
  Contact is invalid without a firstname
    # Not yet implemented
    # ./spec/models/contact_spec.rb:5
  Contact is invalid without a lastname
    # Not yet implemented
    # ./spec/models/contact_spec.rb:6
  Contact is invalid without an email address
    # Not yet implemented
    # ./spec/models/contact_spec.rb:7
  Contact is invalid with a duplicate email address
    # Not yet implemented
    # ./spec/models/contact_spec.rb:8
  Contact returns a contact's full name as a string
    # Not yet implemented
    # ./spec/models/contact_spec.rb:9

Finished in 0.00105 seconds (files took 2.42 seconds to load)
6 examples, 0 failures, 6 pending
```
* * *
## **Новый синтаксис RSpec**
С выходом версии  RSpec 2.11, поменялся синтаксис написания тестов
>старый синтаксис
```ruby
it "adds 2 and 1 to make 3" do
  (2 + 1).should eq 3
end
```
>новый синтаксис
```ruby
it "adds 2 and 1 to make 3" do
  expect(2 + 1).to eq 3
end
```
>Старый синтаксис работает, но будет предупреждение о устаревании. Предупреждение можно отключить, но лучше привыкать к новому синтаксису

Напишем spec'у для модели Contact:

`spec/models/contact_spec.rb`
```
require 'rails_helper'

describe Contact do
  it "is valid with a firstname, lastname and email" do
    contact = Contact.new(
      firstname: 'Aaron',
      lastname: 'Sumner',
      email: 'tester@example.com')
    expect(contact).to be_valid
  end
  
  # remaining examples to come
end
```
>В этом примере используется проверка на валидацию `be_valid`, чтобы убедиться, что наша модель выглядит нужным образом

>Если запустить тестирование сейчас, то будет один успешно пройденый тест.
* * *
## **Тестирование валидаций**
>Валидации — хороший способ подойти к автоматизированному тестированию. Эти тесты могут быть написаны всего одной или двумя строками кода, особенно когда мы используем фабрики (следующая глава). 

Рассмотрим некоторые детали spec'и для проверки имени:

`spec/models/contact_spec.rb`

```ruby
it "is invalid without a firstname" do
  contact = Contact.new(firstname: nil)
  contact.valid?
  expect(contact.errors[:firstname]).to include("can't be blank")
end
```

Мы ожидаем, что новый контакт (с name, явно установленным nil )
будет недействительным, поэтому для имени контакта будет возвращено показанное сообщение об ошибке. `can't be blank`.
Проверяем это с помощью `include` matcher'а, который проверяет значение
Если снова запустить `bin/rspec`, будут два успешных результата тестирования.

Чтобы убедиться, что мы не получаем ложных срабатываний, изменим ожидание с  `to` на `not_to`:

`spec/models/contact_spec.rb`
```ruby
it "is invalid without a firstname" do
contact = Contact.new(firstname: nil)
contact.valid?
expect(contact.errors[:firstname]).not_to include("can't be blank")
end
```
RSpec сообщает об ошибке:
```
Failures:
  1) Contact is invalid without a firstname
     Failure/Error: expect(contact.errors[:firstname]).not_to
         include("can't be blank")
       expected ["can't be blank"] not to include "can't be blank"
     # ./spec/models/contact_spec.rb:15:in `block (2 levels) in
         <top (required)>'
```
Это простой способ убедиться, что тесты работают правильно

>RSpec предоставляет типы ожиданий `not_to` и `to_not` . Они идентичны. В книге используется `not_to`.

>Если вы использовали более раннюю версию RSpec, возможно, вы привыкли использовать `have` matcher и `error_on` хелпер. Они были удалены из ядра RSpec 3. Вы все еще можете их использовать, включив `rspec-collection_matchers` в `Gemfile: test`.

Теперь мы можем использовать тот же подход для валидации `:lastname`.

`spec/models/contact_spec.rb`
```ruby
it "is invalid without a lastname" do
  contact = Contact.new(lastname: nil)
  contact.valid?
  expect(contact.errors[:lastname]).to include("can't be blank")
end
```

>Можно подумать, что эти тесты бессмысленны - что без них можно убедиться, включены ли валидации в модель. Правда в том, что забыть про отсутствие валидаций и  не обратить на это внимание, легче чем вы можете себе представить. Однако, что еще более важно, если вы подумаете о том, какие валидации нужны в вашей модели во время написания тестов (в идеале и в конечном итоге, в стиле `Test-Driven-Development`), вы с большей вероятностью не забудете их включить.

Проверка адреса электронной почты на уникальность:

`spec/models/contact_spec.rb`
```ruby
it "is invalid with a duplicate email address" do
  Contact.create(
    firstname: 'Joe', lastname: 'Tester',
    email: 'tester@example.com'
  )
  contact = Contact.new(
    firstname: 'Jane', lastname: 'Tester',
    email: 'tester@example.com'
  )
  contact.valid?
  expect(contact.errors[:email]).to include("has already been taken")
end
```
>***Обратите внимание на тонкое различие:*** В этом случае мы сохранили контакт **вызвав `create` у Contact вместо `new`** , а затем инстанцировали второй контакт в качестве объекта фактического теста. Для этого первый сохраненный контакт должен быть с именем, фамилией и адресом электронной почты. В дальнейшем мы рассмотрим утилиты для упрощения этого процесса.

Проведем более сложную проверку. Допустим, мы хотим убедиться, что нет дублирования телефонного номера пользователя - его домашний, рабочий и мобильный телефоны должны быть уникальными в рамках данного пользователя. Как мы можем это проверить?

Spec'а модели Phone будет выглядить следующим образом:

`spec/models/phone_spec.rb`
```ruby
require 'rails_helper'

describe Phone do
  it "does not allow duplicate phone numbers per contact" do
    contact = Contact.create(
      firstname: 'Joe',
      lastname: 'Tester',
      email: 'joetester@example.com'
    )
    contact.phones.create(
      phone_type: 'home',
      phone: '785-555-1234'
    )
    mobile_phone = contact.phones.build(
      phone_type: 'mobile',
      phone: '785-555-1234'
    )

    mobile_phone.valid?
    expect(mobile_phone.errors[:phone]).to include('has already been taken')
  end

  it "allows two contacts to share a phone number" do
    contact = Contact.create(
      firstname: 'Joe',
      lastname: 'Tester',
      email: 'joetester@example.com'
    )
    contact.phones.create(
      phone_type: 'home',
      phone: '785-555-1234'
    )
    other_contact = Contact.new
    other_phone = other_contact.phones.build(
      phone_type: 'home',
      phone: '785-555-1234'
    )

    expect(other_phone).to be_valid
  end
end
```
Поскольку модели Contact и Phone связаны между собой посредством Active Record
нужно предоставить немного дополнительной информации. 

В первом случае: у нас есть контакт, за которым закреплены оба телефона. 
Во втором случае: один и тот же номер телефона присвоен двум уникальным контактам.
>Обратите внимание, что в обоих примерах нужно создать контакт или сохранить его в базе данных, чтобы присвоить ему телефоны, которые мы тестируем.

Поскольку модель Phone имеет следующую валидацию:

`app/models/phone.rb`
```ruby
validates :phone, uniqueness: { scope: :contact_id }
```
Эти spec'и пройдут без проблем.
>Конечно, валидация может быть более сложной, чем просто требования специфичного скоупа.
Валидация может включать сложное регулярное выражение или собственный валидатор. Возьмите за правило привычку тестировать эти валидации - не только правильные варианты, где все валидно, но и условия с ошибками. Например, в примерах, которые мы создавали, мы проверяли, что происходит, когда объект инициализируется со значениями nil.
* * *
## **Тестирование instance методов**
Было бы удобно обращаться к @contact.name, чтобы отобразить полные имена наших контактов вместо того, чтобы каждый раз объединять имя и фамилию в строку. Поэтому мы добавили этот метод в класс `Contact`:

`app/models/contact.rb`
```ruby
def name
  [firstname, lastname].join(' ')
end
```
Можем использовать те же методы, которые использовали для валидаций.
Чтобы создать spec'у:

`spec/models/contact_spec.rb`
```ruby
it "returns a contact's full name as a string" do
  contact = Contact.new(firstname: 'John', lastname: 'Doe',
    email: 'johndoe@example.com')
  expect(contact.name).to eq 'John Doe'
end
```
>RSpec использует eq или eql, а не ==, для проверки равенства.
* * *
## **Тестирование методов класса и скоупов**
Давайте проверим способность модели Contact возвращать список контактов, чьи имена начинаются с заданной буквы. Например, если нажать на S, должны получить Smith, Sumner
и так далее, но не Jones. Есть несколько способов реализовать это - для
демонстрационных целей показан один из них.

В модели реализуется эта функциональность в следующем простом методе:

`app/models/contact.rb`
```ruby
def self.by_letter(letter)
  where("lastname LIKE ?", "#{letter}%").order(:lastname)
end
```
Чтобы проверить этот метод, добавим следующее в Contact spec:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do

  # earlier validation examples omitted ...

  it "returns a sorted array of results that match" do
    smith = Contact.create(
      firstname: 'John',
      lastname: 'Smith',
      email: 'jsmith@example.com'
    )
    jones = Contact.create(
      firstname: 'Tim',
      lastname: 'Jones',
      email: 'tjones@example.com'
    )
    johnson = Contact.create(
      firstname: 'John',
      lastname: 'Johnson',
      email: 'jjohnson@example.com'
    )
    expect(Contact.by_letter("J")).to eq [johnson, jones]
  end
end
```

>Обратите внимание, что мы тестируем как результаты запроса, так и порядок сортировки. Так как мы сортируем по фамилии, johnson должен быть первым в результатах.
* * *
## **Тестирование на наличие сбоев**
Мы проверили удачный запрос, когда пользователь выбирает имя, по которому мы можем получить результаты, но как быть в случаях, когда выбранная буква не дает никаких результатов? Надо протестировать и это тоже. Следующая spec'а должна сделать это:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do

  # validation examples ...

  it "omits results that do not match" do
    smith = Contact.create(
      firstname: 'John',
      lastname: 'Smith',
      email: 'jsmith@example.com'
    )
    jones = Contact.create(
      firstname: 'Tim',
      lastname: 'Jones',
      email: 'tjones@example.com'
    )
    johnson = Contact.create(
      firstname: 'John',
      lastname: 'Johnson',
      email: 'jjohnson@example.com'
    )
    expect(Contact.by_letter("J")).not_to include smith
  end
end
```

Эта spec'а использует матчер RSpec - include, чтобы определить, что метод Contact.by_letter("J") возвращает массив не только для букв при выборе которых есть результат, но и для букв без результатов.
* * *
## **Подробнее о матчерах**
Мы уже видели в действии три матчера. 
`be_valid` , который предоставляется гемом `rspec-rails` для проверки валидности модели Rails. 
`eq` и `include` из `rspec-expectations`, установленного вместе с `rspec-rails`

Полный список матчеров RSpec по умолчанию можно найти в README для
репозитория [rspec-expectations на GitHub](https://github.com/rspec/rspec-expectations). В главе 7 мы рассмотрим возможность создания собственных матчеров.
* * *
## **Принцип DRY для spec: describe, context, before и after**

[Принцип DRY](https://ru.wikipedia.org/wiki/Don%E2%80%99t_repeat_yourself) - это принцип разработки программного обеспечения, нацеленный на снижение повторения информации различного рода.

Если вы следили за кодом примера, то наверняка заметили расхождение с тем, что рассматривается здесь. В этом коде используется еще одна функция RSpec [before](https://github.com/everydayrails/rails-4-1-rspec-3-0/blob/13eacf54fc76e60710f915524bf41268b9ad8845/spec/models/contact_spec.rb#L53), чтобы упростить и уменьшить количество кода. Действительно, в примерах `spec` есть некоторая избыточность: мы создаем одни и те же три объекта в каждом примере. Как и в коде вашего приложения, принцип DRY применим к тестам
(за некоторыми исключениями, о которых мы узнаем чуть позже). Давайте воспользуемся несколькими трюками RSpec для наведения порядка.
Первое, что надо сделать, это создать блок `describe` внутри блока `describe Contact`, чтобы логически разделить блоки spec'и. 

Выглядеть это будет следующим образом:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do

  # validation examples ...

  describe "filter last name by letter" do
    # filtering examples ...
  end
end
```
Включим пару блоков `context` - для совпадающих букв и несовпадающих:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do

  # validation examples ...

  describe "filter last name by letter" do
    context "matching letters" do
      # matching examples ...
    end

    context "non-matching letters" do
      # non-matching examples ...
    end
  end
end
```
>Хотя `describe` и `context`  взаимозаменяемы, предпочтительно использовать их следующим образом - `describe` описывает общую функциональность
класса, `context` описывает конкретное ожидаемое состояние. В нашем случае есть состояние когда по выбранной букве есть результат и состояние когда нет.

>Мы создаем набросок примеров, чтобы понять как сортировать похожие примеры вместе. Это делает spec'у более читабельной. 

Закончим очистку нашей реорганизованной spec'и с помощью before:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do

  # validation examples ...

  describe "filter last name by letter" do
    before :each do
      @smith = Contact.create(
        firstname: 'John',
        lastname: 'Smith',
        email: 'jsmith@example.com'
      )
      @jones = Contact.create(
        firstname: 'Tim',
        lastname: 'Jones',
        email: 'tjones@example.com'
      )
      @johnson = Contact.create(
        firstname: 'John',
        lastname: 'Johnson',
        email: 'jjohnson@example.com'
      )
    end

    context "matching letters" do
      # matching examples ...
    end

    context "non-matching letters" do
      # non-matching examples ...
    end
  end
end
```
`before` в RSpec очень важны для очистки ваших `spec` от избыточности кода.
Код содержащийся в блоке `before`, запускается перед каждой проверкой в блоке `description` - но не за пределами этого блока. RSpec
создаст контакты содержимые в блоке `before` для каждой проверки в отдельности. 
В данном примере `before` будет вызываться только в блоке `filter last name by letter`.

>`:each` - это поведение `before` по умолчанию, многие рубисты используют более короткую формулировку  `before do` для создания блоков `before`. Автор предпочитает явно указывать `before :each do`, на протяжении всей книги будет использоваться  `before :each do`.

Обратите внимание, что поскольку тестовые контакты больше не создаются
внутри каждой проверки, мы должны сделать их переменными экземпляра, чтобы они были доступны вне блока `before`.
Если `spec`'а требует какого-то переподключения к базе можем использовать `after` для очистки данных после проверок.
Но поскольку RSpec по умолчанию очищает базы данных, `after` используется крайне редко. А вот `before` незаменим.

Написанная по правилам spec'а:

`spec/models/contact_spec.rb`
```ruby
require 'rails_helper'

describe Contact do
  it "is valid with a firstname, lastname and email" do
    contact = Contact.new(
      firstname: 'Aaron',
      lastname: 'Sumner',
      email: 'tester@example.com')
    expect(contact).to be_valid
  end

  it "is invalid without a firstname" do
    contact = Contact.new(firstname: nil)
    contact.valid?
    expect(contact.errors[:firstname]).to include("can't be blank")
  end

  it "is invalid without a lastname" do
    contact = Contact.new(lastname: nil)
    contact.valid?
    expect(contact.errors[:lastname]).to include("can't be blank")
  end

  it "is invalid without an email address" do
    contact = Contact.new(email: nil)
    contact.valid?
    expect(contact.errors[:email]).to include("can't be blank")
  end

  it "is invalid with a duplicate email address" do
    Contact.create(
      firstname: 'Joe', lastname: 'Tester',
      email: 'tester@example.com'
    )
    contact = Contact.new(
      firstname: 'Jane', lastname: 'Tester',
      email: 'tester@example.com'
    )
    contact.valid?
    expect(contact.errors[:email]).to include("has already been taken")
  end

  it "returns a contact's full name as a string" do
    contact = Contact.new(
      firstname: 'John',
      lastname: 'Doe',
      email: 'johndoe@example.com'
    )
    expect(contact.name).to eq 'John Doe'
  end

  describe "filter last name by letter" do
    before :each do
      @smith = Contact.create(
        firstname: 'John',
        lastname: 'Smith',
        email: 'jsmith@example.com'
      )
      @jones = Contact.create(
        firstname: 'Tim',
        lastname: 'Jones',
        email: 'tjones@example.com'
      )
      @johnson = Contact.create(
        firstname: 'John',
        lastname: 'Johnson',
        email: 'jjohnson@example.com'
      )
    end

    context "with matching letters" do
      it "returns a sorted array of results that match" do
        expect(Contact.by_letter("J")).to eq [@johnson, @jones]
      end
    end

    context "with non-matching letters" do
      it "omits results that do not match" do
        expect(Contact.by_letter("J")).not_to include @smith
      end
    end
  end
end
```
Когда мы запустим `spec`'и, будет читаемый вывод результатов (так как мы сказали RSpec использовать `--format documentation` [во второй главе](https://github.com/everydayrails/rails-4-1-rspec-3-0/blob/e8ac92382d44b86e8df59607020481a38d1f0c0d/.rspec#L3))
```
Contact
  is valid with a firstname, lastname and email
  is invalid without a firstname
  is invalid without a lastname
  is invalid without an email address
  is invalid with a duplicate email address
  returns a contact's full name as a string
  filter last name by letter
    with matching letters
      returns a sorted array of results that match
    with non-matching letters
      omits results that do not match

Phone
  does not allow duplicate phone numbers per contact
  allows two contacts to share a phone number

Finished in 0.51654 seconds (files took 2.24 seconds to load)
10 examples, 0 failures
```
>Некоторые разработчики предпочитают использовать имена методов для описания вложенных `describe` блоков. Например назвать `filter last name by letter` как `#by_letter`. Автору не нравится так делать, поскольку он считает, что название проверки должно описывать поведение кода, а не название проверяемого метода.
Тем не менее, такой подход, тоже можно использовать.
* * *
## **How DRY is too DRY?**
В этой главе мы уделили много времени организации `spec` в простые для понимания блоки кода.

Блоки позволяют добиться удобной читаемости, но ими также легко злоупотреблять.
При создании тестовых условий для вашего приложения, можно отступить от принципа DRY. Если вы обнаружите, что прокручиваете вверх и вниз большой `spec`, чтобы увидеть, что именно вы тестируете (или позже, загружаете слишком много внешних вспомогательных файлов для ваших тестов), подумайте о том, чтобы продублировать тестовые данные внутри меньших блоков тестирования или даже в самих проверках.

Тем не менее, хорошо названные переменные и методы могут быть очень полезны - например в примере выше мы использовали `@jones` и `@johnson` в качестве тестовых контактов, они гораздо удобнее в использовании чем `@user1` и `@user2`, так как нам надо было убедиться в том, что поиск по первым буквам работает так, как нам нужно. 

Хорошими могут быть переменные типа `@admin_user` и `@guest_user`, когда мы перейдем к тестированию пользователей с определенными ролями в `главе 6`. Будьте внимательны к вашими именам переменных!
* * *
## **Резюмируя**
В этой главе основное внимание уделялось тому, как тестировать модели, но также рассмотрели множество других важных моментов.

Методы, которые будем использовать в других типах `spec` в будущем:
- Используйте активные, эксплицитные ожидания: используйте глаголы, чтобы объяснить смысл проверки. Проверяйте только один результат для каждого примера.
- Проверьте, что вы ожидаете от проверки и что может случиться: подумайте об обоих вариантах при написании проверок и соответствующим образом протестируйте
- Тестируйте на крайние(пограничные) случаи: Если у вас есть проверка, которая требует, чтобы пароль был от четырех до десяти символов, не стоит просто проверять восьмисимвольный пароль и считать, что все в порядке. Хороший набор тестов будет проверять как четыре и десять, так и на три и на одиннадцать символов. (Конечно, вы также можете спросить себя, почему вы разрешаете такие короткие пароли или не разрешаете более длинные. Тестирование - это также хорошая возможность поразмышлять над требованиями к приложению и коду).
- Организуйте свои `spec'и` для хорошей читабельности: Используйте `describe` и `context`, для сортировки похожих проверок в читаемый формат, а также блоки `before` и `after` для удаления дублирования кода. Однако в случае с тестами читабельность превалирует над DRY. Если вы обнаружите, что вам приходится слишком часто прокручивать `spec'у` вверх и вниз, стоит немного повториться.
* * *
## **Вопрос**
**Когда использовать `describe` , а когда `context`?**
- С точки зрения RSpec, вы можете использовать `describe` все время, если хотите. `Context` существует для того, чтобы сделать ваши `spec'и` более читабельными.