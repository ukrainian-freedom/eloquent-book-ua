{{meta {code_links: [«code/skillsharing.zip»]}}}

# Проект: Веб-сайт для обміну навичками

{{quote {author: 

Якщо ви володієте знаннями, дозвольте іншим запалити свої свічки.

quote}}

{{index «skill-sharing project», meetup, «project chapter»}}

{{figure {url: «img/chapter_picture_21.jpg», alt: «Ілюстрація із зображенням двох одноколісних велосипедів, притулених до поштової скриньки», »chapter: «Обрамлення"}}}.

Зустріч _((обмін навичками))_ - це захід, на якому люди зі спільними інтересами збираються разом і роблять невеликі, неформальні презентації про те, що вони знають. На зустрічі з обміну навичками в садівництві хтось може пояснити, як вирощувати селеру. Або в групі з обміну навичками програмування ви можете зайти і розповісти людям про Node.js.

У цьому останньому розділі проекту наша мета - створити ((веб-сайт)) для управління ((розмовами)), що відбуваються на зустрічах з обміну навичками. Уявіть собі невелику групу людей, які регулярно зустрічаються в офісі одного з учасників, щоб поговорити про ((одноколісний велосипед)). Попередній організатор зустрічей переїхав до іншого міста, і ніхто не зголосився взяти на себе це завдання. Ми хочемо систему, яка дозволить учасникам пропонувати та обговорювати теми для розмов між собою без активного організатора.

[Як і в [попередньому розділі](node), частина коду в цьому розділі написана для Node.js, і запустити його безпосередньо на HTML-сторінці, яку ви переглядаєте, навряд чи вийде]{якщо інтерактивний}. Повний код проекту можна ((завантажити))отримати з [_https://eloquentjavascript.net/code/skillsharing.zip_](https://eloquentjavascript.net/code/skillsharing.zip).

## Дизайн

{{index «skill-sharing project», persistence}}

Цей проект складається з _((серверної))_ частини, написаної для ((Node.js)), та _((клієнтської))_ частини, написаної для ((браузера)). Сервер зберігає дані системи і надає їх клієнту. Він також обслуговує файли, які реалізують систему на стороні клієнта.

{{index [HTTP, client]}}

Сервер зберігає список ((доповідей)), запропонованих для наступної зустрічі, а клієнт показує цей список. Кожна доповідь має ім'я доповідача, назву, короткий зміст і масив ((коментарів)), пов'язаних з нею. Клієнт дозволяє користувачам пропонувати нові доповіді (додаючи їх до списку), видаляти доповіді та коментувати існуючі. Кожного разу, коли користувач вносить такі зміни, клієнт робить HTTP-запит, щоб повідомити про це серверу.

{{figure {url: «img/skillsharing.png», alt: «Скріншот веб-сайту обміну навичками», width: “10cm”}}}}

{{index «live view», «user experience», «pushing data», connection}}

((додаток)) буде налаштовано так, щоб показувати _живий_ перегляд поточних запропонованих доповідей і коментарів до них. Щоразу, коли хтось, десь, подає нове обговорення або додає коментар, всі люди, у яких відкрита сторінка в браузері, повинні негайно побачити зміни. Це створює певні труднощі - веб-сервер не може відкрити з'єднання з клієнтом, а також не існує надійного способу дізнатися, які саме клієнти в даний момент переглядають даний вебсайт.

{{index «Node.js»}}

Загальне рішення цієї проблеми називається _((довге опитування))_, що стало одним з мотивів для розробки Node.

## Довге опитування

{{сповіщення про індекс, «довге опитування», мережа, [браузер, безпека]}}

Щоб мати змогу негайно сповістити клієнта про те, що щось змінилося, нам потрібне ((з'єднання)) з цим клієнтом. Оскільки веб-браузери традиційно не приймають з'єднання, а клієнти часто знаходяться за ((маршрутизатором)), який у будь-якому випадку блокує такі з'єднання, використання сервера для ініціювання такого з'єднання не є практичним.

{{індексний сокет}}

Ми можемо зробити так, щоб клієнт відкривав з'єднання і підтримував його, щоб сервер міг використовувати його для надсилання інформації, коли йому це потрібно. Але запит ((HTTP)) дозволяє лише простий потік інформації: клієнт надсилає запит, сервер повертається з єдиною відповіддю, і це все. Технологія під назвою _((WebSockets))_ дозволяє відкривати ((з'єднання)) _ для довільного обміну даними, але правильно використовувати такі сокети дещо складно.

У цій главі ми використовуємо простішу техніку, ((довге опитування)), де клієнти постійно запитують у сервера нову інформацію за допомогою звичайних HTTP-запитів, а сервер призупиняє відповідь, коли не має що повідомити нового.

{{index «live view»}}

Якщо клієнт постійно тримає запит на опитування відкритим, він отримуватиме інформацію від сервера швидко, як тільки вона стане доступною. Наприклад, якщо у Фатми в браузері відкритий наш додаток для обміну навичками, цей браузер зробить запит на оновлення і буде чекати на відповідь на цей запит. Коли Іман надішле доповідь на тему «Екстремальний спуск на одноколісному велосипеді», сервер помітить, що Фатма чекає на оновлення, і надішле відповідь з новою доповіддю на її запит, що очікує на відповідь. Браузер Фатми отримає дані і оновить екран, щоб показати доповідь.

{{надійність індексу, таймаут}}

Щоб запобігти тайм-ауту з'єднання (перериванню через відсутність активності), методи ((тривалого опитування)) зазвичай встановлюють максимальний час для кожного запиту, після якого сервер все одно відповість, навіть якщо йому нічого повідомити. Після цього клієнт може почати новий запит. Періодичний перезапуск запиту також робить метод більш надійним, дозволяючи клієнтам відновлюватися після тимчасових ((з'єднання)) збоїв або проблем з сервером.

{{index «Node.js»}}

На завантаженому сервері, який використовує довгі опитування, можуть бути тисячі запитів, що очікують на відповідь, і, відповідно, відкритих ((TCP)) з'єднань. Node, який дозволяє легко керувати багатьма з'єднаннями без створення окремого потоку управління для кожного з них, добре підходить для такої системи.

## Інтерфейс HTTP

{{index «skill-sharing project», [interface, HTTP]}}

Перш ніж ми почнемо проектувати сервер або клієнт, давайте подумаємо про точку, де вони стикаються: інтерфейс ((HTTP)), через який вони спілкуються.

{{index [шлях, URL], [метод, HTTP]}}

Ми будемо використовувати ((JSON)) як формат нашого запиту і тіла відповіді. Як і у файловому сервері з [Глава ?](node#file_server), ми спробуємо добре використовувати методи HTTP і ((header))s. Інтерфейс зосереджено навколо шляху `/talks`. Шляхи, які не починаються з `/talks`, будуть використовуватися для обслуговування ((статичного файлу))s - HTML та JavaScript коду для клієнтської частини системи.

{{index «Метод GET»}}

Запит `GET` до `/talks` повертає такий JSON-документ:

```{lang: «json"}
[{{«title»: «Unituning»,
  «presenter": «Jamal»,
  «summary": «Модифікація вашого циклу для додаткового стилю»,
  «comments": []}]
```

{{index «PUT method», URL}}

Створення нової теки виконується за допомогою запиту `PUT` до URL на кшталт `/talks/Unituning`, де частина після другої косої риски - це назва теки. Тіло запиту `PUT` має містити об'єкт ((JSON)), який має властивості `presenter` та `summary`.

{{index «encodeURIComponent function», [escaping, «in URLs»], [whitespace, «in URLs»]}}

Оскільки заголовки розмов можуть містити пробіли та інші символи, які можуть не відображатися у звичайній URL-адресі, при створенні такої URL-адреси рядки заголовків слід кодувати за допомогою функції `encodeURIComponent`.

```
console.log(«/talks/» + encodeURIComponent(«How to Idle»));
// → /talks/How%20to%20Idle
```

Запит на створення тейлу про байдикування може виглядати приблизно так:

```{lang: http}
PUT /talks/How%20to%20Idle HTTP/1.1
Content-Type: application/json
Content-Length: 92

{«ведучий»: «Maureen»,
 «summary": «Стоячи на місці на одноколісному велосипеді"}
```

Такі URL-адреси також підтримують запити `GET` для отримання JSON-представлення бесіди та запити `DELETE` для видалення бесіди.

{{index «POST метод»}}

Додавання ((коментаря)) до теки виконується за допомогою `POST`-запиту на URL типу `/talks/Unituning/comments`, з тілом JSON, що має властивості `author` та `message`.

```{lang: http}
POST /talks/Unituning/comments HTTP/1.1
Content-Type: application/json
Content-Length: 72

{«author»: «Iman»,
 «message": «Чи будете ви говорити про підняття циклу?"}
```

{{index «рядок запиту», timeout, «заголовок ETag», «заголовок If-None-Match»}}

Для підтримки ((довгого опитування)), `GET` запити до `/talks` можуть містити додаткові заголовки, які інформують сервер про затримку відповіді у разі відсутності нової інформації. Ми використаємо пару заголовків, які зазвичай призначені для керування кешуванням: `ETag` та `If-None-Match`.

{{index «304 (код стану HTTP)»}}

Сервери можуть включати у відповідь заголовок `ETag` («тег сутності»). Його значенням є рядок, який ідентифікує поточну версію ресурсу. Клієнти, коли вони пізніше знову запитують цей ресурс, можуть зробити _((умовний запит))_, включивши заголовок `If-None-Match`, значенням якого є той самий рядок. Якщо ресурс не змінився, сервер відповість кодом стану 304, що означає «не змінено», повідомляючи клієнту, що його кешована версія все ще актуальна. Якщо тег не збігається, сервер відповідає як зазвичай.

{{index «Prefer header»}}

Нам потрібно щось подібне, де клієнт може повідомити серверу, яку версію списку розмов він має, а сервер відповідатиме лише тоді, коли цей список зміниться. Але замість того, щоб негайно повертати відповідь 304, сервер повинен затримати відповідь і повернутися лише тоді, коли з'явиться щось нове або пройде певний проміжок часу. Щоб відрізнити довгі запити-опитування від звичайних умовних запитів, ми додамо їм ще один заголовок, `Prefer: wait=90`, який повідомляє серверу, що клієнт готовий почекати на відповідь до 90 секунд.

Сервер зберігатиме номер версії, який він оновлюватиме щоразу, коли змінюються переговори, і використовуватиме його як значення `ETag`. Клієнти можуть робити подібні запити, щоб отримувати сповіщення про зміну переговорів:

```{lang: null}
GET /talks HTTP/1.1
If-None-Match: «4»
Бажано: wait=90

(минає час)

HTTP/1.1 200 OK
Content-Type: application/json
ETag: «5»
Content-Length: 295

[...]
```

{{індекс безпеки}}

Описаний тут протокол не робить ніякого ((контролю доступу)). Кожен може коментувати, змінювати бесіди і навіть видаляти їх. (Оскільки в інтернеті повно ((хуліганів)), розміщення такої системи у мережі без додаткового захисту, ймовірно, нічим добрим не закінчиться).

## Сервер

{{index «skill-sharing project»}}

Почнемо зі створення ((серверної))-частини програми. Код у цьому розділі виконується на ((Node.js)).

### Маршрутизація

{{index «createServer function», [шлях, URL], [метод, HTTP]}}

Наш сервер буде використовувати функцію `createServer` Node для запуску HTTP-сервера. У функції, яка обробляє новий запит, ми повинні розрізняти різні типи запитів (визначені методом і шляхом), які ми підтримуємо. Це можна зробити за допомогою довгого ланцюжка операторів `if`, але є і більш простий спосіб.

{{index dispatch}}

Маршрутизатор _((router))_ - це компонент, який допомагає надіслати запит до функції, яка може його обробити. Наприклад, ви можете вказати маршрутизатору, що запити `PUT` зі шляхом, який відповідає регулярному виразу `/^\/talks\/([^\/]+)$/` (`/talks/` з наступною назвою розмови) можуть бути оброблені даною функцією. Крім того, вона може допомогти витягти значущі частини шляху (у цьому випадку назву теки), загорнуті у круглі дужки у ((регулярний вираз)), і передати їх до функції-обробника.

Існує декілька хороших пакунків маршрутизаторів на ((NPM)), але тут ми напишемо один з них самі, щоб проілюструвати принцип.

{{index «import keyword», «Router class», module}}

Це `router.mjs, який ми пізніше «імпортуємо» з нашого серверного модуля:

```{includeCode: «>code/skillsharing/router.mjs"}}
export class Router {
  constructor() {
    this.routes = [];
  }
  add(метод, url, обробник) {
    this.routes.push({метод, url, обробник});
  }
  async resolve(request, context) {
    let {pathname} = new URL(request.url, «http://d»);
    for (let {method, url, handler} of this.routes) {
      let match = url.exec(pathname);
      if (!match || request.method != method) continue;
      let parts = match.slice(1).map(decodeURIComponent);
      return handler(context, ...parts, request);
    }
  }
}
```

{{index «Router class»}}

Модуль експортує клас `Router`. Об'єкт router дозволяє реєструвати обробники для певних методів та шаблонів URL за допомогою методу `add`. Коли запит обробляється методом `resolve`, маршрутизатор викликає обробник, метод і URL якого відповідають запиту, і повертає його результат.

{{index «група захоплення», «функція декодуванняURIComponent», [екранування, «в URL-адресах»]}}

Функції-обробники викликаються зі значенням `context`, переданим `resolve`. Ми використаємо це, щоб надати їм доступ до стану нашого сервера. Крім того, вони отримують рядки відповідності для всіх груп, які вони визначили у своєму ((регулярному виразі)), і об'єкт запиту. Рядки мають бути URL-декодовані, оскільки сирий URL може містити коди у стилі `%20`.

### Обслуговування файлів

Коли запит не відповідає жодному з типів запитів, визначених у нашому маршрутизаторі, сервер повинен інтерпретувати його як запит до файлу в каталозі `public`. Для обслуговування таких файлів можна було б використати файловий сервер, визначений у [Глава ?](node#file_server), але нам не потрібно і не хочеться підтримувати запити `PUT` і `DELETE` до файлів, і ми хотіли б мати розширені можливості, такі як підтримка кешування. Давайте скористаємося надійним, добре протестованим ((статичним файловим)) сервером з ((NPM)).

{{index «createServer function», «serve-static package»}}

Я вибрав `serve-static`. Це не єдиний такий сервер на NPM, але він добре працює і підходить для наших цілей. Пакет `serve-static` експортує функцію, яку можна викликати з кореневого каталогу для створення функції-обробника запитів. Функція-обробник приймає аргументи `запит` і `відповідь`, надані сервером з `node:http`, і третій аргумент - функцію, яку буде викликано, якщо не буде знайдено жодного файлу, що відповідає запиту. Ми хочемо, щоб наш сервер спочатку перевіряв запити, які ми повинні обробляти спеціально, як визначено у маршрутизаторі, тому ми обертаємо його в іншу функцію.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
імпортуємо {createServer} з «node:http»;
import serveStatic from «serve-static»;

function notFound(request, response) {
  response.writeHead(404, «Не знайдено»);
  response.end(«<h1>Не знайдено</h1>»);
}

class SkillShareServer {
  constructor(talks) {
    this.talks = talks;
    this.version = 0
    this.waiting = [];

    let fileServer = serveStatic(«./public»);
    this.server = createServer((request, response) => {}; }; this.server = createServer((request, response) => {
      serveFromRouter(this, request, response, () => {})
        fileServer(запит, відповідь,
                   () => notFound(запит, відповідь));
      });
    });
  }
  start(port) {
    this.server.listen(port);
  }
  stop() {
    this.server.close();
  }
}
```

Функція `erveFromRouter` має такий самий інтерфейс, як і `fileServer`, і приймає аргументи `(запит, відповідь, наступний)`. Ми можемо використовувати її для «ланцюжка» з декількох обробників запитів, дозволяючи кожному з них або обробляти запит, або передавати відповідальність за нього наступному обробнику. Останній обробник, `notFound`, просто повертає помилку «не знайдено».

Наша функція ``erveFromRouter`` використовує аналогічну угоду з файловим сервером з [попереднього розділу](node) для відповідей-обробників у обіцянках повернення маршрутизатора, які перетворюються на об'єкти, що описують відповідь.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
імпортуємо {Router} з «./router.mjs»;

const router = new Router();
const defaultHeaders = {«Content-Type»: «text/plain"};

async-функція serveFromRouter(server, request,
                               відповідь, наступний) {
  let resolved = await router.resolve(request, server)
    .catch(error => {
      if (error.status != null) return error;
      return {body: String(err), status: 500};
    });
  if (!resolved) return next();
  let {body, status = 200, headers = defaultHeaders} =
    очікуємо на вирішення;
  response.writeHead(status, headers);
  response.end(body);
}
```

### Розмови як ресурси

Запропоновані ((talk))и зберігаються у властивості `talks` на сервері, об'єкті, іменами властивостей якого є заголовки розмов. Ми додамо кілька обробників до нашого маршрутизатора, які відкриють їх як HTTP ((ресурс))и у `/talks/<title>`.

{{index «метод GET», «404 (код стану HTTP)» «hasOwn function»}}

Обробник для запитів, що `GET` окрему теку, повинен шукати цю теку і відповідати або JSON-даними теки, або відповіддю про помилку 404.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
const talkPath = /^\/talks\/([^\/]+)$/;

router.add(«GET», talkPath, async (server, title) => { })
  if (Object.hasOwn(server.talks, title)) { } }
    return {body: JSON.stringify(server.talks[title]),
            headers: {«Content-Type»: «application/json"}};
  } else {
    return {status: 404, body: `Не знайдено розмови '${title}'`};
  }
});
```

{{index «метод DELETE»}}

Видалення толки відбувається шляхом видалення її з об'єкта `talks`.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}}
router.add(«DELETE», talkPath, async (server, title) => {})
  if (Object.hasOwn(server.talks, title)) { {})
    видаляємо server.talks[title];
    server.updated();
  }
  return {status: 204};
});
```

{{index «long polling», «updated method»}}

Метод `updated`, який ми визначимо [пізніше](skillsharing#updated), сповіщає про зміни запити з довгим опитуванням, що очікують на них.

{{index validation, input, «PUT method»}}

Одним з обробників, який має читати тіла запитів, є обробник `PUT`, який використовується для створення нових ((talk))s. Він має перевірити, чи дані, які йому передано, мають властивості `presenter` та `ummary`, які є рядками. Будь-які дані, що надходять ззовні системи, можуть бути нісенітницею, а ми не хочемо пошкодити нашу внутрішню модель даних або ((крах)), коли надходять погані запити.

{{index «updated method»}}

Якщо дані виглядають коректними, обробник зберігає об'єкт, який представляє нову розмову в об'єкті `talks`, можливо ((перезаписує)) існуючу розмову з такою назвою, і знову викликає `updated`.

{{index «node:stream/consumers package», JSON, «readable stream»}}

Щоб прочитати тіло з потоку запиту, ми використаємо функцію `json` з `«node:stream/consumers»`, яка збирає дані в потоці, а потім розбирає їх як JSON. У цьому пакунку є аналогічні експортні функції `text` (для зчитування вмісту у вигляді рядка) та `buffer` (для зчитування у вигляді двійкових даних). Оскільки `json` є дуже загальною назвою, імпорт перейменовує її на `readJSON`, щоб уникнути плутанини.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
import {json as readJSON} from «node:stream/consumers»;

router.add(«PUT», talkPath,
           async (server, title, request) => {})
  let talk = await readJSON(request);
  if (!talk || typeof talk.presenter !typeof talk.presenter !typeof talk.presenter !
      typeof talk.presenter != «string» ||)
      typeof talk.summary != «string») { })
    return {status: 400, body: «Bad talk data"};
  }
  server.talks[title] = {
    title,
    presenter: talk.presenter,
    summary: talk.summary,
    comments: []
  };
  server.updated();
  return {status: 204};
});
```

Додавання ((comment)) до ((talk)) працює аналогічно. Ми використовуємо `readJSON` для отримання вмісту запиту, перевіряємо отримані дані і зберігаємо їх як коментар, якщо вони виглядають коректно.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
router.add(«POST», /^\/talks\/([^\/]+)\/comments$/,
           async (server, title, request) => { })
  let comment = await readJSON(request);
  if (!comment ||)
      typeof comment.author != «string» ||)
      typeof comment.message != «string») { })
    return {status: 400, body: «Погані дані коментаря"};
  } else if (Object.hasOwn(server.talks, title)) { {
    server.talks[title].comments.push(comment);
    server.updated();
    return {status: 204};
  } else {
    return {status: 404, body: `Не знайдено жодного толку '${title}'`};
  }
});
```

{{index «404 (код статусу HTTP)»}}

Спроба додати коментар до неіснуючого обговорення повертає помилку 404.

### Підтримка довгих опитувань

Найцікавішим аспектом сервера є частина, яка обробляє ((довгі опитування)). Коли надходить запит `GET` для `/talks`, це може бути як звичайний запит, так і запит з довгим опитуванням.

{{index «talkResponse method», «ETag header»}}

Буде багато місць, в яких нам доведеться надсилати клієнту масив розмов, тому спочатку ми визначимо допоміжний метод, який створить такий масив і включить заголовок `ETag` у відповідь.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
SkillShareServer.prototype.talkResponse = function() {
  let talks = Object.keys(this.talks)
    .map(title => this.talks[title]);
  return { { body: body
    тіло: JSON.stringify(talks),
    headers: { «Content-Type»: «application/json»,
              «ETag": `«${this.version}»`,
              «Cache-Control": «no-store"}
  };
};
```

