---
layout: post
title: Déployer une application .NET Core avec AppVeyor Deployment Agent
modified: 2016-11-03
tags: [appveyor, aspnet core, déploiement]
categories: [code]
author: gabrielrobert
---

Les applications .NET Core agissent différemment des applications traditionnelles ASP .NET sur la stack Windows / IIS. En effet, l'application lorsque démarée, procède à des locks sur l'ensemble des fichiers utilisés.

J'entends par là:
- les .DLL
- Les .EXE
- Les fichiers de logs

Cela rend donc impossible le déploiement. Si vous essayez, vous aurez sans doute cette problématique:

{% highlight shell_session %}
Deployment job started
Found 1 deployable artifacts.
Deploying Web application TestProject.Web
Downloading artifact package TestProject.Web.zip (20,523,009 bytes)
Execute before-deploy.ps1 script for TestProject.Web application
All file locks have been released!
Website 'testproject.com' already exists with 'testproject.com' app pool and root directory at 'F:\websites\custom\TestProject'
Updating website bindings to:
 - http *:80:testproject.com
Application path with expanded environment variables: F:\websites\custom\TestProject
Updating web site 'testproject.com' contents from Zip archive C:\Users\appveyor\AppData\Local\Temp\1\2hylwzwk.w4o\TestProject.Web.zip
Info: Updating file (testproject.com\before-deploy.ps1).
Info: Updating file (testproject.com\refs\TestProject.Web.exe).
Info: Updating file (testproject.com\TestProject.Application.dll).
Warning: An error was encountered when processing operation 'Create File' on 'TestProject.Application.dll'. 
Retrying operation 'Update' on object filePath (testproject.com\TestProject.Application.dll). Attempt 1 of 5.
Warning: An error was encountered when processing operation 'Create File' on 'TestProject.Application.dll'. 
Retrying operation 'Update' on object filePath (testproject.com\TestProject.Application.dll). Attempt 2 of 5.
Warning: An error was encountered when processing operation 'Create File' on 'TestProject.Application.dll'. 
Retrying operation 'Update' on object filePath (testproject.com\TestProject.Application.dll). Attempt 3 of 5.
Warning: An error was encountered when processing operation 'Create File' on 'TestProject.Application.dll'. 
Retrying operation 'Update' on object filePath (testproject.com\TestProject.Application.dll). Attempt 4 of 5.
Warning: An error was encountered when processing operation 'Create File' on 'TestProject.Application.dll'. 
Retrying operation 'Update' on object filePath (testproject.com\TestProject.Application.dll). Attempt 5 of 5.
Web Deploy cannot modify the file
{% endhighlight %}


Pour résoudre le problème, il faut créer un fichier "before-deploy.ps1" directement dans le root du projet Web et mettre le code suivant:

{% highlight powershell %}
Restart-WebAppPool "testproject.com"
Write-Host "All file locks have been released!"
{% endhighlight %}

Pas de craintes, Restart-WebAppPool termine les requêtes courantes avant de redémarrer l'application pool.

Finalement, ne surtout pas oublier de l'inclure dans votre project.json:

{% highlight json %}
"publishOptions": {
    "include": [
      "before-deploy.ps1"
    ]
  }
{% endhighlight %}
