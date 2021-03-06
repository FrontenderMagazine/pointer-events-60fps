# 60fps scrolling using pointer-events: none

Paul Lewis did an interesting article a while back about 
[avoiding unnecessary paints][1] through disabling hover effects as the user 
scrolls, which is a great approach. The down side being managing all your hover 
states through a parent class.

  .hover .element:hover {
    box-shadow: 1px 1px 1px #000;
  }

That approach doesn’t scale well and it creates unnecessary specificity 
in your CSS.

The gist of the technique is on scrolling the `.hover` class is removed from the
body and all your `.hover` selectors won’t match until the user stops scrolling
and the class is then added back to the body.

Then I saw a genius little tweet from Christian Schaefer.

<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/paul_irish">@paul_irish</a> Easy. Apply &quot;pointer-events: none&quot; to the &lt;body&gt; on scrollstart and remove it on scrollend. <a href="https://twitter.com/tabatkins">@tabatkins</a></p>&mdash; Christian Schaefer (@derSchepp) <a href="https://twitter.com/derSchepp/statuses/400394164350644224">November 12, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Pointer-events property to the rescue

This is a much better approach as it will just make the mouse pass through the
element that has the `pointer-events: none` property set. Take a look at the
screencast I did showing the dramatic difference disabling hover makes.

<iframe width="640" height="480" class="iframe-fix" src="//www.youtube.com/embed/IyHb0SJms6w" frameborder="0" allowfullscreen></iframe>

We get all the benefits of the original approach without introducing
maintainability and specificity issues in our CSS.

  .disable-hover {
    pointer-events: none;
  }

All we have to do is add the `.disable-hover` class to the body when the user
begins to scroll. This then allows the users cursor to pass through the body and
thus disable any hover effects.

  var body = document.body,
      timer;

  window.addEventListener('scroll', function() {
    clearTimeout(timer);
    if(!body.classList.contains('disable-hover')) {
      body.classList.add('disable-hover')
    }
    
    timer = setTimeout(function(){
      body.classList.remove('disable-hover')
    },500);
  }, false);

The code is pretty simple we clear the timeout, this is important after the
initial scroll, check to see if the disable class isn’t already on the body
element and then set up a delayed timer to remove the class once the user has
stopped scrolling for at least 500ms.

## A more robust technique

Now applying pointer-events to the body will work just fine in most cases but if
pointer-events: auto is applied to any child elements they will override the
parent property and cause janky scroll.

  .disable-hover,
  .disable-hover * {
    pointer-events: none !important;
  }

Easy solution is to do the asterisk selector and add an important to the
property value so it will disable any child elements with pointer-events turned
on.

Take a look at the [testcase][2] for yourself and run some timelines to see the
performance gains from this simple technique.

[1]: http://www.html5rocks.com/en/tutorials/speed/unnecessary-paints/
[2]: http://jsbin.com/oNiVUYe/5/quiet