{{index «query string», «url package», parsing}}

Оброблювач повинен сам переглянути заголовки запиту, щоб побачити, чи присутні заголовки `If-None-Match` і `Prefer`. Node зберігає заголовки, імена яких визначено як нечутливі до регістру, під їхніми малими літерами.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
router.add(«GET», /^\/talks$/, async (server, request) => {})
  let tag = /«(.*)»/.exec(request.headers[«if-none-match»]);
  let wait = /\bwait=(\d+)/.exec(request.headers[«prefer»]);
  if (!tag || tag[1] != server.version) {
    повернути server.talkResponse();
  } else if (!wait) {
    return {status: 304};
  } else {
    return server.waitForChanges(Number(wait[1]));
  }
});
```

{{index «long polling», «waitForChanges method», «If-None-Match header», «Prefer header»}}

Якщо тег не було вказано або було вказано тег, який не відповідає поточній версії сервера, обробник відповідає списком розмов. Якщо запит є умовним і розмови не змінилися, ми звертаємося до заголовка `Prefer`, щоб побачити, чи слід відкласти відповідь або відповісти негайно.

{{index «304 (код статусу HTTP)», «setTimeout function», timeout, «callback function»}}

Функції зворотного виклику для відкладених запитів зберігаються в масиві `waiting` на сервері, щоб їх можна було сповістити, коли щось станеться. Метод `waitForChanges` також негайно встановлює таймер для відповіді зі статусом 304, коли запит чекає досить довго.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
SkillShareServer.prototype.waitForChanges = function(time) {
  return new Promise(resolve => {
    this.waiting.push(resolve);
    setTimeout(() => {})
      if (!this.waiting.includes(resolve)) return;
      this.waiting = this.waiting.filter(r => r != resolve);
      resolve({status: 304});
    }, time * 1000);
  });
};
```

