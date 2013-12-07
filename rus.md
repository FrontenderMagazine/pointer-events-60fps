# 60 кадров в секунду с помощью pointer-events: none; при прокрутке

Пол Льюис (Paul Lewis) не так давно опубликовал интересную заметку о том, как 
[избежать ненужных перерисовок][1], отключив эффекты при наведении во время
прокрутки страницы пользователем — и это отличный подход. Обратной 
стороной этого решения становится необходимость управлять состоянием при 
наведении с помощью класса элемента-предка.

    .hover .element:hover {
        box-shadow: 1px 1px 1px #000;
    }

Этот подход приводит к недостаточной масштабируемости и добавляет излишнюю
специфичность селекторам в CSS.

Суть этой техники в том, что при прокрутке класс `.hover` убирается с 
элемента `body`, и все селекторы, содержащие `.hover`, перестают работать до 
того момента, когда пользователь остановит прокрутку и класс будет снова 
добавлен к `body`.

Позже я увидел гениальный твит от Кристиана Шэфера (Christian Schaefer).

<blockquote class="twitter-tweet" lang="en">
<p><a href="https://twitter.com/tabatkins">@tabatkins</a> It&#39;s complicated… common technique of toggling global class causes big recalc style. <a href="http://t.co/DwXXRZh7dC">http://t.co/DwXXRZh7dC</a> Anyway ignore me ;)</p>&mdash; Paul Irish (@paul_irish) <a href="https://twitter.com/paul_irish/statuses/400393234754449408">November 12, 2013</a>
<p><a href="https://twitter.com/paul_irish">@paul_irish</a> Easy. Apply &quot;pointer-events: none&quot; to the &lt;body&gt; on scrollstart and remove it on scrollend. <a href="https://twitter.com/tabatkins">@tabatkins</a></p>&mdash; Christian Schaefer (@derSchepp) <a href="https://twitter.com/derSchepp/statuses/400394164350644224">November 12, 2013</a>
</blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

> **Пол Айриш (Paul Irish, [@paul_irish][3])**: [@tabatkins][6] Это сложно… 
> Распространенная техника переключения глобального класса приводит к 
> серьезному пересчету стилей [crbug.com/317007][4] В любом случае, не обращай
> на меня внимания :)  
> **Кристиан Шэфер ([@derSchepp][5])**: [@paul_irish][3] Все проще. Применяй 
> `"pointer-events: none"` к `<body>` при scrollstart и удаляй его при 
> scrollend [@tabatkins][6]


## Всех спасет свойство pointer-events

Такой подход гораздо лучше, потому что позволяет элементу, у которого задано 
свойство `pointer-events: none`, просто не реагировать на наведение на него 
мышью. Посмотрите мой скринкаст, показывающий значительную разницу в скорости
отрисовки при отключении эффектов при наведении.

<iframe width="640" height="480" class="iframe-fix" src="//www.youtube.com/embed/IyHb0SJms6w" frameborder="0" allowfullscreen></iframe>

Мы получаем все преимущества оригинального подхода без проблем с 
поддерживаемостью кода и специфичностью в CSS.

    .disable-hover {
      pointer-events: none;
    }

Все, что нам надо сделать — это добавить класс `.disable-hover` к `body`, когда 
пользователь начинает прокручивать страницу. Это позволит курсору «пролететь» 
над страницей, не вызывая у элементов реакции при наведении на них мышью.

    var body = document.body,
        timer;
    
    window.addEventListener('scroll', function() {
      clearTimeout(timer);
      if(!body.classList.contains('disable-hover')) {
        body.classList.add('disable-hover')
      }
      
      timer = setTimeout(function(){
        body.classList.remove('disable-hover')
      }, 500);
    }, false);

Код достаточно прост — мы сбрасываем таймер (важно сделать это после 
первоначальной прокрутки), проверяем, не установлен ли уже класс для элемента 
`body`, а затем запускаем таймер для отложенного удаления класса спустя 
500 мс после того, как пользватель остановил прокрутку.

## Более основательная техника

Применение свойства `pointer-events` к `body` отлично сработает в большинстве 
случаев, если только для дочерних элементов не указано значение 
`pointer-events: auto`, перекрывающее родительское значение и вызывающее 
торможение при прокрутке.


Решить эту проблему можно использовав селектор `*` и добавив `!important` к 
значению свойства, чтобы отключить `pointer-events` для всех дочерних элементов.

    .disable-hover,
    .disable-hover * {
      pointer-events: none !important;
    }

Взгляните сами на [тесткейс][2], открыв Timeline в инструментах разработчика, 
чтобы увидеть прирост производительности, полученный с помощью этой простой 
техники.

[1]: http://www.html5rocks.com/en/tutorials/speed/unnecessary-paints/
[2]: http://jsbin.com/oNiVUYe/5/quiet
[3]: http://twitter.com/paul_irish
[4]: http://crbug.com/317007
[5]: http://twitter.com/derSchepp
[6]: http://twitter.com/tabatkins
