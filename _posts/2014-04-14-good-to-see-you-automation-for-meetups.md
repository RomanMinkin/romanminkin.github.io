---
layout:     post
title:      Good to see you automation for meetups
date:       2014-06-08 11:21:29
summary:    Simple JavaScript functions which does say 'Good to see you' to everybody in one action.
categories: blog
tags:       jquery, meetup.com
---

After each meetup many of you receive email with the subject **Stay in touch!** and a link to [Meetup.com](http://meetup.com) with simple questionnaire page **How was the Meetup?**

To keep networking level high I prefer to <b>Say hello </b>to everybody, but i'm too lazy to do it manually otherwise I would not be a developer, so here is a solution!<br />

[Meetup.com](http://meetup.com) as many sites uses [jQuery](http://jquery.com/) library, which means that we can open <a href="https://developers.google.com/chrome-developer-tools/docs/console">Developer Console</a> in our browser and run simple JavaScript command which will do all the magic for us:

1. Go to [Meetup.com](http://meetup.com) page with questionnaire form
2. Open <a href="https://developers.google.com/chrome-developer-tools/docs/console">Developer Console</a>&nbsp;in Crome, FireFox, Safari, etc
3. Run following code:
</ol>
<ol style="text-align: left;">
</ol>

{% highlight javascript %}
    (function(){

        var i;

        function tryToSayHello() {
            var button = $('p.gtsy > a.D_submit').not('.g2cu')[0],
                next   = $('a[rel=next]');


            if (button) {
                button.click();
                console.log('click');

            } else if (next) {
                next.click();

            } else {
                clearInterval(i);
                console.log('Done!');
            }
        }

        i = setInterval(tryToSayHello, 300);
    })()
{% endhighlight %}

Enjoy!