{{index «updated method»}}

{{id updated}}

Реєстрація зміни за допомогою `updated` збільшує властивість `version` і пробуджує всі запити, що очікують.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}}
SkillShareServer.prototype.updated = function() {
  this.version++;
  let response = this.talkResponse();
  this.waiting.forEach(resolve => resolve(response));
  this.waiting = [];
};
```

{{index [HTTP, server]}}

На цьому код сервера завершується. Якщо ми створимо екземпляр `SkillShareServer` і запустимо його на порту 8000, отриманий HTTP-сервер обслуговуватиме файли з підкаталогу `public` разом з інтерфейсом керування бесідами за адресою `/talks`.

```{includeCode: «>code/skillsharing/skillsharing_server.mjs"}
new SkillShareServer({}).start(8000);
```

## Клієнт

{{index «skill-sharing project»}}

((клієнтська)) частина веб-сайту обміну навичками складається з трьох файлів: маленької HTML-сторінки, таблиці стилів та JavaScript-файлу.

### HTML

{{index «index.html»}}

Це широко розповсюджена угода для веб-серверів - намагатися обслуговувати файл з ім'ям `index.html`, коли запит робиться безпосередньо до шляху, який відповідає каталогу. Модуль ((файловий сервер)), який ми використовуємо, `erve-static`, підтримує цю домовленість. Коли запит виконується до шляху `/`, сервер шукає файл `./public/index.html` (`./public` - це корінь, який ми йому дали) і повертає цей файл, якщо він знайдений.

Таким чином, якщо ми хочемо, щоб сторінка відображалася, коли браузер вказує на наш сервер, ми повинні помістити її в `public/index.html`. Це наш індексний файл:

```{lang: «html», includeCode: «>code/skillsharing/public/index.html"}
<!doctype html>
<meta charset=«utf-8»>
<title>Обмін навичками</title>
<link rel=«stylesheet» href=«skillsharing.css»>

<h1>Обмін навичками</h1>

<script src=«skillsharing_client.js»></script></script
```

{{index CSS}}

Він визначає документ ((заголовок)) і включає таблицю стилів, яка визначає кілька стилів, щоб, серед іншого, переконатися, що між розмовами є певний пробіл. Потім він додає заголовок у верхній частині сторінки і завантажує скрипт, який містить програму на стороні ((клієнта)).

### Дії

Стан додатку складається зі списку бесід та імені користувача, і ми зберігатимемо його в об'єкті `{talks, user}`. Ми не дозволяємо користувацькому інтерфейсу безпосередньо маніпулювати станом або надсилати HTTP-запити. Натомість, він може випромінювати _дії_, які описують, що користувач намагається зробити.

{{index «handleAction function»}}

Функція `handleAction` приймає таку дію і виконує її. Оскільки наші оновлення станів дуже прості, зміни станів обробляються у тій самій функції.

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}
function handleAction(state, action) {
  if (action.type == «setUser») {
    localStorage.setItem(«userName», action.user);
    return {...state, user: action.user};
  } else if (action.type == «setTalks») { { localStorage.setTalks()
    return {...state, talks: action.talks};
  } else if (action.type == «newTalk») { return {...
    fetchOK(talkURL(action.title), {
      метод: «PUT»,
      headers: { «Content-Type»: «application/json"},
      body: JSON.stringify({
        presenter: state.user,
        summary: action.summary
      })
    }).catch(reportError);
  } else if (action.type == «deleteTalk») {
    fetchOK(talkURL(action.talk), {method: «DELETE"})
      .catch(reportError);
  } else if (action.type == «newComment») { {fetchOK(talkURL(action.talk), {метод: «deleteTalk»})
    fetchOK(talkURL(action.talk) + «/comments», {
      метод: «POST»,
      headers: { «Content-Type»: «application/json"},
      body: JSON.stringify({
        author: state.user,
        message: action.message
      })
    }).catch(reportError);
  }
  повернути state;
}
```

{{index «localStorage object»}}

Ми збережемо ім'я користувача в `localStorage`, щоб його можна було відновити при завантаженні сторінки.

{{index «fetch function», «status property»}}

Дії, які потребують залучення сервера, роблять мережеві запити, використовуючи `fetch`, до інтерфейсу HTTP, описаного раніше. Ми використовуємо функцію-обгортку `fetchOK`, яка гарантує, що повернута обіцянка буде відхилена, коли сервер поверне код помилки.

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}
function fetchOK(url, options) {
  return fetch(url, options).then(response => {
    if (response.status < 400) return response;
    else throw new Error(response.statusText);
  });
}
```

{{index «talkURL-функція», «encodeURIComponent-функція»}}

Ця допоміжна функція використовується для побудови ((URL)) для розмови із заданим заголовком.

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}}
function talkURL(title) {
  return «talks/» + encodeURIComponent(title);
}
```

{{index «error handling», «user experience», «reportError function»}}

Коли запит зазнає невдачі, ми не хочемо, щоб наша сторінка просто сиділа і нічого не робила без пояснень. Функція `reportError`, яку ми використали як обробник `catch`, показує користувачеві грубий діалог, щоб повідомити йому, що щось пішло не так.

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}
function reportError(error) {
  alert(String(error));
}
```

### Рендеринг компонентів

{{index «renderUserField function»}}

Ми використаємо підхід, подібний до того, який ми бачили у [Розділ ?](paint), розбиваючи програму на компоненти. Однак, оскільки деякі компоненти або ніколи не потребують оновлення, або завжди повністю перемальовуються при оновленні, ми визначимо їх не як класи, а як функції, що безпосередньо повертають DOM-вузол. Наприклад, ось компонент, який показує поле, де користувач може ввести своє ім'я:

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}
function renderUserField(name, dispatch) {
  return elt(«label», {}, «Ваше ім'я: “, elt(”input», {
    тип: «text»,
    value: name,
    onchange(подія) { }), onchange(подія) {
      dispatch({type: «setUser», user: event.target.value});
    }
  }));
}
```

