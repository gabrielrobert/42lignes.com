---
layout: post
title: "Optimisation du code: le state pattern"
modified: 2017-05-12
tags: [C#, refactoring]
categories: [code]
author: gabrielrobert
---

Le state pattern permet d'implémenter un bout de code d'une façon orientée objet. Il permet de __résoudre des problèmes d'architecture__ qui deviennent lourds à maintenir et se traduit en dette technique.

Mon cas préféré d'un state pattern&nbsp;typique adapté à la réalité est une méthode qui exécute une logique basée sur des instructions "switch" et qui est souvent reliée à un état précis (un "enum" par exemple). Voyons voir, avec une problématique de la vie réelle, comment on peut améliorer un peu nos implémentations du code.


Je désire obtenir le prix d'un instrument de musique.


On se retrouve souvent avec ce genre de classes, qui à première vue ne semble pas causer de problème, mais qui à long terme peuvent poser des problèmes de maintenabilité. Considérant que j'ai une solide suite de tests couvrant cette fonctionnalité, imaginons le code suivant:

{% highlight c# %}
public class Instrument
{
    public InstrumentType Type;

    public Instrument(InstrumentType type)
    {
        Type = type;
    }

    public int GetPrice()
    {
        switch (Type)
        {
            case InstrumentType.Guitar:
                return 200;
            case InstrumentType.Piano:
                return 435;
            case InstrumentType.Drum:
                return 320;
            default:
                throw new Exception("Incorrect type");
        }
    }
}
{% endhighlight %}



## Étape 1
Je vais abstraire le prix de sorte que son calcul soit effectué dans la classe appropriée et créer quatre nouvelles entités. Je vais modifier la classe "Instrument" en lui déclarant une méthode abstraite nommée "GetPrice", obligeant mes sous-classes à implémenter celle-ci:

{% highlight C# %}
public abstract class InstrumentPrice
{
    public abstract InstrumentType GetType();
}

public class GuitarPrice : InstrumentPrice
{
    public override InstrumentType GetType()
    {
        return InstrumentType.Guitar;
    }
}

public class PianoPrice : InstrumentPrice
{
    public override InstrumentType GetType()
    {
        return InstrumentType.Piano;
    }
}

public class DrumPrice : InstrumentPrice
{
    public override InstrumentType GetType()
    {
        return InstrumentType.Drum;
    }
}
{% endhighlight %}


## Étape 2
Je dois faire en sorte d'utiliser le bon prix dans la classe "Instrument". Il faut donc créer une nouvelle propriété qui contiendra ce prix, ainsi qu'une méthode pour l'instancier.


{% highlight c# %}
public class Instrument
{
    private InstrumentPrice _price;
    public InstrumentType Type;

    public Instrument(InstrumentType type)
    {
        Type = type;
        SetPrice(type);
    }

    public void SetPrice(InstrumentType type)
    {
        switch (type)
        {
            case InstrumentType.Guitar:
                _price = new GuitarPrice();
                break;
            case InstrumentType.Piano:
                _price = new PianoPrice();
                break;
            case InstrumentType.Drum:
                _price = new DrumPrice();
                break;
            default:
                throw new Exception("Invalid");
        }
    }
}
{% endhighlight %}


## Étape 3
Il faut maintenant déplacer la logique de calcul des prix dans la classe appropriée, soit "InstrumentPrice".

{% highlight c# %}
public abstract class InstrumentPrice
{
    public abstract InstrumentType GetType();

    public int GetPrice()
    {
        switch (GetType())
        {
            case InstrumentType.Guitar:
                return 200;
            case InstrumentType.Piano:
                return 435;
            case InstrumentType.Drum:
                return 320;
            default:
                throw new Exception("Invalid");
        }
    }
}

public class Instrument {
    private InstrumentPrice _price;

    // ...

    public int GetPrice() {
        return _price.GetPrice();
    }
}
{% endhighlight %}


## Étape 4
Ensuite, c'est le moment pour mettre en place le polymorphisme. Chaque sous-classe doit implémenter sa propre logique pour avoir le bon prix. À noter qu'il faut mettre la méthode "GetPrice" de la classe "InstrumentPrice" à abstraite.

{% highlight c# %}
public class GuitarPrice : InstrumentPrice {
    // ...
    public override int GetPrice() {
        return 200;
    }
}

public class PianoPrice : InstrumentPrice {
    // ...
    public override int GetPrice() {
        return 435;
    }
}

public class DrumPrice : InstrumentPrice {
    // ...
    public override int GetPrice() {
        return 320;
    }
}
{% endhighlight %}

## Conclusion
Voici maintenant notre architecture finale. C'est plus de code qu'au départ me direz-vous? Certes, notamment parce que notre contexte est de base. La complexité de notre programme n'est pas assez grande pour que ce pattern s'applique à la perfection, mais lorsque notre programme va prendre de l'expansion, et qu'il faudra notamment:

- Attribuer des logiques de prix différentes pour chaque instrument
- Gérer les accessoires pour chaque instrument
- Gérer les frais d'entretien

Nous pourrons alors nous dire satisfaits d'une telle modification! Je crois qu'il faut savoir prendre la décision en tant que développeur de prendre &nbsp;la décision d'inclure le state pattern dès le départ ou bien de l'omettre. Si on sait qu'aucune fonctionnalité n'est prévue nécessitant cet effort, on peut se permettre de faire la modification lorsque le temps viendra. Cependant, si nous sommes en train de monter un système d'inventaire, il serait peut-être bon de prévoir le coup l'avance.

{% highlight c# %}
public class Instrument
{
    public InstrumentType Type;
    private InstrumentPrice _price;

    public Instrument(InstrumentType type)
    {
        Type = type;
        SetPrice(type);
    }

    public int GetPrice()
    {
        return _price.GetPrice();
    }

    public void SetPrice(InstrumentType type)
    {
        switch (type)
        {
            case InstrumentType.Guitar:
                _price = new GuitarPrice();
                break;
            case InstrumentType.Piano:
                _price = new PianoPrice();
                break;
            case InstrumentType.Drum:
                _price = new DrumPrice();
                break;
            default:
                throw new Exception("Invalid");
        }
    }
}

public abstract class InstrumentPrice
{
    public abstract InstrumentType GetType();

    public abstract int GetPrice();
}

public class GuitarPrice : InstrumentPrice
{
    public override InstrumentType GetType()
    {
        return InstrumentType.Guitar;
    }

    public override int GetPrice()
    {
        return 200;
    }
}

public class PianoPrice : InstrumentPrice
{
    public override InstrumentType GetType()
    {
        return InstrumentType.Piano;
    }

    public override int GetPrice()
    {
        return 435;
    }
}

public class DrumPrice : InstrumentPrice
{
    public override InstrumentType GetType()
    {
        return InstrumentType.Drum;
    }

    public override int GetPrice()
    {
        return 320;
    }
}
{% endhighlight %}
