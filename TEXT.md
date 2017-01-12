# Играем в браузере

##Вступление

Всем привет!

###Кто я такой?

Меня зовут Всеволод.  Я разработчик в команде API Яндекс.Карт. Мой сегодняшний доклад никак не связан с моей основной работой. 
На самом деле, когда я подавал доклад на WSD, я не знал, что он будет проходить в Яндексе. Вот такое бывает =)

###Почему рассказываю этот доклад?

Но почему я рассказываю доклад фактически про игры? Дело в том, что я свободное от работы времени увлекаюсь играми. 
Более того, до Яндекса я работал в компании, которая разрабатывала игры на Flash для социальных сетей.
Да. В какой-то момент я сменил один язык основанный на ECMAScript на другой.

Совсем недавно Flash в очередной раз умер. Chrome планомерно отключает сторонние плагины в браузере последние несколько лет. 
В версии 55 так и вообще flash остался только на сайтах, которые не смогут без него работать. Сейчас на таких сайтах flash включается по клику. 
Под раздачу попали и другие внешние плагины. К примеру, Unity Technologies в какой момент отказались от Unity Web Player в пользу экспорта в пользу HTML5-игр.

Одновременно с этим последние несколько лет появилось множество спецификаций, которые расширяют возможности браузеров. 
Эта тенденция коснулась и спецификаций, которые можно использовать для разработки игр. А некоторые, к примеру GamePad API, так и вообще только для игр и создаются.

Есть много движком HTML5-игр. И consturct2,  GameMaker, тот же Unity, но так или иначе они все всё равно опираются на возможности браузеров.
Глядя на все это, захотелось сделать обзорный доклад этих спецификаций. Многие технологии наверняка уже вам знакомы, но вот захотелось поглядеть на них с непривычного угла.
Почти все спецификации из этого доклада уже достаточно давно были реализованы в браузерах.

В принципе, весь доклад построен по схеме "общие практики - особенности реализации средствами браузера".
Я вспоминал с каким задачи я сталкивался при разработке игр и искал способ решения в современных Web-спецификациях.

##Теория
Перед тем, как продолжить необходимо рассказать немного теории. Это совсем маленький раздел.

###Game loop
Обычный паттерн игры - это повторяющий цикл, в котором происходит различные расчеты, проверка сигналом и перерисовка по рассчитаному состоянию.
К примеру, есть платформер, в котором персонаж может стокнутся с блоком. Как раз в расчетах проверяется пересечение геометрий персонажа и блока.
Так же в game loop происходит опрос датчиков, которыми могут быть различные устройства ввода. К примеру, клавиши клавиатуры.

В среде Flash есть внутренний loop (на него даже можно подписаться событием ENTER_FRAME), в среде XNA game loop разделен на два этапа update (расчет) и draw (отрисовка).
В некоторых движка перерасчет физики происходит чаще, чем перерисовка графики.
В браузере есть похожий механизм - это event loop. Так что игровой цикл можно постоить на базе setInterval, а обновление по requestAnimationFrame.

На эту тему есть очень хорошая статья на MDN https://developer.mozilla.org/en-US/docs/Games/Anatomy

В цикле для расчета новой позиции игрового объекта используют скорость движения и время, которое прошло с предыдущего цикла. 
Это делается, так как никто не гарантирует, что предыдущие итерации не заняли больше отведенного времени.
Точно такая же проблема, что и нулевой timeout в браузере.

##Графика
Для отображения игры используются все те же самые технологии, что для других интерактивных мультимедийных элементов. 

* DOM
* Canvas
* WebGL
* SVG

###DOM
Для отображения игры в DOM ничего особе революционного не используют. Тот же объект Image или CSS-свойство background-image. Различные способы позиционирования элементов, разые слои.
Для некоторых эффектом используют трасформацию. К примеру, один мой коллега однажды сделал изометрийскую сетку при помощи обычной таблицы и CSS-трасформации.
Или же бывает очень удобно использовать step-анимацию для отображения спрайтовой анимации. Такую технологию использует Google для своих дудлов.
Но все же для отображения динамичных элементов обычно используют Canvas, так как тот легче оптимизировать для показа большого кол-ва элементов, а DOM элементы используют для более статичесных элементов.
К примеру, для элементов меню. 

