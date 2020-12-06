## 1. 3xtR4WeebTask (web)
### Условие:
```
Мой знакомый наткнулся на странный сайт, ему рассказывали, что зашедших на него людей приследовали неудачи, помогите разобраться в этом
```
http://web.ctf.msk.ru/ctn2021/prequals/3xtR4WeebTask/
### Как решать?
Переходим на сайт и видим красивого котика. Не заметив больше ничего интересного сразу открываем [html код](http://htmlbook.ru/samhtml/struktura-html-koda) страницы <br>

![](https://storage.geekclass.ru/images/549e53cc-1769-4b0c-8221-637fd778021c.png)
Замечаем флаг в тэге `tittle`, но он оказывается неверным. (на что немного указывает содержимое - `search_better`) <br>
Продолжаем изучать html и не видим больше ничего интересного кроме тэга `style` и комментария в нем - `/* ahh beautiful white background */`. Интересно, зачем на сайте белый фон если по умолчанию он и так белый. Что-то явно не так)

Открываем вкладку `Sources` в хроме и смотрим на изображение. Оно закодировано в формате `base64`, а значит пытаемся сразу его декодировать. Например, через [Cyberchef](https://gchq.github.io/CyberChef/)

![](https://storage.geekclass.ru/images/01960aa4-dba9-496d-9f24-383abf29a4e4.png)

![](https://storage.geekclass.ru/images/3049c216-1d46-4818-aae4-23ee14ad7788.png)

Видим, что в конце изображения приписан флаг. 

## 2. Direct Message (web)
### Условие:

```
Прямо таки сказать, тут ничего не понятно.
```
http://tasks.ctn2021.ctf.msk.ru:5005
### Как решать?
Переходим на сайт, видим обычную картинку на фоне, в html тоже ничего интересного.
Но если внимательно присмотреться к адресной строке, заметим что в конце приписан символ `%7D`, который при декодировании (см. сноску) превращается в символ `}`. Хм, интересно.

> Обратите внимание, что в адресной строке используется так называемый [url encoding](https://wm-school.ru/html/html_urlencode.html)

![](https://storage.geekclass.ru/images/3ff0cea3-d4d9-4139-8c3c-f6a5d72a6d28.png)

Давайте посмотрим какие запросы происходили во время загрузки сайта. Для этого нам поможет вкладочка `Network`.

> Не забудьте поставить галочки напротив `Preserve log` и `Disable cache`, они помогают не потерять нужное

![](https://storage.geekclass.ru/images/e5902cd6-8332-4c89-a3ed-36f1115d479e.png)

Изучив историю запросов, понимаем, что сайт использовал редирект (код `302`) и перенаправлял наши запросы из корня сайта(`/`) по заданным путям (сначала `/C` потом `/Y` и так далее до `\}`). Не сложно заметить, что из этих путей и складывается наш флаг.

Перепечатываем ручками флаг и не забываем провести `url decoding` всех спецсимволов.

```
%7B -> {
%7D -> }
```

## 3. Robots not allowed (web)
### Условие:
```
Говорят, что на этот сайт не пускают роботов
```
http://tasks.ctn2021.ctf.msk.ru:5001 

### Как решать?

Заходим на сайт, снова ничего интересного. Но название таска и картинка на фоне намекают нам глянуть файлик `robots.txt`

> Что такое [robots.txt](https://netpeak.net/ru/blog/chto-takoe-robots-txt-i-zachem-on-voobshche-nuzhen/)

Открываем файлик и видим кучу ссылок по которым роботам запрещено переходить. Отлично, скорее всего флаг находится по одной из ней.

![](https://storage.geekclass.ru/images/c546cc15-fc14-4ee9-bc45-a5b5bf2419df.png)

Теперь у нас есть два пути:
1) руками перебрать все пути (их всего 100)
2) написать скрипт на питоне
3) ~~запустить авто скан в Owasp Zap или Burp suite~~ (слишком просто)

Я легких путей не ищу, поэтому вот скрипт:
```
import requests

# указываем адрес таска
HOST = "http://tasks.ctn2021.ctf.msk.ru:5001"

# запрашиваем гет запросом файл /robots.txt
data = requests.get(HOST + "/robots.txt").text

# разделяем полученный текст на строчки по символу перевода строки
rows = data.split("\n")

# цикл для каждой строчки кроме первой (rows[1:])
for row in rows[1:]:

    # разделяем строчку на две части по пробелу
    trash, link = row.split()
    
    # делаем запрос на полученный путь
    response = requests.get(HOST+link).text

    # если в ответе есть префикс флага выводим его и прерываем цикл
    if "CYBERTHON" in response:
        print(response)
        break
```

> библиотеки requests нет в питоне по умолчанию, но она очень удобная. Ее можно установить при помощи `%python% -m pip install requests`, где вместо `%python%` ваша ссылка на питон (например `python2` или `python3`)

Запускаем код и получаем флаг


## 4. Have you done yor homework (stego)
### Условие:
```
На компьютере одного из учеников нашли странный файл. Какая сейчас сложная домашка!
```
https://yadi.sk/d/hEU8DvY2hLVk-Q 
### Как решать?

Скачиваем изображение, видим на картинке какие-то страшные математические символы (кошмар студентов первого курса, `предел` и `интеграл` собственной персоной)

![](https://storage.geekclass.ru/images/717780ae-46af-426a-889c-51d20b8faa33.png)

Замечаем, что картинка весит довольного много (аж `26 Мб`) для простого черного текста на белом фоне. Похоже как будто в ней что-то есть. Для подтверждения наших опасений запустим `binwalk`:

![](https://storage.geekclass.ru/images/2ed6688c-8fe8-4a4b-8d5d-9e175abc4755.png)

Понимаем, что к картинке пришит `zip архив`(да еще и с аниме!). Можно вытащить его тем же binwalk'ом, а можно поступить проще. Просто переименовать расширение с `.gif` на `.zip ` и открыть любым архиватором, например с помощью WinRar:

![](https://storage.geekclass.ru/images/77de8150-059b-45ad-8678-8832e0080fb9.png)

Сразу кликаем на `flag.txt`, но вот незадача, от нас требуется пароль. Попробовав классику в виде `12345` и `qwerty` и не получив должного результата, вспоминаем про само изображение. На нем какое-то математическое выражение, может быть решив его мы получим пароль? Дальше дело за малым, решаем предел и интеграл:

![](https://storage.geekclass.ru/images/77f959fd-bb02-41da-a041-b4c03e49f6ac.jpg)

(шутка) <br>
Можно воспользоваться готовыми решениями, например приложением `photomath` (только не говорите, что о нем не знаете) или любым сайтом для решения интегралов\пределов, ультимативно -  `WolframAlpha` (там полно примеров, так что забить туда выражение не составляет особой сложности)

Получив пароль читаем `flag.txt` и находим флаг

### Вариант № 2 (для ленивых гуманитариев Xd)

Просто брутфорсим пароль, используя `John The Ripper`
```
zip2john anime.zip > output.txt
john --format=zip output.txt
```

## 5. Best Extension (web)
### Условие:
```
Мы получили доступ к странному файлу. Интересно, для чего он использовался?
```
http://web.ctf.msk.ru/ctn2021/prequals/best_extension/best_extension.crx 
### Как решать?

Получаем файл с непонятным расширением `.crx`, сразу гуглим и понимаем, что это файл хромовского расширения для браузера.

или просто используем `file`:
```
$ file best_extension.crx
best_extension.crx: Google Chrome extension, version 3
```

Пробуем импортировать расширение в хром, но не получается (в новых версиях хрома похоже запрещено импортировать неофициальные расширения).Однако хромовские расширения работают и в других браузерах, например в `Opera`. Запустив расширение получим (а не скажу что, сами попробуйте)

Продолжив изучения `.crx` формата легко заметить, что это просто запакованный `zip` архив. Используем тот же прием, что и в прошлой задач
е, переименовываем с `.crx` на `.zip` и смотрим что за файлы в архиве.

`manifest.json` содержит мета-информацию о расширении и нам не особо полезен, а вот `background.js` представляет заметный интерес.

Сразу обращаем внимание на:
```
const ip_port = "://185.4.73.35:5002/";
и
chrome.webRequest.onBeforeSendHeaders.addListener(
    ...
    for (let s of shuffle(Object.keys(song))){
      details.requestHeaders.splice(details.requestHeaders.length, 0, {"name":song[s], "value":s});
    }
    ...
)
```

`webRequest.onBeforeSendHeaders` и дальнейший код намекают нам, что перед отправкой запроса как-то изменяются заголовки. Так же вероятно они проверяются на сайте с указанным `ip_port`, не зря же он прописан в коде.

Сделав обычный GET запрос на заданный адрес получаем ответ: `you're too blind to see`, что подтверждает наши догадки.

Изучая javascript код, понимаем что перед каждым запросом добавляются 6 заголовков (`S1`, `S2`, `S3...`), отсортированных в случайном порядке (`shuffle(Object.keys(song)`), причем значения заголовков - строки одной очень известной песни. Загуглив правильный порядок слов (на что намекает комментарий - `// forgot right order err... probably this way...`) и руками отправив запрос получаем флаг.

Вот скрипт на питоне, который это делает:
```
import requests

HOST = "http://185.4.73.35:5002"

song = '''Never gonna give you up
Never gonna let you down
Never gonna run around and desert you
Never gonna make you cry
Never gonna say goodbye
Never gonna tell a lie and hurt you'''.split("\n")

# словарь можно было бы просто создать руками - {"S1":"Never...up", "S2":...}
custom_headers = {
    key:value for key,value in zip(["S1", "S2", "S3", "S4", "S5", "S6"], song)
    }

flag = requests.get(HOST, headers=custom_headers).text

print(flag)
```

## 6. Way out of poverty (ppc)
### Условие:
```
Стэнли - ужасно неквалифицированный работник, решающий ужасно унылые задачи на работе.
Но у Стэнли есть мечта - переехать в Калифорнию. Помоги Стэнли осуществить его мечту.
```
nc tasks.ctn2021.ctf.msk.ru 5004
### Как решать?

Используя утлиту netcat, заходим по указанному адресу и смотрим, что нам прислал сервер:

![](https://storage.geekclass.ru/images/7a38288d-fd3a-4687-b826-c033d40ad547.png)

Ага, довольно простой текстовый квест. Решая математический пример каждый день нужно заработать на билет в Калифорнию. Не стоит забывать покупать еды и воды, а то ничем хорошим это не закончится. (Опытным путем выясняем, что покупать воду надо где-то каждые 3 дня, а еду - каждые 7) Можно также вложиться в прибыльную схему, но надо будет подгадать правильное время, иначе вам вместо солнышка на пляже будет светить тюрьма:)

![](https://storage.geekclass.ru/images/bc99526e-a600-46bd-a349-814f77973375.png)


![](https://storage.geekclass.ru/images/ac1c28ed-80ef-4692-91bc-7902479287ba.png)
*Какая скучная у Стэнли работа...*

Пишем код на питоне, который будет автоматизировать наши действия:

```
тут будет код
```

Запускаем код, немного ждем и получаем флаг. При большом терпении можно было решить таск и руками)

> netcat можно установить при помощи пакетного менеджера, например: `sudo apt install netcat`

## 7. Hackers Rampage (osint)
### Условие:
```
В призах за кибертон прошлого года в одной из наклеек пропала буква R. Мы сразу инициировали расследование и выяснили, что типография, в которой печатались стикеры была атакована. Пароль внутренней админской учетки был изменен, а на внешнем сайте появилась надпись: "Здесь были Хакеры Большого Свинорья". Наш оперативный отдел навел справки об этой APT группировке и выяснил, что они атакуют все ресурсы как-либо связанные с селом Большое Свинорье, а также пользуются прогрессивными методами коммуникации, основанными на геолокации.

Попробуйте обнаружить данную группировку и определить изменённый пароль админской записи. (работа типографии сейчас парализована)
Формат флага: CYBERTHON{новый_пароль}

HINT 1: ***а также пользуются прогрессивными методами коммуникации, основанными на геолокации(!)***

HINT 2:
http://web.ctf.msk.ru/ctn2021/prequals/HackersRampage/img.jpg
```
### Как решать?

Задача решалась достаточно просто, особенно после двух хинтов.
В таске нужно было обратить внимание на последнее предложение:
```
а также пользуются прогрессивными методами коммуникации, основанными на геолокации.
```

Немного поразмыслив можно было свести все к поиску соц-сетей\мессенджеров с возможностью общения по геолокации. Правильным вариантом здесь были геочаты в телеграме. (Я видел что кто-то искал в Инстаграме и Тиндере, но вы серьезно думайте что такая крупная APT группировка как `Хакеры Большого Свинорья` будет переписываться там?)))

Итак, варианты решения:
1) Быстрый и Дорогой - заказываем такси до Большого Свинорья. Место большим спросом не пользуется (а зря!) поэтому доехать туда можно всего за тысячу-две и один-два часа.
2) Медленный и Дешевый - добраться до Большого Свинорья можно на метро и автобусе. Это обойдется всего в ~ 200 рублей (туда и обратно) и займет всего пару часов вашего времени.
3) Бюджетный - Для желающих увидеть все красоты Большого Свинорья и близлежащей местности при минимуме денежных затрат. Всего ~8 часов туда и столько же обратно. Вполне можно уложиться до конца соревнований. Для любителей велоспорта маршрут займет в два раза меньше времени. (Отличные виды на черничные и клубничные поля включены)

>Лично я человек довольно занятой, поэтому пришлось выбрать первый вариант для создания таска. Зато с комфортом!

Приехав в пункт назначения открываем геочаты и находим чат хакеров Большого Свинорья:

![](https://storage.geekclass.ru/images/b04e7cb7-5faf-46e8-9d99-10325f10f695.jpg)

Здесь же и указан новый пароль для админской учетки типографии.