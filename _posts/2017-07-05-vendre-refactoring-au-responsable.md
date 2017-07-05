---
layout: post
title: "Comment vendre l'idée du refactoring au responsable du projet?"
modified: 2017-07-05
tags: [refactoring]
categories: [code]
author: gabrielrobert
---



Dans un contexte traditionnel agile, la réécriture de code doit systématiquement être pratiquée en parallèle d'une fonctionnalité. Le refactoring n'apporte aucune plus-value aux utilisateurs, mais bien à nos collègues et aux prochains individus qui auront à implémenter de nouvelles fonctionnalités. Dans un tel contexte, cela peut devenir plutôt difficile pour un responsable de projet, qui parfois n’a aucune connaissance du code, d’approuver ce genre de travaux.

__Ça ne devrait pas être le cas.__

Le développeur jugeant qu'une pièce de code devrait être réécrite ne doit jamais être écarté sans effectuer le calcul du retour sur investissement.


## Les points positifs du refactoring

Si on vous demande quels sont les bien faits de la réécriture de code, les points à soulever à ces gens peuvent être les suivants:

- Amélioration de la maintenabilité du programme.
- Augmentation de la vélocité sur le moyen / long terme.
- Introduction des nouveaux développeurs grandement facilitée
- Un code retravaillé est beaucoup plus simple à améliorer ensuite en cas de problème de performance. Le fait que les méthodes soient courtes et bien exprimées rend le travail de profilage beaucoup plus simple.


## Si le responsable est beaucoup plus axé sur la qualité du produit

Les responsables de produit visant la qualité sur le long terme du produit seront ceux qui auront le plus facilement conscience de la nécessité du refactoring. Ces gens savent qu'un produit de qualité doit être capable d'itérer et se renouveler rapidement et convenablement, quelles que soient les contraintes du marché. Un produit de qualité est composé de modules correctement pensé, et implanté de la meilleure manière possible. *Cependant, la meilleure implantation est rarement la première, d'où l'importance du refactoring.* 


## Si le responsable est axé sur le temps et l'argent

Il ne faut pas leur en parler. __Vraiment__. Si vous tenez à conserver votre efficacité et vous tenez à augmenter votre productivité en livrant plus rapidement, gonflez vos estimations afin de permettre de corriger les erreurs du passé. Vous serez gagnant sur le long terme. Il faut cependant faire preuve de jugement, très important!


## En mode service, ça donne quoi?

Si votre travail est de desservir un client en mode forfaitaire ou au taux horaire:

- Le client n'a pas à savoir qu'un refactoring a été effectué dans son application. Je crois cependant qu'il faut être honnête et partager le bénéfice d'une telle pratique, même si c'est plus compliqué à expliquer à quelqu'un en dehors du domaine.
- Il faut toujours faire du refactoring en ayant une plus-value en tête et pas parce que ça nous tente.
- Modifier des bouts de code qui sont en production depuis un bon moment et qui ne requièrent aucune modification, peu importe la laideur de son implémentation, ne mérite pas d'être réécrit.


Si l'on vous dit que le refactoring ne devrait pas être facturé au client, je crois que c'est également faux. Prenons un contexte moins précis et dirigeons-nous vers l'automobile: lorsque vous achetez une voiture, vous l'utilisez pendant un certain moment. Vous roulez et profitez pleinement de celle-ci, cependant il vient un jour où l'huile sera usée et vous devrez débloquer un effort monétaire pour le faire. Le mécanicien est en droit de vous facturer ses services et le droit est identique dans un contexte logiciel. Nous sommes des professionnels oeuvrant sur des solutions qui évolues, il ne faut pas être surpris si à un moment ou un autre il faut faire de la maintenance.

