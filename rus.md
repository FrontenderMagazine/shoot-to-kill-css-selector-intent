Огонь на поражение: правильная цель для CSS-селектора
==========================================================

Я расстраиваюсь каждый раз, когда вижу, что кто-то написал неточный
селектор. Такой селектор подобен ковровой 
бомбардировке — его цель слишком общая. Селектор `.header ul{}`
против `.main-nav{}`, `.widget h2{}` против `.widget-title`, `article > p
:first-child{}` против `.intro{}`. Всё это — недостаточно точные
селекторы.

Рассмотрим пример с селектором `.header ul{}`. Предположим, что `ul` — главное
меню на нашем сайте. Как вы и ожидаете, меню находится в шапке сайта, и сейчас
это единственный `ul` в блоке `.header`. `.header ul{}` — подходящий селектор,
верно? **Не совсем**. Я имею ввиду, что он, может быть, и работает, но
недостаточно хорошо. Этот селектор не учитывает возможные изменения в разметке
в будущем и, кроме того, не отличается точностью. Как
только мы добавим в шапку ещё один `ul`, к нему применятся все стили нашего 
главного меню, а это, скорее всего, не то, что мы хотели. 
Всё это приведёт к рефакторингу большого количества кода *или* к вынужденному 
сбрасыванию стилей для нового `ul`, чтобы избавиться от воздействия слишком общего селектора.

Цель селектора должна совпадать с причиной, по которой вы решили что-то
стилизовать. Спросите себя: **«Я пишу этот селектор, потому что мне нужно
выбрать `ul` внутри `.header`, или потому, что я хочу выделить главное меню?»**
Ответ на этот вопрос поможет вам правильно выбрать селектор.


## Это всё из-за ключевых селекторов… ##

Что действительно определяет действие всего селектора — так этот ключевой 
селектор. Ключевой селектор — это последний селектор перед
открывающейся фигурной скобкой `{`. Это очень важная деталь в мире CSS, в
котором ***браузеры читают селекторы справа налево***.

	.header ul      { /* Ключевой селектор — ‘ul’ */ }
	.ul li a        { /* Ключевой селектор — ‘a’ */ }
	p:last-child    { /* Ключевой селектор — ‘:last-child’ */ }

Как я уже писал в своей статье [«Как писать CSS-селекторы, эффективные с точки зрения
производительности»][1], ключевой селектор играет главную
роль в производительности CSS, но всё же не стоит забывать, что на выбор селектора
должна влиять его цель. Селектор `html > body > section.content > article span{}`
витиеват до нелепости и настолько ужасен, что я уверен, что никто
и никогда такое не писал, правда? Но даже несмотря на катастрофичность и
огромный вес этого селектора, его ключевой селектор `span` всё ещё 
**слишком широковещательный**. По большому счёту без разницы *что* вы пишете 
до ключевого селектора, потому что ключевой селектор — единственное, что 
действительно имеет значение.

В качестве **основного правила** постарайтесь никогда не использовать
селектор по тегу для ключевого селектора (обычно, это элементы типа `ul` или
`span`) и также не используйте для этой цели базовые объекты (`.nav` или
`.media`). Если `.media` находится в контентной области, это не означает, 
что элемент будет там всегда.

Давайте посмотрим на пример с `.header ul{}`. Предположим, в своей разметке
мы используем [nav абстракцию][2]:

	<div class=header>
	
		<ul class=nav>
			[ссылки]
		</ul>
	
	</div>

Мы можем выбрать элемент навигации несколькими способами:

	.header ul{
		[Стили для главного меню]
	}

Плохой вариант, так как после добавления ещё одного `ul` в шапку он
будет выглядеть как главное меню. Но этой опасной ситуации легко избежать.

Затем мы попробуем:

	.header .nav{
		[Стили для главного меню]
	}

Это ненамного лучше примера с `.header ul`, но тем не менее.
Теперь мы можем спокойно добавлять `ul`, но невозможно добавить что-либо 
ещё с классом `.nav`. Добавление вложенного меню или «хлебных крошек»
становится кошмаром!

В итоге лучшим решением нашей проблемы будет добавление второго класса к
элементу `ul` — `.main-nav`:

	<div class=header>
	
		<ul class="nav main-nav">
			[ссылки]
		</ul>
	
	</div>
	
	.main-nav{
		[Стили для главного меню]
	}

Это правильная цель для селектора: теперь мы выбираем элемент по **совершенно
четким** причинам, а не по косвенным обстоятельствам или вовсе случайно. Теперь
мы можем добавлять сколько угодно `ul` и `.nav` в `.header`, и стили главной 
навигации никогда не повлияют на что-то ещё. Мы больше не занимаемся
ковровым бомбометанием!

