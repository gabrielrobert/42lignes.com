---
layout: post
title: "Supporter le mode hors-ligne dans un contexte d'application mobile Xamarin"
modified: 2017-09-05
tags: [azure, xamarin, c#]
categories: [code]
published: false
author: gabrielrobert
---

L'accès aux données dans une application est toujours de mise, et surtout lorsqu'on est au niveau d'une application mobile native, le support du mode hors-ligne ne doit pas être épargné. Cet article portera sur la synchronisation d'une base de donnée **SQL Server** hébergée sur Azure vers une base de donnée **SQLite** directement stockée sur l'appareil grâce à la suite d'outils **Azure Mobile Apps**. De plus, nous pourrons donc développer en mode "offline-only" avec la possibilité de rafraichir les donnés dans le background de l'application si la connexion au réseau est possible.

## Mise en place de la structure contenant les données

À terme, nous aurons une [Azure Mobile Apps](https://azure.microsoft.com/en-ca/services/app-service/mobile/) contenant une base de donnée SQL Server à laquelle sera attaché une flotte de **Easy tables**.

> Qu'est-ce qu'une _Azure Mobile App_?

Une Azure Mobile App est exactement comme un Azure Website

> Qu'est-ce qu'une _easy table_?

Concrètement, ce concept prend la forme de mirroir du schema de votre base de donnée SQL Server. Vous y retrouvez les mêmes tables et les mêmes données. Son utilité vient du fait qu'il expose une suite d'outils et de SDK pour simplifier l'accès des clients mobiles. L'implémentation des Easy tables est faites en nodejs, mais n'ayez craintes, vous n'aurez pas à y toucher. 

Basé sur l'exemple [suivant](https://blog.xamarin.com/getting-started-azure-mobile-apps-easy-tables/), voici donc les étapes pour mettre en place le tout. Veuillez vous référer à l'exemple pour plus de détails.

1) Créer une Azure Mobile App avec le nom approprié à votre application.
- Créer dans la région la plus près de vos utilisateurs.
- Placer le mode `Always On` afin que l'instance ne soit pas fermée lorsqu'elle n'est pas utilisée.
- Mettre au niveau `standard` pour avoir des performances dignes du nom.

2) Ensuite, ajouter une `data connection`. Ce sont les informations de votre base de donnée SQL Server.
- Inscrire le nom de votre base de donnée.
- Choisir un admin login convenable.
- Choisir un mot de passe sécuritaire.
- Cocher `Allow azure services to access server`.

## Mise en place des données

Il faudra maintenant peupler les tables de votre base de donnée SQL Server. Évidemment, vous pouvez le faire en accédant directement à la base de donnée en utilisant [SQL Server Mangement Studio](https://en.wikipedia.org/wiki/SQL_Server_Management_Studio), mais là où c'est vraiment intéressant, c'est que l'ensemble de l'infrastructure est compatible avec Entity Framework. Il sera donc possible de créer, éxécuter et supprimer des migrations à votre bon vouloir.

Mais, évidemment, il a un mais. Pour pouvoir rendre l'option d'Entity Framework possible, il faudra que l'ensemble de vos entités implémentent les champs suivants:

```csharp
[Key]
[TableColumn(TableColumnType.Id)]
public string id { get; set; }

[Timestamp]
[TableColumn(TableColumnType.Version)]
public byte[] version { get; set; }

[Index(IsClustered = true)]
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
[TableColumn(TableColumnType.CreatedAt)]
public DateTimeOffset? createdAt { get; set; }

[DatabaseGenerated(DatabaseGeneratedOption.Computed)]
[TableColumn(TableColumnType.UpdatedAt)]
public DateTimeOffset? updatedAt { get; set; }

[TableColumn(TableColumnType.Deleted)]
public bool deleted { get; set; }
```

Vous avez bien lu. Le champ `id` est de type `string` et est écrit en minuscule. De plus,il faudra impérativement que ce champ contienne un `Guid` pour être considéré valide. Les champs `version`, `createdAt`, `updatedAt` et `deleted` ne sont pas gérés par vous, les Easy tables gère ces propriétés de manière autonome.

Personnellement, j'aime bien faire hérité l'ensemble de mes entités d'une classe semblable à celle-ci:

```csharp
public class Entity
{
    public Entity()
    {
        id = Guid.NewGuid().ToString();
    }

    [Key]
    [TableColumn(TableColumnType.Id)]
    public string id { get; set; }

    [Timestamp]
    [TableColumn(TableColumnType.Version)]
    public byte[] version { get; set; }

    [Index(IsClustered = true)]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    [TableColumn(TableColumnType.CreatedAt)]
    public DateTimeOffset? createdAt { get; set; }

    [DatabaseGenerated(DatabaseGeneratedOption.Computed)]
    [TableColumn(TableColumnType.UpdatedAt)]
    public DateTimeOffset? updatedAt { get; set; }

    [TableColumn(TableColumnType.Deleted)]
    public bool deleted { get; set; }
}
```


## Synchronisation des données

Faites attention à ne synchroniser que les données attribuées à un utilisateur précis. Un base de donnée SQLite sur un appareil peut être extraite puis accéder dans dehors du contexte de notre application.