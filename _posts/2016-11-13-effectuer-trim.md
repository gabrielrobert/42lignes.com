---
layout: post
title: Effectuer un "Trim()" sur l'ensemble des données envoyées au serveur
modified: 2016-11-13
tags: [aspnet core, mvc, model binding]
categories: [code]
author: gabrielrobert
---

Trop souvent, une base de donnée contient des données avec des caractères blancs complètement inutiles. Par exemple un nom, au lieu de "Gabriel" on se retrouve avec "Gabriel " ou " Gabriel". C'est très facile pallier à ce problème en effectuant un .Trim() tout bête juste avant la persistance des données. Cependant, ça devient vite facile de l'oublier.

C'est pourquoi un custom model binder peut pallier à cette problématique très rapidemment, et ce pour l'ensemble de l'application.

{% highlight c# %}
public class TrimModelBinderProvider : IModelBinderProvider
{
    public IModelBinder GetBinder(ModelBinderProviderContext context)
    {
        if (context.Metadata.ModelType.GetTypeInfo().IsValueType ||
            (context.Metadata.ModelType == typeof(string)))
        {
            return new TrimModelBinder();
        }

        return null;
    }
}
{% endhighlight %}

{% highlight c# %}
public class TrimModelBinder : IModelBinder
{
    public Task BindModelAsync(ModelBindingContext bindingContext)
    {
        if (bindingContext == null)
        {
            throw new ArgumentNullException(nameof(bindingContext));
        }

        //Get the value
        var valueProviderResult = bindingContext.ValueProvider.GetValue(bindingContext.ModelName);
        if (valueProviderResult == ValueProviderResult.None)
        {
            // no entry
            return Task.CompletedTask;
        }

        bindingContext.ModelState.SetModelValue(bindingContext.ModelName, valueProviderResult);

        //Set the value, this has to match the property type.
        var typeConverter = TypeDescriptor.GetConverter(bindingContext.ModelType);
        var propValue = typeConverter.ConvertFromString(valueProviderResult.FirstValue.Trim());
        bindingContext.Result = ModelBindingResult.Success(propValue);
        return Task.CompletedTask;
    }
}
{% endhighlight %}

Finalement, il suffit d'aller ajouter le code suivant dans votre Startup.cs pour ajouter votre model binder à la liste des ModelBinderProviders du framework. Je le place à la première position, car je veux qu'il soit éxécuté le plus rapidemment possible.

{% highlight c# %}
services.Configure(options =>
{
    options.ModelBinderProviders.Insert(0, new TrimModelBinderProvider());
});
{% endhighlight %}
