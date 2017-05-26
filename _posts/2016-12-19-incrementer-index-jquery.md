---
layout: post
title: Incrémenter les index de l'ensemble des inputs / span d'une div
modified: 2016-12-19
tags: [aspnet, javascript]
categories: [code]
author: gabrielrobert
---

Parfois on ne veut pas inclure certaines librairies plutôt lourde pour un besoin très minime. Dans mon cas, c'était une simple liste d'objets avec la possibilité d'en ajouter des occurences côté front-end. Je voulais éviter d'utiliser handlebars ou même pire, un framework complèt pour en venir à mes fins.

{% highlight javascript %}
var _incrementIndex = function ($element) {
    var attributes = ["id", "name", "data-valmsg-for"];
    $element.find("input, span, select, textarea").each(function (index, elem) {
        
        var $elem = $(elem);

        for (index = 0; index < attributes.length; ++index) {
            var attrValue = attributes[index];
            if ($elem.attr(attrValue)) {
                var id = $elem.attr(attrValue).replace(/[^\d]/g, '');
                var newId = $elem.attr(attrValue).replace(id, parseInt(id, 10) + 1);
                $elem.attr(attrValue, newId);
            }
        }
    });
}
{% endhighlight %}

En passant une div à cette fonction, elle s'occupera de mettre à jour l'ensemble des index en gardant la schémantique de ASP .NET.