Держите ваши селекторы точными и настолько явными, насколько вы можете,
предпочитая для этого скорее классы, чем что-либо другое. 
Добавление специфических стилей с использованием нечёткого селектора
может создать проблемы в будущем. Общий селектор должен нести с собой общие
стили, поэтому, если вы хотите повлиять на конкретный элемент, то скорее
всего, вам необходимо добавить на него уточняющий класс. **Конкретная цель
требует явного и точного селектора.**


## Примеры из жизни ##

Хороший пример, в котором я спутал карты самому себе, — проект, сделанный
мной в Sky. У меня был простой селектор `#content table{}`. (Бррр, я даже
использовал ID!). Этот селектор опасен сразу по трём причинам: во-первых, в
нём используются ID, чего категорически [нельзя делать][3], во-вторых, этот
селектор имеет куда больший вес, чем ему нужно, и, последняя и самая важная
причина, — этот селектор слишком неточный. Я не хотел стилизовать те таблицы 
**только потому**, что они находились внутри `#content` — 
просто уже была такая разметка, и я решил этим воспользоваться. 
Я *полностью* признаю свою вину.

Первые несколько недель всё было отлично, но потом нам внезапно понадобилось
добавить в `#content` таблицы, которые бы выглядели иначе, чем существующие. Мой
предыдущий селектор был слишком общим, и теперь мне приходилось сбрасывать
стили, которые я задал для **каждой** таблицы внутри блока `#content`. Эх, если бы
я только имел более точные селекторы, чем эти:

	#content table{
		[стили для первой таблицы]
	}
	#content .bar{
		[отмена стилей первой таблицы]
		[стили для второй таблицы]
	}

Мне следовало использовать:

	.foo{
		[стили для первой таблицы]
	}
	.bar{
		[стили для второй таблицы]
	}

В итоге **гораздо** меньше головной боли. Думая наперёд и уделив больше внимания 
цели селектора, я сэкономил бы себе кучу времени.

## Исключения ##

Конечно, **всегда** есть исключения. Разумной причиной для селектора
`.main-nav > li` может быть то, что целевым селектором **является** селектор по
тегу. Также отличной идеей будет использовать `a`, чтобы выбрать абсолютно все ссылки,
например, так:

	html{
		color:#333;
		background-color:#fff;
	}

	/* Инвертированная цветовая схема для рекламируемых товаров */
	.promo{
		color:#fff;
		background-color:#333;
	}
	.promo a{
		color:#fff;
		text-decoration:underline;
	}

Вполне разумно использовать общий селектор, когда нужно
стилизовать каждый `а` в режиме ковровой бомбардировки.


## Заключение ##

В общем, вместо коврового бомбометания по элементам, стреляйте на поражение, 
выбирайте специфичные и точные селекторы. Убедитесь, что ваши селекторы 
аккуратны и направлены прямо в цель.

Внимательно думайте о том, какова цель ваших стилей, и выбирайте
более точный и осмысленный селектор; совершенствуйте ваши цели. Вы имеете ввиду:

	.header em{}

или на самом деле:

	.tagline{}

Вы хотите это:

	.footer p{}

или, на самом деле, это:

	.copyright{}

Будет ли разумно написать так:

	.sidebar form{}

или же безопаснее так:

	.search-form{}

Изучите цели ваших селекторов: были ли вы достаточно точны? Выбирают ли
селекторы правильные элементы по правильным причинам или это просто стечение
обстоятельств? Стреляйте на поражение. Будьте CSS-снайпером, а не
бомбардировщиком.

Между прочим, если вы переключитесь с длинных селекторов `.header ul` на
селекторы вроде `.main-nav`, это также уменьшит вес селекторов и 
увеличит их производительность. Победа! Все счастливы!

Также следует заметить, что [Джонатан Снук][4] написал схожий материал под
названием [*«Длина селектора»*][5]…

[1]: http://csswizardry.com/2011/09/writing-efficient-css-selectors/ "Writing efficient CSS selectors"
[2]: http://csswizardry.com/2011/09/the-nav-abstraction/ "The ‘nav’ abstraction"
[3]: http://csswizardry.com/2011/09/when-using-ids-can-be-a-pain-in-the-class/ "When using IDs can be a pain in the class.."
[4]: https://twitter.com/snookca "@snookca"
[5]: http://smacss.com/book/applicability "Depth of Applicability"
