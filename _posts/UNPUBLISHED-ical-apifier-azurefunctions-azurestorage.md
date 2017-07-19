---
layout: post
title: "Un flux iCal dynamique avec Apifier, Azure Functions et Azure Storage"
modified: 2017-07-18
tags: [azure, hacks]
categories: [code]
published: false
author: thatfrankdev
---

Je crois que l'outil dont je me sert le plus dans ma "vie numérique" est le calendrier en ligne. Que ce soit via Google, Outlook ou autre, les calendriers en ligne permettent d'organiser notre vie qui va parfois très très vite de nos jour.

Je joue au Ultimate chaque semaine dans les diverses ligue mises en place par [Ultimate Québec](http://www.ultimatequebec.ca/). Bien que le site de l'organisation soit très bien fait, il me manquait un accès simple au calendrier des parties à venir, et avoir accès à cette info sur mon téléphone intelligent. Dernièrement, j'ai décidé de remédier à la situation, avec l'aide de quelques outils très intéressants.

## Première étape, extraire les données

La première chose dont j'ai besoin avant de faire quoi que ce soit, ce sont des données structurées. Mais ce n'est pas si simple...

![tableau de la ligue sur le web](/images/posts/ical-apifier-azurefunctions-azurestorage/league-schedule-table.png)
_Les données sont dans un tableau HTML sur le site web, et il faut être connecté pour y accéder._

J'aurai donc recours au [_web scrapping_](https://en.wikipedia.org/wiki/Web_scraping). Dans mes recherches je suis tombé sur [Apifier](https://www.apifier.com/), un produit très intéressant. Ce service permet d'automatiser le _scraping_ du site web, et d'exposer les données extraites via API HTTP. Le service offre suffisamment de contrôle sur le processus de scraping pour que je puisse automatiser :

* La connection au site web
* L'accès à la page d'horaire de la ligue
* l'extraction de toute les parties affichées dans le tableau
* La navigation de page en page pour répéter le processus

Le _scraper_ est exécuté quotidiennement, et chaque exécution produit un résultat en JSON accessible par HTTP. Voici l'exemple d'une partie extraite :

{% highlight json %}

{
    "date": "mardi, 18 juillet 2017",
    "hour": "18h30",
    "place": {
        "name": "Cégep Garneau, 1",
        "url": "http://fichiers.ultimatequebec.ca/Sans titre.png"
    },
    "team1": {
        "name": "Les Touristes",
        "url": "http://www.ultimatequebec.ca/members/teams/les-touristes--2"
    },
    "team2": {
        "name": "Discothèque",
        "url": "http://www.ultimatequebec.ca/members/teams/les-touristes--2"
    },
    "url": "http://www.ultimatequebec.ca/members/games/11438"
}

{% endhighlight %}

## Transformer les données

Maintenant les données sont extraites sur une base quotidienne et accessibles en tout temps, je dois maintenant les transformer en calendrier [iCal](https://icalendar.org/RFC-Specifications/iCalendar-RFC-5545/).

J'utiliserai une autre superbe fonctionnalité d'Apifier

TODO

https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-your-first-function-visual-studio

https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-blob

## Stocker les données

## Wrap Up