---
layout: post
title: Téléchargement d'un fichier en javascript
modified: 2017-03-24
tags: [vuejs, javascript]
categories: [code]
author: gabrielrobert
---

Cette fonctionnalité se couple particulièrement bien avec [celle-là](http://www.gabrielrobert.com/2017/03/barre-de-progression-simpliste-en-vuejs.html), puisque le téléchargement du fichier permet d'avoir accès à la progression.

Concrètement, on s'occupe d'aller activer le download directement par AJAX, d'avoir le retour directement dans un blob puis de générer un bouton qu'on s'occupera de cliquer de manière seamless.

{% highlight javascript %}
// La façon la plus simple, sans progression visuelle pour l'utilisateur
File.download('/export')
  .then((link) => { link.click() })

// Avec un paramètre X
File.download('/export', { x : 'paramètre' })
  .then((link) => { link.click() })

// Avec un paramètre et la progression. Je simule partiellement un component en VueJS pour les besoins de la cause :)
methods: {
  loadFile() {
    File.download('/export', { x : 'paramètre' }, this.progress)
      .then((link) => { link.click() })
  },
  progress(e) {
    // Attribuer les valeurs à notre component ProgressBar
    console.log('PROGRESS', e)
  }
}
{% endhighlight %}

{% highlight javascript %}
import Vue from 'vue'

// Votre librairie favorite pour les calls HTTP
const client = {}

export default {
  download(url, data, progressCallBack) {
    let request = client.get(url, {
      progress: progressCallBack
    })
    return request.then((response) => {
      var result = document.createElement('a');
      var contentDisposition = response.headers.get('Content-Disposition') || '';
      var filename = contentDisposition.match(/filename=(.+);/)[1];
      filename = filename.replace(/"/g, '')
      return response.blob()
        .then(function (data) {
          result.href = window.URL.createObjectURL(data);
          result.target = '_self';
          result.download = filename;
          return result;
        })
    })
  }
}
{% endhighlight %}