###Canvas
В Canvas значительно легче оптимизировать отображение объектов. К примеру, многие рекомендуют использовать технологию dirty rectangles
https://www.html5rocks.com/en/tutorials/canvas/performance/#toc-render-diff
Здесь при каждом обновлении кадра происходит рассчет экранного прямоугольника, который надо будет перерисовать. Соответственно кадр обновляется быстрее.

###WebGL
Некоторые движки, такие как consturct2, pixi.js используют WebGL для отображения двухмерных спрайтов.

*TODO*

Или же WebGL использует для отображения полностью трехмерных сцен.
Красивый пример Blend4Web http://eyes.nasa.gov/curiosity/

###SVG
Кто-то даже пытается сделать отображение игры на чистом SVG, но это не очень быстро работает. В принципе, как и любая векторная графика. Особенно если элементов очень много на экране.
Но все же можно найти интересное применение SVG. На предыдущей работе мы хранили ассеты в векторном формате, а перед отображением растрировали их.
Это нам позволяло загружать меньше данных, так как векторные данные значительно меньше весят. И позволяло получить преимущество в отображении, так как в среде flash перерисовка растра происходит значительно быстрее, чем вектор.
Примерно тоже самое мы делаем и в API Яндекс.Карт для отображения обычных меток с любым цветом. У нас есть базовый ассет в формате SVG. Мы программно меняем одно свойства цвета, а далее отрисовываем SVG изображение в canvas.
Далее формируем Base64 или Blob с ссылкой и передаем это как значение background-image.

##Взаимодействие с игрой

###Клавиатура/мышь
Браузеры сейчас предоставляют много способов взаимодействия со страницей. Я уверен, что абсолютно все в зале подписывались на события мышки и клавиатуры. Это такие старые техники, что без них уже почти невозможно представить программирование на JS на клиенте. 

###Множественные касания
Есть две спецификации, которые позволяют отслеживать множественные прикосновения к экрану. Touch Events и Pointer Events. 
Первая разработана компанией Apple, а вторая Microsoft. Разница с позиции разработки игры между ними не очень большая. 
Pointer Events более низкоуровневое API, которое работает с каждым прикосновением отдельно.
В целом это мало влияет на разработку особенно, если используется canvas для отображения игры. 
Сейчас обе спецификации имеют статус рекомендаций. Пока ни одна из спецификаций не имеет полную поддержку во всех браузерах. 
Это в основном всякие юридические неурядицы. Но и та, и другая часто доступны из под флага.
Браузер Chrome (и последователи) совсем недавно получили поддержку обеих спецификаций.
TouchEvents поддерживается еще со времен пачкования от WebKit, а PointerEvents появились в версии 55.
Надеюсь, что в будущем все браузеры придут к единой поддержке, но пока приходится учитывать обе.

###Gamepad API
Следующая спецификация пока находится в состоянии Working Draft, но уже активно используется в движках. И имеет почти полную поддержку браузерами. 
В отличии от других сложно придумать применение этой спецификации кроме как для игры. Речь идет про Gamepad API. Я даже с собой принес xbox 
совместивый геймпад, чтобы показать, как это работает.
Эта спецификация в отличии от других интерфейсов выглядит очень непривычно. В ней не описаны события нажатия кнопок геймпада. 
Приходится в цикле самому пробегаться по массиму кнопок и направлений. Интерфейс очень низкоуровневый. 

###PointerLock API
Еще одна занимательная спецификация называется PointerLock API. Она к моему удивлению уже имеет статус рекомендации и почти 100% поддержку браузерами.
Эта спецификация позволяет позволяет работать с мышкой в рамках одного элемента. Дальше начинают поступать события с относительными значениями координат сдвига мышки. Это может быть полезным, если вы будете создавать игру с видом от первого лица. И хотите поворачивать камеру в сторону движения мышки. * Пример вращения панорамы через pointerLock *

###Device Orientation
Ну и нельзя не вспомнить еще про игры, где управление происходит через наклон телефона. Все же помнять Doodle Jump? 
Сейчас таких игр в целом стало выходить значительно меньше, но все же такое управление еще используется в гонках. А еще активно используется в VR играх.

##Звук
Звук так же является одной из важнейших частей любого мультимедийного продукта. Без хорошего звукового сопровождения не получится сделать полного погружения в мир игры.
Вспомните заглавную тему Morrowind&Skyrim.
Часто требуется не просто проиграть определенный звук при наступлении некоторого события, а более точно управлять звуковыми дорожками. 
Делать различные эффекты - паннинг или постепенное угасание.