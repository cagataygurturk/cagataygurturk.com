---
layout: post
title:  "Load a Bootstrap popover content with AJAX"
date:  2015-04-10 17:37:00
categories: web
---
Loading a content via AJAX in a <a href="http://getbootstrap.com/javascript/#popovers" target="_blank">Bootstrap popover</a> is a very common pattern and, although it is not supported out of the box by Bootstrap, it is very easy to get this functionality with jQuery.

First we should add a `data-poload` attribute to the elements you would like to add a pop over to. The content of this attribute should be the url to be loaded (absolute or relative):

{% highlight javascript %}
    <a href="#" title="blabla" data-poload="/test.php">blabla</a>
{% endhighlight %}

And in JavaScript, preferably in a $(document).ready();

{% highlight javascript %}
    $('*[data-poload]').hover(function() {
        var e=$(this);
        e.off('hover');
        $.get(e.data('poload'),function(d) {
            e.popover({content: d}).popover('show');
        });
    });
{% endhighlight %}

`off('hover')` prevents loading data more than once and `popover()` binds a new hover event. If you want the data to be refreshed at every hover event, you should remove the `off`.

Please see the working <a href="https://jsfiddle.net/DTcHh/6415/" rel="nofollow">JSFiddle</a> of the example.

This blog post is created using my original answer on <a href="http://stackoverflow.com/questions/8130069/load-a-bootstrap-popover-content-with-ajax-is-this-possible" target="_blank" rel="nofollow">stackoverflow</a>.