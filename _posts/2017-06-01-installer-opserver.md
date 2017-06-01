---
layout: post
title: "Système basique de monitoring de servers SQL"
modified: 2017-06-01
tags: [monitoring, sql server]
categories: [code]
author: gabrielrobert
---

Chez [Spektrum Media](http://spektrummedia.com), nous avons près d'une dizaine de serveurs ayant chacune d'elles plusieurs bases de données. Au début, gérer et maintenir une ou deux machines est un jeu d'enfant. Mais plus le chiffre gonfle, et plus on se rend compte d'un manque: _un outil qui permet en un coup d'oeil d'avoir la santé de l'ensemble de notre flotte de serveurs_.

C'est à ce moment qu'on a découvert [Opserver](https://github.com/opserver/Opserver) de [Stack Exchange](https://stackexchange.com/), un outil OSS qui permet ce genre de choses. Sachant que les gens chez Stack Exchange ont aussi une stack [fondamentalement basée sur .NET](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/), pourquoi ne pas l'essayer?


### Installation du projet sur un poste de travail

On va d'abbord faire fonctionner le tout en local. Ensuite, on va passer pour des systèmes en production.

1) Faire la commande suivante dans votre console: `git clone https://github.com/opserver/Opserver`

2) Ouverture du projet dans Visual Studio et compilation.

3) Ajouter un site dans IIS poitant vers `Opserver` (et non `Opserver.Core`). Personnellement, je vais le faire pointer vers `opserver.local`.

4) Ajouter l'entrée dans le fichier host pour faire pointer `127.0.0.1` vers `opserver.local`.


### Les configurations de sécurité
![problème 1](/images/posts/2016-11-03-deployer-application-appveyor-agent/probleme-1_configuration_error.jpg "Problème 1")

La cause du problème vient du fait que le fichier pointé sur l'image ci-haut n'existe pas. Heureusement, nous avons un fichier exemple qui se situe juste ici: `/Config/SecuritySettings.config.example`.

Optionel: On peut ajouter certain network dans ce fichier afin de pouvoir accèder à Opserver sans être authentifié.

Pour le moment, je vais laisser la configuration à `<SecuritySettings provider="alladmin" />`, puis supprimer le `.example` de l'extension du fichier. Une fois fait, c'est le moment de rafraichir notre page.


### Page de connexion
![Woot!](/images/posts/2016-11-03-deployer-application-appveyor-agent/login_page.jpg "Page de connexion")

À cause de nos configurations précédentes, nous n'avons pas besoin d'identifiants pour se connecter, il donc suffit de cliquer sur `Log in` pour accèder au tableau de bord du logiciel.

### Les configurations du matériel
![Aucune configuration](/images/posts/2016-11-03-deployer-application-appveyor-agent/no_configuration.jpg "Aucune configuration")

Il fallait bien s'en douter, nous n'avons configuré aucune machine externe à notre instance de Opserver. Dans notre cas, ce qui nous intéresse le plus, c'est la capacité de moitorer nos instances de SQL Server. Pour ce faire, il faut aller dans le dossier `Config` et supprimer l'extension `.example` du fichier `SQLSettings.json.

Ce fichier-là est très intéressant. En voici son contenu:

```json
{
    "defaultConnectionString": "Data Source=$ServerName$;Initial Catalog=master;Integrated Security=SSPI;",
    "clusters": [
        {
        	"name": "NY-SQLCL03",
        	"refreshIntervalSeconds": 20,
        	"nodes": [
        		{ "name": "NY-SQL01" },
        		{ "name": "NY-SQL02" },
        		{ "name": "OR-SQL01" },
        	]
        },
        {
        	"name": "NY-SQLCL04",
        	"refreshIntervalSeconds": 20,
        	"nodes": [
        		{ "name": "NY-SQL03" },
        		{ "name": "NY-SQL04" },
        		{ "name": "OR-SQL02" },
        	]
        }
    ],
    "instances": [
        { 
            "name": "NY-DB05",
            "connectionString": "Data Source=NY-DB05;Initial Catalog=bob;Integrated Security=SSPI;", 
        },
        { "name": "NY-DESQL01" },
        { "name": "NY-RESTORESQL01" },
        { "name": "NY-UTILSQL01" },
        { "name": "OR-DESQL01" },
        { "name": "OR-HALOG01" }
    ]
}
```

Concrètement, on peut y voir une série de clusters et une série d'instances accompagnées de chaînes de connexion.

On va donc commencer à tester notre système en poitant sur notre poste de travail. Évidemment, notre environnement local ne possède pas le concept de clusters, on peut donc supprimer cette section et ajouter nos informations comme suit.


```json
{
  "defaultConnectionString": "Data Source=.\\sqlexpress;Initial Catalog=master;User Id=sa;Password=sa;",
  "instances": [
    {
      "name": "\\SQLEXPRESS",
      "connectionString": "Data Source=.\\sqlexpress;Initial Catalog=master;User Id=sa;Password=sa;"
    }
  ]
}
```

Les choses importantes à noter:

- Le `Data Source` pointant vers mon environnement local.
- Le `Initial Catalog` poitant vers `master`.
- J'ai remplacé le `Integrated Security` par un utilisateur de ma base de donnée qui a les droits sur l'ensemble des tables. MSSQL propose l'utilisateur `sa` out-of-the-box, c'est celui-là que j'ai pris en changeant pour un mot de passe. __Ne faites pas ça en production__.


## On y est pour l'environnement local

![Local monitoring](/images/posts/2016-11-03-deployer-application-appveyor-agent/sql_dashboard.jpg "Local monitoring")

On y est pour notre preuve de concept. C'est maintenant le moment de pousser ça en production et d'en faire un outil __indipensable__.


## Déploiement
Étant un fan d'Azure et du cloud, je vais faire passer le tout directement dans le nuage. Pour cette étape-ci, vous aurez besoin du [`azure-cli`](https://github.com/Azure/azure-cli).

On ne va pas se casser la tête et créer une webapp.

```powershell
# Connexion
az login -u {VOTRE_USERNAME} -u {VOTRE_PASSWORD}

# Création de l'application. Notez qu'il faut changer `opserver` pour un nom unique.
az webapp create --name opserver --resource-group {resource-group-name} --plan {plan-name}

# Créer un user pour le deployment
az appservice web deployment user set --user-name <username> --password <password>

# Configurer l'application pour utiliser le git local. Utilisez un repo distinct de celui de github/opserver.
# Sauvegardez le résultat sous le format suivant: https://<username>@<app-name>.scm.azurewebsites.net:443/<app-name>.git
az appservice web source-control config-local-git --name <app-name> --resource-group {resource-group-name} --query url --output tsv

# Pousser le répertoire local vers l'environnement
git remote add azure https://username@<app-name>.scm.azurewebsites.net/<app-name>.git
git push azure master
```

## Voilà, c'est bon!

Il vous suffit maintenant d'aller mettre vos fichiers manuellement dans votre application sous le dossier `Config` et vous pourrez avoir votre solution de monitoring de serveurs prêt à faire feu.

Assurez-vous de ne divulguer __aucune__ chaînes de connexion de production dans votre répertoire. Il serait tellement facile pour quiconque mal intentioné de pouvoir s'amuser, car n'oublions surtout pas que ce projet contiendra littéralement une porte d'entrée vers l'ensemble de vos servers SQL.
