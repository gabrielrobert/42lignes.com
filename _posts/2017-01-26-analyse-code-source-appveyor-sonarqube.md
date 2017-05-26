---
layout: post
title: Analyse du code source avec SonarQube et AppVeyor
modified: 2016-12-19
tags: [appveyor, intégration continue, sonarqube]
categories: [code]
author: gabrielrobert
---

Vous avez déjà entendu parler de [SonarQube](https://www.sonarqube.org/)? Concrètement, c'est un analyseur de code source qui permet de calculer plusieurs métriques sur l'ensemble du codebase de votre projet. Il est disponible pour la majorité des gros langages et est incroyablement paramétrable. C'est tellement facile à utiliser que je ne trouve pas une bonne raison de ne pas le faire.

Pour ma part, je l'ai connecté directement dans AppVeyor. Ainsi, à chaque fois que j'ai un build qui est lancé, SonarQube est appelé afin d'en analyser son contenu.

Son intégration est simple, voici les 3 lignes d'instructions qui permettront de l'intégrer à votre projet:

{% highlight yaml %}
init:
  - choco install "msbuild-sonarqube-runner" -y

before_build:
  - MSBuild.SonarQube.Runner.exe begin /k:"[UNIQUE_KEY]" /n:"[PROJECT_NAME]" /v:"[VERSION]" /d:sonar.host.url=https://sonarqube.com /d:sonar.login=[TOKEN]
  
after_build:
  - MSBuild.SonarQube.Runner.exe end /d:sonar.login=[TOKEN]
{% endhighlight %}


__[UNIQUE_KEY]__: Mettez ce que vous désirez. Cela doit être unique cependant, je conseil d'utiliser le nom de votre repository. (exemple: gabriel-robert1)

__[PROJECT_NAME]__: Mettez ce que vous désirez. C'est le nom qui sera affiché dans la liste des projets du côté de SonarQube. (exemple: Gabriel Robert)

__[VERSION]__: Mettez ce que vous désirez. (exemple: 1.0.0)

__[TOKEN]__: C'est la valeur du token que vous devez générer [ici](https://sonarqube.com/account/security/). Il serait préférable d'encrypter cette donnée avec les outils d'AppVeyor



That's it!