{{index «elt function»}}

Для побудови елементів DOM використовується функція `elt`, яку ми використовували у [Главі ?](paint).

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no, hidden: true}
function elt(type, props, ...children) {
  let dom = document.createElement(type);
  if (props) Object.assign(dom, props);
  for (let child of children) {
    if (typeof child != «string») dom.appendChild(child);
    else dom.appendChild(document.createTextNode(child));
  }
  return dom;
}
```

{{index «renderTalk function»}}

Аналогічна функція використовується для рендерингу бесід, які містять список коментарів та форму для додавання нового ((comment)).

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}}
function renderTalk(talk, dispatch) {
  return elt(
    «section», {className: «talk"},
    elt(«h2», null, talk.title, « “, elt(”button», {
      type: «button»,
      onclick() {
        dispatch({type: «deleteTalk», talk: talk.title});
      }
    }, «Видалити»)),
    elt(«div», null, «by »,
        elt(«strong», null, talk.presenter)),
    elt(«p», null, talk.summary),
    ...talk.comments.map(renderComment),
    elt(«form», {
      onsubmit(event) {
        event.preventDefault();
        let form = event.target;
        dispatch({type: «newComment»,
                  talk: talk.title,
                  message: form.elements.comment.value});
        form.reset();
      }
    }, elt(«input», {type: «text», name: «comment»}), « »,
       elt(«button», {type: «submit»}, «Додати коментар»)));
}
```

