
[Former](http://anahkiasen.github.io/former/) - это название небольшого проекта, который я хочу тебе показать. Садись поближе, сейчас начнется...    
Так вот, Former - это такой хитрый пакет PHP, который позволяет делать всевозможные манипуляции с формами, и он очень удобный в использовании. Да и вообще он крутой чувак, если узнать его поближе.

Former пережуёт все и положит тебе прямо в рот - он обработает за тебя и отвалидирует ввод, автоматически сгруппирует поля, отловит отчет об ошибке, и даже создаст разметку для твоего любимого css-фрейма (Bootstrap, Foundation).
Ну а что бы узнать больше, я предлагаю тебе ознакомится со всеми, описанными ниже, фишками.


## Введение
Former нацелен на переосмысление удобного создания форм - изменяем свойства каждого поля в его собственной модели, со своими методами и свойствами. Это значит, что можно мутить такие фишки типа:
```php
Former::horizontal_open()
  ->id('MyForm')
  ->secure()
  ->rules(['name' => 'required'])
  ->method('GET')

  Former::xlarge_text('name')
    ->class('myclass')
    ->value('Joseph')
    ->required();

  Former::textarea('comments')
    ->rows(10)->columns(20)
    ->autofocus();

  Former::actions()
    ->large_primary_submit('Submit')
    ->large_inverse_reset('Reset')

Former::close()
```
Каждый раз, когда ты вызываешь метод, которого на самом деле нет, Former думает, что ты пытаешься задать какой-то атрибут и создает его магически. Поэтому ты можешь мутить такие вещи, как в приведенном выше примере ` ->rows(10) ;`, а если ты хочешь задать атрибут который содержит дефисы, ты просто заменяешь дефисы подчеркиваниями:
`->data_foo('bar')` преобразуется в  `data-foo="bar"`.
Теперь ты конечно спросишь: "а фто ефли я хатю падсеркивания в африбутах?"(ну охренеть, какой ты умный!). Ты всегда можешь обратится к методу `setAttribute('data_foo', 'bar')`. И тадааам! Наслаждайся.

В общем, в этом самая суть. Однако, Former - это и кое что еще... Читай ниже и ты узнаешь, на что способен Former.

## С чего начать
###Основная концепция
Former был задуман как View helper – в том смысле, что он предоставляет тебе класс, который ты можешь использовать прямо во вьюхах, для вывода готового HTML-кода.

```php
{!! Former::open()->method('GET') !!}
    {!! Former::text('name')->required() !!}
{!! Former::close() !!}
```
*(шорттеги были заменены мной на blade-синтаксис - п.п.)*

Но, ты точно также можешь запихать результат в переменную для передачи ее в шаблон в уже готовом виде:

```php
$form = Former::open()->method('GET');
    $form .= Former::text('name')->required();
$form .= Former::close();
```
## Установка
### Под Laravel 3
Копируешь laravel3 ветку репозитория в папку со своими бандлами, и добавляешь `'former' => array('auto' => true)` в файл бандлов. Потом добавляешь алиас `'Former' => 'Former\Facades\Former`' в  application файл, профит!

### Под Laravel 4
Установка Former'a проста, как два пальца об асфальт. Добавляешь строку в `composer.json` :
```json
"anahkiasen/former": "dev-master"
```
Допиливаешь поставщика Former'а в `app/config/app.php`
```
'Former\FormerServiceProvider',
```
и там же пишешь алиас:
```
'Former' => 'Former\Facades\Former',
```
 
###Или через установщик пакетов
Просто сделай  `artisan package:install anahkiasen/former:dev-master` и не парься.

###Без фрейма
Также как и под Laravel 4, добавь Former в файл Composer'а. А потом:
```php
use Former\Facades\Former;
```
И готово.

##Фишки
###Поддержка интеграции с Bootstrap и Foundation из коробки.
Это все ништяк, но вроде выглядит, будто налепили модификашек на Laravel'овский  Form-класс. Ну что у автора совсем нет фантазии? Вот где спрятана магия: Former распознает какого типа ты создаешь форму - вертикальную или горизонтальную, и сам оборачивает все поля в контрол-группы. Это значит, что если ты пишешь что-то вроде:
```php
Former::select('clients')->options($clients, 2)
  ->help('Pick some dude')
  ->state('warning')
 ```
На выходе получится вот это (Bootstrap) :
```html
<div class="control-group warning">
  <label for="clients" class="control-label">Clients</label>
  <div class="controls">
    <select id="clients" name="clients">
      <option value="0">Mickael</option>
      <option value="1">Joseph</option>
      <option value="2" selected="selected">Patrick</option>
    </select>
    <span class="help-inline">Pick some dude</span>
  </div>
</div>
```
По умолчанию Former использует Twitter Bootstrap, но ты можешь выбрать какой фрейм использовать с помощью метода `Former::framework()`. На данный момент поддерживаются 'TwitterBootstrap', 'ZurbFoundation' и 'Nude' (без фрейма).
```php
// Выключаем Bootstrap
Former::framework('Nude');

// Включаем его назад (господи, да включи ты мозг)
Former::framework('TwitterBootstrap');
```
Да, да и  для Foundation есть пример:
```php
Former::framework('ZurbFoundation');

Former::four_text('foo')->state('error')->help('bar')
```
Выдаст :
```html
<div class="error">
  <label for="foo">Foo</label>
  <input class="four" type="text" name="foo" id="foo">
  <small>Bar</small>
</div>
```
####Связываем с Laravel'овским валидатором.
Ладно, ок, мы типа посчистли все от мусора и бесконечных вызовов `Form::control_group()`. Теперь я слышу: "О, ты знаеф, я до фих пор проверял мои формы врусьную и со..." - нет, ты больше не будешь этого делать. Добро пожаловать в волшебный мир хелпера `withErrors`, который сделает тебя счастливым, и заставит коров давать вдвое больше молока. Раз уж ты уже завернул свои хорошенькие поля в контрол-группы, осталось только пойти до конца - проверить объект валидации на ошибки, которые могут иметь поля, и добавить их в .help-inline.

Тут ты используешь Former по-разному в зависимости от, как твой код ведет себя после зафейленой проверки:

Если ты рендеришь вьюху при ошибке (без редиректа):
```php
if($validation->fails()) {
  Former::withErrors($validation);
  return View::make('myview');
}
```
 Если у тебя редирект при фейле:
 ```php
if($validation->fails()) {
  return Redirect::to('login')
    ->with_errors($validation);
}
```
В последнем примере уже не нужно вызывать `Former::withErrors()`, ни в контроллере ни во вьюхе. Почему? Да потому что формер автоматом пытается распарсить ошибки в сессии, каждый раз, когда вызывается `Former::open()` Формер любит тебя и заботится о тебе. Но конечно же, мой догадливый друг, ты как-то можешь отключить это поведение. А именно: `Former::setOption('fetch_errors', false)`.

####Заполнение форм
Ты легко можешь заполонить форму данными используя метод `Former::populate()`. Ты даже можешь сделать это двумя разными способами. Первый - это просто передать туда массив:
```php
// Заполнит поле с именем 'name' значением 'value'
Former::populate( array('name' => 'value') )
```
А еще ты можешь передать прямо туда экземпляр модели Eloquent (это, кстати, открывает доступ к кое каким фишулям; но мы с тобой к этому еще вернемся).
```php
Former::populate( Client::find(2) )
```
Former распознает модель, и заполнит соответствующие поля соответствующими значениями.

Ну и конечно, ты можешь переопределить определенные поля ( да я мастер каламбуров ) уже после того как ты их заполнил из модели или массива:
```php
Former::populate($project)
Former::populateField('client', $project->client->name)
```
Оставшиеся поля формы можно легко заполнить просто запилив `->value('чето-там')`.   
Чтобы создать список опций для селекта, ты можешь вызвать `Former::select('foo')->options([array], [facultative: selected value])`. А еще ты можешь заполнить опции прямо из коллекции, типа вот так:
```php
Former::select('foo')->fromQuery(Client::all(), 'name', 'id')
```
Первый аргумент - собственно сама коллекция, второй - имя атрибута, который пойдет в текст, а третий, как ты уже догадался - значение (которое, кстати, `id` по дефолту ). 
Former, между тем, и тут без магии не обошелся - можно вообще не передавать дополнительные аргументы в метод. Надо только саму модель настроить. Former получит ключ по Eloquent'овскому методу `get_key()`, и использует `__toString()` метод для отрисовки текста. Например так:
```php
class Client extends Eloquent
{
  public static $key = 'code';

  public function __toString()
  {
    return $this->name;
  }
}

Former::select('clients')->fromQuery(Client::all());
```

Все это отрисуется в подобный код:
```html
<div class="control-group">
  <label for="foo" class="control-label">Foo</label>
  <div class="controls">
    <select id="foo" name="foo">
      @foreach(Client::all() as $client)
        @if(Input::get('foo', Input::old('foo')) == $client->code)
          <option selected="selected" value="{{ $client->code }}">{{ $client->name }}</option>
        @else
          <option value="{{ $client->code }}">{{ $client->name }}</option>
        @endif
      @endforeach
    </select>
  </div>
</div>
```
Former может даже заполнить поля прямо из отношений. Вместо тысячи слов:
```php
Former::populate(Client::find(2))

// Заполняешь $client->name
Former::text('name')

// Заполняешь $client->store->name
Former::text('store.name')

// Да детка, еще глубже
Former::text('customer.name.adress')

// Заполняешь из всех дат в резерве
Former::select('reservations.date')

// По сути тоже самое ^
Former::select('reservations')->fromQuery($client->reservations, 'date')

// Если ты запилишь отношение в текстовое поле, вместо селекта,
// они успешно сконкатенируются
Former::text('customers.name') // Выведет "name, name, name"

// кстати, можно перименовать поле уже после заполнения, коли приспичит.
Former::text('comment.title')->name('title')
```
##Datalist
А что еще он умеет делать? Datalist, он умеет делать Datalist. Ты не знаешь что это? Окааай... знаешь, типа иногда можно дать людям не только выбор из списка, но еще и позволить им ввести что-то в ручную? Вот это и есть datalist. С Former'ом такие вещи делаются на раз-два:
```php
Former::text('clients')->useDatalist($clients)

// Или можно опять опять же подрубить модель как делали с fromQuery()
Former::text('projects')->useDatalist(Project::all(), 'name')
```

Ты также можешь (если тебе оно надо) запилить кастомный айдишник на созданном DataList:
```php
 Former::text('foo')->list('myId')->useDatalist();
 ```
Тут он автоматически сгенерирует соответствующий `<datalist>` и привяжет его по айдишнику на это поле   
*(я вообще нихрена не понял, что он имел ввиду - п.п.)*  
Это означает, что этот самый инпут, будет заполнятся значением из твоего массива, пока чел вводит то, что он хочет, если он не найдет счастья и/или попаболи...   
*(тут он меня окончательно запутал; похоже надо самому поискать счастье и/или попаболь; читай попробовать этот даталист, чтобы понять в чем соль - п.п)*.
 

##Горячая валидация
Дальше. Окай, мнгновенная валидация - ты любишь ее, негодник, не так ли? Как ты возможно уже знаешь, все современные браузеры поддерживают валидацию по HTML атрибутам. Никакого жабаскрипта, никаких полифилов. Есть несколько атрибутов, которые могут облегчить тебе работу: pattern, required, max/min и т.д. Ты же знаешь как валидировать POST-данные через $rules? А ты не хотел бы просто передать такие правила в форму для горячей валидации? О, да, проказник - ты хотел бы... И поэтому Former это умеет. Ведь Former любит тебя и заботится о тебе.
```php
Former::open()->rules(array(
  'name'     => 'required|max:20|alpha',
  'age'      => 'between:18,24',
  'email'    => 'email',
  'show'     => 'in:batman,spiderman',
  'random'   => 'match:/[a-zA-Z]+/',
  'birthday' => 'before:1968-12-03',
  'avatar'   => 'image',
));
```
Former просмотрит эти правила и преобразует их, как сможет. Сейчас там не очень много правил, но я планирую их пополнять.
```html
<input name="name"      type="text"   required maxlength="20" pattern="[a-zA-Z]+" />
<input name="age"       type="number" min="18" max="24" />
<input name="email"     type="email" />
<input name="show"      type="text"   pattern="^(batman|spiderman)$" />
<input name="random"    type="text"   pattern="[a-zA-Z]+" />
<input name="birthday"  type="date"   max="1968-12-03" />
<input name="avatar"    type="file"   accept="image/jpeg,image/png,image/gif,image/bmp" />
```

Заметь, что ты всегда можешь сам назначить кастомные парвила, точно также, как ты добавляешь атрибуты. И если ты не шаришь в регэкспах, то это твой единственный путь.
```php
Former::number('age')->min(18)

Former::text('client_code')->pattern('[a-z]{4}[0-9]{2}')
```
И вот оно что! Это шикарная новость: с Bootstrap шарит в HTML-правилах. Просто вот так: соббщил правила, откинулся в кресле, смотришь в свой Chrome/Firefox, и наслаждаешься прекрсными всплывающими боксами "Ты должен заполнить это поле, чувак" или "Это ни разу не электронная почта, кого хотел провести?".

Ну и конечно, ты всегда можешь вручную вклининить стейт для контрол-груп в Bootstrap или Foundation. Success, warning, error and info.
```php
Former::text('name')->state('error')
```

####Обработка файлов
В Former, как и в Laravel ты легко можешь сделать  `Former::file()`. А вот что нового, так это то что ты можешь запилить мультифайловую кнопку `<input type="file" name="foo[]" multiple />`.
Еще есть один из специальных методов `->accept()` с которым можно сделать следующее:
```php
// Используя шорткаты (image, video or audio)
Former::files('avatar')->accept('image')

// Используя шорткаты длая миме-тайпов
Former::files('avatar')->accept('gif', 'jpg')

// Или буквально указывая миме-тайп
Former::files('avatar')->accept('image/jpeg', 'image/png')
```

Ты также можешь указывать макс-сайз для файлов
```php
Former::file('foo')->max(2, 'MB')
Former::file('foo')->max(400, 'Ko')
Former::file('foo')->max(1, 'TB')
```
Это создаст скрытое поле `MAX_FILE_SIZE` с правильным размером в байтах.

####Чекбоксы и Радио
Чекбоксы и радио, мужик, кого они не бесят? А потом ты такой: "бляха муха, а как все это валидировать?". Но с Foremer'ом это все немного проще :
```php
// Простой оффнутый чекбокс
Former::checkbox('checkme')


// отмеченный с текстом
Former::checkbox('checkme')
  ->text('YO CHECK THIS OUT')
  ->check()


// Четыре связанных чекбокса
Former::checkboxes('checkme')
  ->checkboxes('first', 'second', 'third', 'fourth')

// Они же инлайн
Former::checkboxes('checkme')
  ->checkboxes($checkboxes)->inline()

// Все тоже самое пашет для радио
Former::radios('radio')
  ->radios(array('label' => 'name', 'label' => 'name'))
  ->stacked()

// немного магии
Former::inline_checkboxes('foo')->checkboxes('foo', 'bar')
Former::stacked_radios('foo')->radios('foo', 'bar')

// Отмеченные/неотмеченные в один мув
Former::checkboxes('level')
  ->checkboxes(0, 1, 2)
  ->check(array('level_0' => true, 'level_1' => false, 'level_2' => true))

// Полная настройка
Former::radios('radio')
  ->radios(array(
    'label' => array('name' => 'foo', 'value' => 'bar', 'data-foo' => 'bar'),
    'label' => array('name' => 'foo', 'value' => 'bar', 'data-foo' => 'bar'),
  ))
```

Мотай на ус: Former дает тебе возможность, пушить отжатый флаг. Чё? Это когда ты передаешь состояние флага (checked/unchecked) в POST, а не "передаешь его или не передаешь". Это звучит логично, но это не то, как ведут себя формы по дефолту. 

Когда плишь чекаблы через checkboxes/radios() метод, по дефолту, для каждого чекабла создается уникальное имя в конец просто ставится индекс (например это может быть <input type="checkbox" name="checkme_2">). Это позволит значениям оставаться отмеченными при сабмитах.

####Хелперы локализации
Если ты пилишь мультиязычный проект, Former и тут спасет тебя. По дефолту при создании поля, если лейбл не задан Former будет использовать имя по дефолту *(не, ну я сегодня жугу на каламбуры -  п.п.)*.Что более важно, он еще и попытается перевести его.То же самое касается флажков лейюлов, легенды. Это означает, следующее:
```php
// Это
Former::label(__('validation.attributes.name'))
Former::text('name', __('validation.attributes.name'))
Former::text('name')->inlineHelp(__('help.name'))
Former::checkbox('rules')->text(__('my.translation'))
<legend>{{ __('validation.attributes.mylegend') }}</legend>

// Тоже самое, что это
Former::label('name')
Former::text('name')
Former::text('name')->inlineHelp('help.name')
Former::checkbox('rules')->text('my.translation')
Former::legend('mylegend')
```
Как видно, это довльно круто. Former сначала сам попытается перевести строку, т.е. `my.text` вернет `__('my.text')`, а если оно зафелится, это значит, что он не нашел то, что искал, там где искал. Ты также можешь ткнуть Former носом, непосредственно туда, где нужноискать - ну чтоб он уж наверняка не промахнулся. `Former::setOption('translate_from', [boolean])` (по умолчанию он ищет validation.attributes). Засеки - это должен быть массив.


####Примечания по порядку установки значения полей
Все классы форм сталкиваются с одной и той же проблемой: какие данные должны быть в приоритете? Для заполнения полей в Former'e установен следующий порядок:

1. POST-данные имеют самый высокий приоритет - если пользователь просто ввел что-то в поле, скорее всего, это то, что он захочет увидеть в следующий раз.
2. Затем идет заполнение данных данных с помощью  метода `->forceValue()` - это хитрый форк от `->value()`, созданный, чтобы перезаписать значение, и пофиг все.
3. После идут любые значения, установленные с помощью `Former::populate()`
4. Наконец обчный `->value()` получает самый низкий приоритет.

*(тут у меня пальцы отсохли; думаю спеки по методам и на буржуйском почитать можно - п.п. )*
