---
layout: post
title: Passer les champs d'une base de donnée VARCHAR vers NVARCHAR
modified: 2016-11-11
tags: [sécurité, sql server]
categories: [code]
author: gabrielrobert
---

Dans le cadre d'un projet d'amélioration de la sécurité d'une application, j'ai du notamment passer l'ensemble des champs VARCHAR d'une base de donnée vers NVARCHAR. La raison est bien simple: protéger le contenu des injections XSS.

Il faut en outre que:
- Le caractère &nbsp;"&gt;" devienne "&amp;gt;"
- Le caractère "&lt;" devienne "&amp;lt;"
- Le caractère &nbsp;" devienne &amp;quot;

Pour ce faire, il suffit de générer une série d'instructions "ALTER TABLE" à partir de cette requête SQL:

{% highlight sql %}
SELECT 'ALTER TABLE ' + isnull(schema_name(syo.id), 'dbo') + '.[' +  syo.name +'] ' 
    + ' ALTER COLUMN [' + syc.name + '] NVARCHAR(' + case syc.length when -1 then 'MAX' 
        ELSE convert(nvarchar(10),syc.length) end + ') '+ 
        case  syc.isnullable when 1 then ' NULL' ELSE ' NOT NULL' END +';' 
   FROM sysobjects syo 
   JOIN syscolumns syc ON 
     syc.id = syo.id 
   JOIN systypes syt ON 
     syt.xtype = syc.xtype 
   WHERE 
     syt.name = 'varchar' 
    and syo.xtype='U'
{% endhighlight %}



Finalement, ce n'est qu'une <a href="https://www.owasp.org/index.php/Main_Page">partie des éléments</a> à mettre en place pour sécuriser une application convenablement d'une possible faille XSS.
