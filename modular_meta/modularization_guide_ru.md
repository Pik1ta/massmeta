# Руководство по модуляризации кода – MassMeta style, v0.3

**Соблюдение этого Гайда – залог успешного мержа Вашей фичи в репозиторий.**

## Вступление

Проект **МассМета** 🧰 – это проект основанный на базе кода [/TG/station](https://github.com/tgstation/tgstation).

В этом проекте Вы и другие игроки могут внести свой неоценимый вклад в виде добавления прикольной фичи, или например вырезания неудачной. Знающие кодеры обязательно помогут Вам в этом, на первый взгляд, непростом деле.

Преполагается, что у Вас есть первичный опыт работы с C/C++ и/или JS подобным кодом, а также с Git-подобными системами контроля версий. Будет круто, если у Вас уже есть опыт программирования на здешнем языке DM и/или опыт работы с React JS, на чем собственно и основан интерфейл TGUI.

Но как говорится, все с чего-то начинали, также как и автор этого текста. От себя дам парочку полезных советов не просто по кодингу, а вообще.

* Если ты умеешь "гуглить" - значит ты сможешь быстро находить любую свободную информацию самостоятельно и где угодно. (не только в гугле)
* Прямое общение со знающими специалистами - значительно ускорит процесс решения возникшей трудности. (но перед тем как задать вопрос - погугли и подумай сам)
* Знание английского языка снимает с тебя барьер общения и позволяет значительно увеличить эффективность двух пунктов выше.
* Правильно поставленный вопрос - уже половина решения, сделанная именно тобой. А значит специалист будет охотнее помогать. (не поверите какие они порой лентяи)
* Любая крупная задача разбивается на подзадачи. А те уже проще по одной организованно довести до победного конца.
* Не держи всю информацию в голове - записывай!
* Не держи все задачи в голове - составь чек-лист и иди по нему.
* Если ты разобрался в какой-то проблеме - сделай краткую шпаргалку для самого себя же. (и поделись с остальными)

**Слонёнок:** А когда не знаешь как, нужно у кого-нибудь спросить… – из м/ф `38 попугаев` (1976)

Что-то как-то много философии... Достаточно напутствий, теперь к делу.

Вам предстоит кодить, много кодить, порой не сразу понимать что вы кодите... Иногда смотреть в другой репозиторий и в отчаянных попытках перенести уже готовенькое смешное или полезное к себе. Понять что это забагованно и чинить, порой много чинить... Да, баги они такие.

⚠️ **Все Баг-фиксы и рефакторы немодульного кода, изменение не Наших карт – Вам нужно будет заливать именно в апстрим /TG/station !**

### Про тестирование своих [PR'ов](https://ru.stackoverflow.com/questions/134183/%D0%A7%D1%82%D0%BE-%D1%82%D0%B0%D0%BA%D0%BE%D0%B5-pull-request) 🔬

Прежде чем открывать PR на слияние, то было бы неплохо проверить Ваш код на работоспособность.

А именно:
* **Скомпилируй билд** на своей локальной машине и посмотри, что там выдал компилятор Byond. Если билд успешно скомпилился, то уже хорошо.
* **Запусти билд** с Фичей и посмотрите как она себя ведёт, если ли какие-либо аномалии в работе модуля. Достаточно поверхностной проверки.
* **Протестируй и так и эдак фичу**, например, если Ваша Фича предполагает взаимодействие нескольких игроков, то можете воспользоваться Гостевым аккаунтом. Для этого выйди из BYOND-хаба (надо закрыть Бьенд) и зайди на локалку как Guest-[много циферок].

После этого смело создавайте PR на наш репозиторий, там Вам уже подскажут как можно доработать тот или иной неочевидный момент.

### Про платформу GitHub

Это одна из многих видов Систем Контроля Версий. Сам Git одновременно и достаточно проработан в плане алгоритмов, он ими же и ограничен. Они не всегда могут однозначно самостоятельно разрешить определенные изменения в коде, что приводит к конфликтам, которые требуется решать вручную.

Подробнее про сам Git и как с ним работать в [очень хорошей книжке-мануале](https://git-scm.com/book/ru/v2).

## Суть Конфликта ⚔️

Начнём сразу с показательного примера.

Предположим, что в какой-то строчке оригинального файла `foobar.dm` /TG/station было так:

```byond
var/something = 1
```

Однако, под наши нужны нам потребовалось изменить значение с **1** на **2** под какую-то фичу в проекте,

```diff
- var/something = 1
+ var/something = 2 //MASSMETA EDIT
```

Но неожиданно апстрим /TG/station вносит свои изменения (commit: their-feature) в эту же строку файла, меняя её у себя с **1** на **4**,

```diff
- var/something = 1
+ var/something = 4
```
Затем мы решили синхронизировать изменения и видим следующее,

```byond
<<<<<< HEAD:foobar.dm
var/something = 2 //MASSMETA EDIT
======
var/something = 4
>>>>>> their-feature:foobar.dm
```

В данном случае в череду коммитов /TG/station внедряется дополнительный, про который известно только нам самим. ГитХаб, видя подобное несоответствие - даёт нам сделать выбор.

Например, нам нужно оставить только Наше изменение, то просто удаляем все что нам добавил ГитХаб и оставляем только нужное,

```byond
var/something = 2 //MASSMETA EDIT
```

Подобного рода конфликты разрешаются именно ручками, однако есть другие подходы в виде модульного кода, о которых мы расскажем далее в данном руководстве.

Подробнее про [Ветвления и Слияния](https://git-scm.com/book/ru/v2/Ветвление-в-Git-Основы-ветвления-и-слияния).

## Протокол модуляризации 🛠️

У Вашего модуля должно быть короткое и информативное название в документации, например - **`shuttle_toggle`**.

Этим уникальным **ID** Вы затем назовёте:

- Свою модульную папку `modular_meta/features/shuttle_toggle/`, в которой вы будете локально работать.

- А также в дальнейшем Вы будете помечать все **Немодульные** изменения в коде /TG/station.

- В редких случаях он может пригодится как некая метка для включения/отключения модуля. Об этом будет отдельно рассказано ниже.

Подытожим, эта метка должна присуствовать около всех изменений сделаных этим модулем.

Теперь подробнее про виды модуляризации.

### Не Модульные Изменения

Иногда наступает момент, когда редактирование **оригинальных** файлов /TG/station намного удобнее и практичее.

📌 Пожалуйста, не забудьте записать факт их изменения под пунктом **"TG Proc/File Changes"** в `readme.md` вашего модуля.

В этих случаях мы решили применять следующую стандартизацию:

- **Добавление:**

  ```byond
  //MASSMETA EDIT ADDITION BEGIN (shuttle_toggle)
  var/adminEmergencyNoRecall = FALSE
  var/lastMode = SHUTTLE_IDLE
  var/lastCallTime = 6000
  //MASSMETA EDIT ADDITION END
  ```

- **Удаление:**

  ```byond
  //MASSMETA EDIT REMOVAL BEGIN (shuttle_toggle)
  /*
  for(var/obj/docking_port/stationary/S in stationary)
      if(S.id = id)
          return S
  */
  //MASSMETA EDIT REMOVAL END
  ```

- **Изменения:**

  ```byond
  //MASSMETA EDIT CHANGE BEGIN (shuttle_toggle)
  /* ORIGINAL
  if(SHUTTLE_STRANDED, SHUTTLE_ESCAPE)
  */
  if(SHUTTLE_STRANDED, SHUTTLE_ESCAPE, SHUTTLE_DISABLED)
  //MASSMETA EDIT CHANGE END
      return 1
  ```

  💡 Если предполагается **"Масштабное"** изменений или удаление кода, то его уже можно переместить в модуль. На месте удаления обязательно допишите куда оно было перемещенно: (Moved to: `modular_meta/features/shuttle_toggle/randomverbs.dm`)
  
  ⚠️ Обязательно оставляем все что было до вашего вмешательства под пометкой **ORIGINAL**!

### Полностью Модульные Изменения

В нашем проекте присутствует папка `modular_meta/`, там будут храниться все наши **"Модульные"** изменения кода /TG/station.

💡 Она полностью независима и этим мы гарантируем, что кодеры с /TG/station не будут туда вмешиваться.

В этой папке есть ещё несколько подпапок и файлов:

| Папка/Файл                         |
| ---------------------------------- |
| **\_defines**                   📁 |
| **\_globalvars**                📁 |
| **\_helpers**                   📁 |
| **features**                    📁 |
| **reverts**                     📁 |
| **russification**               📁 |
| **tools**                       📁 |
| **\_modpacks_subsystem.dm**     📄 |
| **main_modular_include.dm**     📄 |
| **modularization_guide_ru.md**  📝 |
| **module_template.md**          📝 |

Теперь подробнее про каждую из Папок:

- **`_defines/ и _globalvars/ и _helpers/`** 📂

  Здесь лежат все наши модульные "определения" (defines), "гробальные пеменные" (globalvars) и "помощники" (helpers).

  Вынесены отдельно из папки `features/` из-за того, что их требуется ставить выше основного ТГ кода, сразу же после аналогичных у ТГ.
  
  💡 За счет этого мы можем использовать наши дефайны в коде ТГ, а не только в модулях.
  
  В целях увеличения читабельности данных папок, добавляйте максимум по 1 файлу в каждую из папкок, назовите их также как `ID` вашего модуля.
  
  Все эти файлы в папках должны быть включены в соотвесвтвующий им:
  - **`modular_meta/_defines/_main_modular_defines_include.dm`**
  - **`modular_meta/_globalvars/_main_modular_globalvars_include.dm`**
  - **`modular_meta/_helpers/_main_modular_helpers_include.dm`**

  📌 Пожалуйста, не забудьте записать факт их добавления под пунктом **"Defines/Helpers:"** в `readme.md` вашего модуля.

  💡 Если же они применяются только лишь в рамках одного файла, то их достаточно объявить вверху и внизу файла. Такое записывать в `readme.md` не надо!

  ```byond
  #define MY_DEFINE
  //some code with MY_DEFINE here
  #undef MY_DEFINE
  ```

- **`features/`** 📂

  Здесь лежат все модульные файлы **"Новых Фич"**, которых нет в апстриме. Каждой присвоен уникальный **"module_id"**.

  Подробнее про строение папок модуля расскажем чуть ниже.

- **`reverts/`** 📂

  Папка аналогичная по строению с `features/`, но как можно догадаться, там располагаются возвраты хороших фич и факты удаления плохих по нашему мнению фич, введёных апстримом /TG/station.
  
  ❗ Если фича была уже выпилена давно или же апстрим произвел её полное удаление сразу, то она уже может рассматриваться как самостоятельный модуль уже в папке `features/`!

  Такой модуль именовать обязательно с припиской **"re_"**, например: **"re_buff_lasers"**.

  ⚠️ Укажите в `readme.md` модуля ссылку на PR или серию PR-ов, который откатываевается!

- **`russification/`** 📂

  Папка аналогичная по строению с `features/`, но в ней уже лежат различные переводы на Русский всякого в игре. А-ля руссификаторы во многих других компьютерных игрушках.

  Выделена отдельно из "фич" по понятным причинам.
  
  Также если это потребуется для код-базы, то можно будет сделать сразу всю папку легко отключаемой из компиляции кода для англо-язычных форков. (про этот момент будет подробнее ниже)
  
  Такой модуль именовать уже обязательно с припиской **"ru_"**, например вот так: **ru_crayons**.

- **`tools/`** 📂

  Тут лежат все дополнительные инструменты проверки нашего кода.

  **Они проверяют только файлы в модульной папке**, помогая нам не совершать дополнительных ошибкок.

  К ним идет прямое обращение только в файле: `.github/workflows/ci_suite.yml`.

## Подробнее про наполнение папок (features/ russification/ reverts/)

Чтобы сохранить общий стиль и обеспечить удобную навигацию по большинству модулей, а также контролировать количество файлов и папок в репозитории, Вы должны располагать определённые типы файлов по своим папкам.

⚠️ Каждый модуль обязан содержать в себе файл документации модуля – `readme.md`.

| Папка/Файл         | Содержимое                                                            |
| ------------------ | --------------------------------------------------------------------- |
| **code/** 📁       | Файлы кода: **`.dm`**                                                 |
| **icons/** 📁      | Файлы иконок и картинок: **`.dmi`** и **`.png`**                      |
| **sound/** 📁      | Звуковые файлы: **`.ogg`** и **`.waw`**                               |
| **includes.dm** 📄 | Инклюд всех файлов в папке **code/** и объект модуля `/datum/modpack/`|
| **preview.dmi** 🖼️ | Картинка-превью вашего модуля, используется в меню модпаков           |
| **readme.md** 📝   | Полная документация к модулю, [пример](module_template.md)            |

⚠️ Файлы строк: `.txt` и `.json` вы помещаете в папку `strings/meta/`, т.к. код /TG/station не может полноценно работать со всеми файлами-строк вне этой папки. (если вдруг в будуем будет такая возможноть, то надо будет перенести в модуль)

⛔ У проектов **Skyrat** присуствует папка `master_files/`, однако у нас в проекте её НЕТ! Все переопредения кода у нас помещаются полностью в модуль с особыми пометками! Пояснения будут позже.

❗ Также у проектов **Skyrat** стандартно все новые файлы сразу включаются в общий файл `tgstation.dme`, что я считаю достаточно трудным для дальнейшней поддержки. У нашего проекта другой поход в этом моменте, как Вы видите.

### Подробнее про папку **`code/`**

⚠️ В этих файлах не должно быть закомментированного кода, тем более полностью закомментированных файлов! А вот пояснения к коду оставлять можно, порой даже и нужно.

Здесь располагается код двух типов:

#### Абсолютно новые функции и объекты 🆕

Просто ложите все свои новые файлы кода в папку `code/` своего модуля.

💡 Можете разбивать файлы по подпапкам, если в этом есть нужда.

💡 Если модуль предполагается неимоверно фантастических размеров, то в таком случае можете ввести иерархию папок аналогичную как в папке `code/` у /TG/station

Не забывайте проставлять все пути к иконкам и звуку правильно!

#### Переопределение объектов и функций /TG/station 🔀

С помощью этих файлов мы косвенно изменяем основной код /TG/station. Это позволяет нам очень изящно внедрять свои коррективы, не вмешиваясь напрямую в основной код. Тем самым не нарушая их череду коммитов и не создавая для нас самих в будущем **Конфликтов Слияния**.

Однако у такого подхода есть свой недостаток. Гитхаб не сможет нам оперативно подсказать где файл поменялся из-за вмешательства апстрима и где следует учесть измененое или дополнительное переопределение. Иногда прямые изменения кода через `//MASSMETA EDIT` предпочтительнее. Старайтесь использовать здравый смысл в этом вопросе.

Эти файлы выносите в "отдельную группу" с помощью пометки `"m_"` в названии (от слова master), например: `m_tg_filename.dm`.

⚠️ У малого кол-ва данных файлов **Не надо соблюдать Иерархию** аналогичной папки `code/` у /TG/station! Просто ложите вместе со всеми файлами в модуль.

💡 Однако если вдруг у модуля предполагается неимоверно огромное кол-во таких переопределений, то в таком случае можете ввести иерархию папок аналогичную как в папке `code/` у /TG/station

⚠️ Над каждым блоком таких функций/объектов **подписывайте в каком оригинальном файле /TG/station они расположены**. Таким образом нам будет проще смекнуть что к чему.

- **Пример модульного переопределения объекта** 💡

  Например, Вы решили модульно переопределить иконку и описание у мольберта (easel).

  Оригинальный объект в коде /TG/station по пути `code/modules/art/paintings.dm`:

  ```byond
  /obj/structure/easel
		name = "easel"
		desc = "Only for the finest of art!"
		icon = 'icons/obj/art/artstuff.dmi'
		icon_state = "easel"
		density = TRUE
		resistance_flags = FLAMMABLE
		max_integrity = 60
		var/obj/item/canvas/painting = null
  ```

  Для этого создайте новый файл желательно с таким же именем как у оригинала и расположите в папке `code/` вашего модуля: `code/m_paintings.dm`.

  Для выполнения нашей цели, наполнение даного файла будет выглядеть примерно так:

  ```byond
  //ORIGINAL FILE: code/modules/art/paintings.dm
  /obj/structure/easel
		desc = "Let yourself draw!"
		icon = 'modular_meta/features/art/icons/artstuff.dmi'
		icon_state = "new_easel"
  ```

  Теперь при компилировании проекта, данное переопределение подменит эти переменные у оригинального объекта мольберта. Тем самым в готовом проекте у мольберта будет уже Новая иконка и описание. Даже код /TG/station менять не пришлось! 🎉

- **Пример модульного добавления фичи в функцию** 💡

  Для простоты предположим, что вы хотите заставить оружие искрить при выстреле для имитации дульной вспышки.
  
  Также Вы хотите, чтобы это можно было использовать со всеми видами оружия, использующими эту функцию.

  В модульном файле объекта можно начать с добавления новой переменной `var/muzzle_flash`.

  ```byond
  /obj/item/gun
		var/muzzle_flash = TRUE
  ```

  Теперь у Вас будет **у каждого** наследника этого объекта доступна эта переменная. После этого, допустим, вы захотите проверять её и вызвать искры после выстрела.

  У этого объекта уже есть процедура, что вызывается при стрельбе:

  ```byond
  /obj/item/gun/proc/shoot_live_shot(mob/living/user, pointblank = 0, atom/pbtarget = null, message = 1)
  ```

  Теперь мы начинаем **добавлять код** для работы нашей фичи в дочернюю процедуру `/obj/item/gun/shoot_live_shot()`.

  ```byond
  /obj/item/gun/shoot_live_shot(mob/living/user, pointblank = 0, atom/pbtarget = null, message = 1)
		. = ..()
		if(muzzle_flash)
			spawn_sparks(src)
  ```

  Тут мы обязательно вызываем такую конструкцию `. = ..()`, она по сути своей говорит, что мы **наследуемся от родительской функции**. После данной конструкции добаляем весь наш новый код.

  Теперь при компилировании проекта, данные "добавления" допишутся в оригинальный объект и функцию. Этим мы добились того, что оружие при выстреле ещё и искрит, при том не вмешиваясь в функции /TG/station напрямую! 🎉

### Про содержание модулей ОГРОМНЫХ размеров

В очень редких случаях в проекте может появиться такая уже не очень маленьких размеров пакость, что будет эпизодически напоминаль про себя, оссобено при мержапстримах (синхронизации нашего форка с апстримом).

В таком случае папка модуля может полностью повторять иерархию папок /TG/station.

В `readme.md` укажите что файл огромен и скорее всего требует более пристального внимания при дальнейшем вмешательстве.

### **`. = ..()`** для чайников

Как вы уже могли заметить, у языка DM частично заложена парадигма [ООП](https://practicum.yandex.ru/blog/obektno-orientirovannoe-programmirovanie/), нас в данном случае интересует процесс наследования объектов и их функций.

💡 Вы также можете нажать F1 в Dream Maker и прочитать подробный мануал на английском. Или например [тут](https://github.com/F0lak/dm_open_ref).

- `.` – это возвращаемое значение нашей функции по умолчанию. Изначально оно равно `null`.

- `..` – это возвращаемое значение родительской функции.

  Через `..()` Вы обращаетесь к родительской функции, этим вызывая её в нужном месте Вашей функции-наследника.

И теперь, если вы сделаетете подобый манёвр `. = ..()`, то Вы вызовите родительскую функцию и присвоите её значение возвращаемому значению нашей функции. После этого можете свободно дополнять свою `.` чем хотите.

Так же в случае, если Вы не хотите возвращать родительский вывод, то Вы можете сохранить его в любую переменную или просто использовать `..()`

Мы так же можем вообще не вызывать `..()` - тогда функция оверрайдится (переопределяется) полностью. Но я бы не рекомендовал делать подобное, ради перемещения ориг. функции в модуль и дальнейшей модификации её уже там, ибо когда /TG/station поменяет её у себя, то пиши пропало.

Также учтите, что вы не сможете при модульном дополнии функции использовать те переменные, которые были объявленны в функции оригинале!

## Отключаемые модули

Если в папке модуля есть файл `modular_meta/__config_modpacks.dm`, то это дает нам парочку очень удобных возможностей.

А именно, в нём то перечисленны все модули, что можно отключить в игре. Для отключения требуется перекомпиляция проекта, т.к. эти изменения вступают в силу только на этапе работы [препроцессора](https://ru.wikipedia.org/wiki/%D0%9F%D1%80%D0%B5%D0%BF%D1%80%D0%BE%D1%86%D0%B5%D1%81%D1%81%D0%BE%D1%80).

Скорее всего там будут присутсвовать модули **"Перевода"** ну и некоторые другие, что были сделаны отключаемыми по желанию автора модуля. В ином случае, модуль можно убрать только если залезать в код напрямую.

Например, Вы так можете сделать отключаемым **"Ивентовый модуль"**. Пояснения излишне почему и зачем.

Однако учтите, что если таких модулей будет много, то неизбежно будут возникать взаимосвязи между ними, и порой одим модуль без другого работать не будет.

TODO: описать чуть подробнее про взаимосвязанные модули, и что нужно делать, чтобы они автоматически проверялись на инклюд.

### Процедура создания отключаемого модуля

Изначально мы имеем в оригинале код:

```byond
/obj/item/toy/crayon/proc/crayon_text_strip(text)
	text = copytext(text, 1, MAX_MESSAGE_LEN)
	var/static/regex/crayon_regex = new /regex(@"[^\w!?,.=&%#+/\-]", "ig")

	return LOWER_TEXT(crayon_regex.Replace(text, ""))
```

Но нам потребовалось изменить одну строчку кода функции, что к сожалению просто так нельзя поместить в модуль.

Для начала создадим в файле `modular_meta/__config_modpacks.dm` новый дефайн `RU_CRAYONS` и сразу присваеваем ему значение `1` = включен.

```byond
#define RU_CRAYONS 1
```

Далее заходим в интересующий нас файл и там оборачиваем изменяемый блок строчек в `#if – #else – #endif` конструкцию, где `RU_CRAYONS` – это **ID** вашего модуля.

```byond
/obj/item/toy/crayon/proc/crayon_text_strip(text)
	text = copytext(text, 1, MAX_MESSAGE_LEN)
#if RU_CRAYONS // MASSMETA EDIT
	var/static/regex/crayon_regex = new /regex(@"[^\wА-Яа-яЁё!?,.=&%#+/\-]", "ig")
#else
	var/static/regex/crayon_regex = new /regex(@"[^\w!?,.=&%#+/\-]", "ig")
#endif
	return LOWER_TEXT(crayon_regex.Replace(text, ""))
```

Когда модуль включен `1` – то будет подставляться верхняя строчка кода, когда выключен `0` – оригинальная.

⚠️ Такие измения уже содержат ID вашего модуля, поэтому достаточно их помечать только конструкцией `// MASSMETA EDIT`.

## Карты 🗺️

Используются карты:
- 🔴 Оригинальные с офф ТГ (без прямых наших изменений)
- 🟡 Заимствованные с других билдов (некоторые желательно не менять, т.к. мы можем подтягивать обновления в других билдов)
- 🟢 Наши самодельные, они же полностью независимые, например, ProtoBoxStation (меняем и чиним как хотим)

### Новые карты (наши самодельные)

Все наши Новые карты лежат по пути со всеми остальными в `_maps/map_files/` (не в модульной папке).

К каждой карте идёт дополнительно `.json` файл-конфигурации в `_maps/`, не забудьте добавить его тоже!

### Модульное Изменение Карт ТГ (через применение Авто-мапперов)

⚠️ Не изменяйте оригинальные карты /TG/station напрямую, Вы стокнётесь потом с таким же хаосом, как если вы бы меняли файлы иконок! Для внесения изменений ипользуйте модуль Авто-маппера.

Когда вы добавляете новый элемент на карту, то вы должны сперва определеить масштаб переделок.

- Если это **небольшое изменение на 1 предмет**, то используйте простой автоматизатор области.

  Автомаппер простых областей использует записи точек отсчета, чтобы поместить один элемент в область карты, которая имеет определенный смысл.
  
- Если речь идет об **изменении целой комнаты**, то используйте автоматизатор шаблонов.

  Автомаппер использует готовые шаблоны для переопределения участков карты, используя координаты для определения начального местоположения. Примеры смотрите в файле automapper_config.toml.

TODO: добавить подробный пошаговый гайд по использованию автомаппера.

## Модульный TGUI (TG User Interface)

**TGUI** - еще один исключительный случай, поскольку он использует язык Javascript, который не может быть модульным, нежели же код DM.

ВСЕ файлы TGUI находятся в папке `/tgui/packages/tgui/interfaces/` и ее подкаталогах. Нет какой-то конкретной папки для Наших TGUI файлов!

Частным примером может служить TGUI файл для оформления меню Модпаков `/tgui/packages/tgui/interfaces/Modpacks.tsx`.

📌 Пожалуйста, не забудьте записать факт их добавления/изменения под пунктом **"TGUI Files:"** в `readme.md` вашего модуля.

### Изменение оригинальных файлов /TG/station

При изменении оригинальных файлов TGUI поступаем аналогично, как и при изменении вышележащего кода DM, однако схема написания комментариев тут несколько иная.

Вы можете использовать как `// MASSMETA EDIT`, так и `/* MASSMETA EDIT */`, хотя в некоторых случаях вам придется использовать одно вместо другого. (в некотрых языках '//' - могут не являться комментированием, учтите это)

В целом, старайтесь, чтобы комментарии к изменениям находились на той же строке, что и само изменение. Предпочтительно внутри JSX-тега. Например:

```js
<Button
	onClick={() => act('spin', { high_quality: true })}
	icon="rat" // MASSMETA EDIT ADDITION
</Button>
```

```js
<Button
	onClick={() => act('spin', { high_quality: true })}
	// MASSMETA EDIT ADDITION START - another example, multiline changes
	icon="rat"
	tooltip="spin the rat."
	// MASSMETA EDIT ADDITION END
</Button>
```

```js
<SomeThing someProp="whatever" /* it also works in self-closing tags */ />
```

В крайнем случае Вы можете заключить ваше редактирование в фигурные скобки, например так: 

```js
{/* MASSMETA EDIT ADDITION START */} 
<SomeThing>
	someProp="whatever"
</SomeThing>
{/* MASSMETA EDIT ADDITION END */}
```

### Создание новых файлов TGUI 

⚠️ При создании нового файла TGUI с нуля, пожалуйста, добавьте **Заголовочный Комментарий** в самом верху файла:

```js
// THIS IS A MASSMETA UI FILE
```

Таким образом, они легко идентифицируются как **Наши** модульные файлы TGUI `.tsx` и `.jsx`.

Собственно ничего больше делать и не нужно, комментарии `// MASSMETA EDIT` в таком файле TGUI излишне.

<!-- ## Exemplary PR's // TODO: REPLACE THESE!

Here are a couple PR's that are great examples of the guide being followed, reference them if you are stuck:

- <https://github.com/Skyrat-SS13/Skyrat-tg/pull/241>
- <https://github.com/Skyrat-SS13/Skyrat-tg/pull/111> -->

## В заключении

Терпение и труд – ТГ к\*дера перетрут. Если мы будем последовательны, то в конечном итоге это избавит НАС от будущих болей в области ГМ, когда Нам (и Вам) же придется разрешать конфликты вручную.

Благодаря более скрупулезному документированию будет сразу понятно, какие изменения были сделаны, где и с помощью каких функций, и все станет гораздо менее двусмысленным и запутанным.

Желаю удачи в ТГ кодинге. Помните, что сообщество всегда готово помочь Вам, если вдруг понадобится помощь.

Оригинальное руководство: Skyrat/NovaSector. Идея модульности: Nebula and Bandastation (SS220). Перевод и дополнения: Artemchik542. Доработка модульной системы: Artemchik542, Huz2e. Отдельное спасибо: SmArtKar.