{{index «submit event»}}

Обробник події `«submit»` викликає `form.reset` для очищення вмісту форми після створення дії `«newComment»`.

При створенні помірно складних фрагментів DOM такий стиль програмування починає виглядати досить безладно. Щоб уникнути цього, люди часто використовують _((мову шаблонів))_, яка дозволяє написати інтерфейс у вигляді HTML-файлу з деякими спеціальними маркерами, щоб вказати, куди йдуть динамічні елементи. Або використовують _((JSX))_, нестандартний діалект JavaScript, який дозволяє писати щось дуже близьке до тегів HTML у вашій програмі так, ніби це вирази JavaScript. Обидва ці підходи використовують додаткові інструменти для попередньої обробки коду перед виконанням, чого ми будемо уникати в цій главі.

Коментарі просто відображати.

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}
function renderComment(comment) {
  return elt(«p», {className: «comment"},
             elt(«strong», null, comment.author),
             «: », comment.message);
}
```

{{index «форма (HTML-тег)», «функція renderTalkForm»}}

Нарешті, форма, яку користувач може використати для створення нового толку, рендериться таким чином:

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}}
function renderTalkForm(dispatch) {
  let title = elt(«input», {type: «text»});
  let summary = elt(«input», {type: «text»});
  return elt(«form», {
    onsubmit(event) {
      event.preventDefault();
      dispatch({type: «newTalk»,
                title: title.value,
                summary: summary.value});
      event.target.reset();
    }
  }, elt(«h3», null, «Подати доповідь»),
     elt(«label», null, «Назва: », title),
     elt(«label», null, «Резюме: », summary),
     elt(«button», {type: «submit»}, «Submit»));
}
```

### Опитування

{{index «pollTalks function», «long polling», «If-None-Match header», «Prefer header», «fetch function»}}

Для запуску програми нам потрібен поточний список токенів. Оскільки початкове завантаження тісно пов'язане з довгим процесом опитування - `ETag` з завантаження повинен використовуватися при опитуванні - ми напишемо функцію, яка продовжує опитувати сервер на предмет `/talks` і викликає ((функцію зворотного виклику)), коли з'являється новий набір розмов.

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}}
async function pollTalks(update) {
  let tag = undefined;
  for (;;) {
    let response;
    try {
      response = await fetchOK(«/talks», {
        headers: tag && {«If-None-Match»: tag,
                         «Prefer": «wait=90"}
      });
    } catch (e) {
      console.log("Запит не виконано: » + e);
      wait new Promise(resolve => setTimeout(resolve, 500));
      continue;
    }
    if (response.status == 304) continue;
    tag = response.headers.get(«ETag»);
    update(await response.json());
  }
}
```

{{index «async function»}}

Це `async`-функція, щоб спростити циклічне виконання та очікування запиту. Вона запускає нескінченний цикл, який на кожній ітерації отримує список розмов - або у звичайному вигляді, або, якщо це не перший запит, із заголовками, які роблять його довгим запитом на опитування.

{{index «обробка помилок», «клас обіцянки», «функція setTimeout»}}

Коли запит завершується невдало, функція чекає деякий час, а потім повторює спробу. Таким чином, якщо ваше мережеве з'єднання пропадає на деякий час, а потім повертається, програма може відновитися і продовжити оновлення. Обіцянка, оброблена за допомогою `setTimeout`, є способом змусити функцію `async` чекати.

{{index «304 (код стану HTTP)», «заголовок ETag»}}

Коли сервер повертає відповідь 304, це означає, що тривалий запит на опитування вичерпано, тому функція повинна просто негайно почати наступний запит. Якщо відповідь є звичайною 200 відповіддю, її тіло зчитується як ((JSON)) і передається у функцію зворотного виклику, а значення заголовка `ETag` зберігається для наступної ітерації.

### Додаток

{{index «SkillShareApp class»}}

Наступний компонент пов'язує весь інтерфейс користувача разом:

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}}
class SkillShareApp {
  constructor(state, dispatch) {
    this.dispatch = dispatch;
    this.talkDOM = elt(«div», {className: «talks»});
    this.dom = elt(«div», null,
                   renderUserField(state.user, dispatch),
                   this.talkDOM,
                   renderTalkForm(dispatch));
    this.syncState(state);
  }

  syncState(state) { } }
    if (state.talks != this.talks) {
      this.talkDOM.textContent = «»;
      for (let talk of state.talks) {
        this.talkDOM.appendChild(
          renderTalk(talk, this.dispatch));
      }
      this.talks = state.talks;
    }
  }
}
```

{{index synchronization, «live view»}}

Коли розмови змінюються, цей компонент перемальовує їх усі. Це просто, але водночас марнотратно. Ми повернемося до цього у вправах.

Ми можемо запустити програму таким чином:

```{includeCode: «>code/skillsharing/public/skillsharing_client.js», test: no}
function runApp() {
  let user = localStorage.getItem(«userName») || «Anon»;
  let state, app;
  function dispatch(action) {
    state = handleAction(state, action);
    app.syncState(state);
  }

  pollTalks(talks => {
    if (!app) {
      state = {user, talks};
      app = new SkillShareApp(state, dispatch);
      document.body.appendChild(app.dom);
    } else {
      dispatch({type: «setTalks», talks});
    }
  }).catch(reportError);
}

runApp();
```

Якщо запустити сервер і відкрити поруч два вікна браузера для [http://localhost:8000_](http://localhost:8000/), то можна побачити, що дії, які ви виконуєте в одному вікні, одразу видно в іншому.

## Вправи

{{index «Node.js», NPM}}

Наступні вправи передбачають модифікацію системи, описаної у цій главі. Щоб працювати над ними, переконайтеся, що ви ((завантажили)) код ([https://eloquentjavascript.net/code/skillsharing.zip_](https://eloquentjavascript.net/code/skillsharing.zip)), встановили Node ([https://nodejs.org_](https://nodejs.org)) та встановили залежність проекту за допомогою `npm install`.

### Збереженість даних на диску

{{index «data loss», persistence, [memory, persistence]}}

Сервер обміну навичками зберігає свої дані виключно у пам'яті. Це означає, що коли він ((аварійно)) завершує роботу або перезапускається з будь-якої причини, всі бесіди та коментарі втрачаються.

{{index «жорсткий диск»}}

Розширте сервер так, щоб він зберігав дані розмов на диску і автоматично перезавантажував їх при перезапуску. Не турбуйтеся про ефективність - зробіть найпростіше, що працює.

{{hint

{{index «файлова система», «функція writeFile», «оновлений метод», persistence}}

Найпростіше рішення, яке я можу запропонувати, це кодувати весь об'єкт `talks` як ((JSON)) і скидати його у файл за допомогою `writeFile`. Вже існує метод (`updated`), який викликається кожного разу, коли дані на сервері змінюються. Його можна розширити для запису нових даних на диск.

{{index «readFile function», «JSON.parse function»}}

Виберіть ім'я ((файлу)), наприклад `./talks.json`. Коли сервер запуститься, він може спробувати прочитати цей файл за допомогою `readFile`, і якщо це вдасться, сервер може використовувати вміст файлу як свої початкові дані.

підказка}}

### Скидання поля коментаря

{{index «скидання поля коментаря (вправа)», template, [state, «of application»]}}

Повне перемальовування бесід працює досить добре, оскільки зазвичай ви не можете відрізнити DOM-вузол від його ідентичної заміни. Але бувають винятки. Якщо ви почнете вводити щось у полі ((поле)) для коментаря в одному вікні браузера, а потім в іншому вікні додасте коментар до цієї теки, поле у першому вікні буде перемальовано, видаливши як його вміст, так і його ((фокус)).

Коли кілька людей додають коментарі одночасно, це може дратувати. Чи можете ви придумати спосіб вирішити цю проблему?

{{hint

{{index «скидання поля коментаря (вправа)», template, «метод syncState»}}

Найкращий спосіб зробити це, ймовірно, полягає у тому, щоб зробити компонент обговорення об'єктом з методом syncState, щоб його можна було оновити, щоб показати змінену версію обговорення. Під час звичайної роботи, єдиний спосіб, у який можна змінити тек - це додати більше коментарів, тому метод `syncState` може бути відносно простим.

Складність полягає у тому, що коли надходить змінений список токенів, ми повинні узгодити існуючий список DOM-компонентів з токенами у новому списку - видалити компоненти, чиї токени було видалено, і оновити компоненти, чиї токени було змінено.

{{index synchronization, «live view»}}

Для цього може бути корисним зберігати структуру даних, яка зберігає компоненти під назвами теки, щоб ви могли легко з'ясувати, чи існує компонент для даної теки. Після цього ви можете циклічно переглядати новий масив розмов і для кожної з них синхронізувати наявний компонент або створювати новий. Щоб видалити компоненти для видалених розмов, вам доведеться також пройтись по компонентах і перевірити, чи існують ще відповідні розмови.

підказка}}
