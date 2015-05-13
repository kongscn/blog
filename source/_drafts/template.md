title: "A Colorful Blog"
tags:
---

`npm install hexo-tag-bootstrap --save`

Ref and example: `http://wzpan.github.io/hexo-theme-freemind/2014/03/16/tag-plugins-cn/`

# Text

{% textcolor muted %}Fusce dapibus, tellus ac cursus commodo, tortor mauris nibh.{% endtextcolor %}
{% textcolor primary %}Nullam id dolor id nibh ultricies vehicula ut id elit.{% endtextcolor %}
{% textcolor success %}Duis mollis, est non commodo luctus, nisi erat porttitor ligula.{% endtextcolor %}
{% textcolor info %}Maecenas sed diam eget risus varius blandit sit amet non magna.{% endtextcolor %}
{% textcolor warning %}Etiam porta sem malesuada magna mollis euismod.{% endtextcolor %}
{% textcolor danger %}Donec ullamcorper nulla non metus auctor fringilla.{% endtextcolor %}

## Buttons

{% btn http://hahack.com hahack %}
{% btn http://hahack.com hahack primary %}
{% btn http://hahack.com hahack success %}
{% btn http://hahack.com hahack warning %}
{% btn http://hahack.com hahack danger %}
{% btn http://hahack.com hahack info %}

## Labels
{% label default %}
{% label warinng warning %}
{% label success success %}
{% label danger danger %}
{% label primary primary %}
{% label info info %}

## Badges
{% badge 42 %}



## Alerts

{% alert warning %}Best check yo self, you're not looking too good.{% endalert %}
{% alert danger %}Change a few things up and try submitting again.{% endalert %}
{% alert success %}You successfully read this important alert message.{% endalert %}
{% alert info %}This alert needs your attention, but it's not super important.{% endalert %}
