---
layout: post
title: "Recueil de commandes / scripts utilitaires"
modified: 2017-08-16
tags: [outils, powershell, productivité, terminal]
categories: [code]
author: gabrielrobert
---


Voici une liste plutôt personelle de différentes commandes pouvant être utiles et qui permettent d'améliorer la productivité dans la vie de tout les jours. C'est plus une cheat sheet qu'un cas typique d'article.


## Supprimer les dossiers `bin` et `obj` d'une solution

Attention, cela va aussi affecter les dossiers `node_modules`, veuillez donc effectuer cette commande avec précaution.

```powershell
Get-ChildItem .\ -include bin,obj -Recurse | foreach ($_) { remove-item $_.fullname -Force -Recurse }
```

[Voir les sources](https://stackoverflow.com/questions/755382/i-want-to-delete-all-bin-and-obj-folders-to-force-all-projects-to-rebuild-everyt)