---
layout: post
title: "Outiller son développement sous Windows: Un profil PowerShell propre"
modified: 2017-05-23
tags: [powershell, outils, productivité, terminal]
categories: [code]
author: thatfrankdev
---

Si tu as déjà jasé développement avec moi, il y a de fortes chances que PowerShell soit venu sur le sujet. Avant l'arrivée de PowerShell sur Windows en 2006, les développeurs Windows qui voulaient tirer profit de la ligne de commande étaient pris avec le bon vieux BATCH (cmd.exe). Bien qu'on puisse se débrouiller et arriver à un certain niveau avec le terminal classique, c'est très loin d'être optimal.

PowerShell nous amène beaucoup plus loin et adresse plusieurs irritants pour les développeurs qui veulent utiliser la ligne de commandes dans leur routine quotidienne. Notamment, le support de [script de profil](https://msdn.microsoft.com/powershell/reference/5.1/Microsoft.PowerShell.Core/about/about_Profiles) qui permet de mettre à notre main nos sessions PowerShell.

Ce petit guide propose une façon simple d'avoir un profil PowerShell propre, bien structué et réutilisable, en quelques étapes facile.

## Créer un répertoire Git pour les scripts de profile

Le meilleur moyen de stocker et éventuellement partager notre profil PowerShell est de le versionner dans un répertoire Git. De cette façon, notre boîte à outil est accessible de partout. J'ai personnellement choisi [GitHub](https://github.com/thatfrankdev/powershell-dev-environment), mais plusieurs options s'offrent à toi. [Gitlab](https://about.gitlab.com) et [Bitbucket](https://bitbucket.org/) offrent gratuitement des répertoires privés, si c'est important pour toi.

### La structure du répertoire
```
powershell-dev-environment/
├── src/
│   ├── profile.ps1
│   ├── _imports.ps1
│   ├── _prompt.ps1
│   └── _ui.ps1
└── install.ps1
```

## Le point d'entrée

Le fichier `profile.ps1` est le point d'entrée de notre profil.

```powershell
# profile.ps1

$env:devProfileDir = $PSScriptRoot

. "$env:devProfileDir\_imports.ps1";
. "$env:devProfileDir\_ui.ps1";
. "$env:devProfileDir\_prompt.ps1";
```

Noter qu'on stocke le chemin vers notre profil dans une variable d'environnement `
$env:devProfileDir`. Ça peut être utile pour y revenir rapidement.

Le fichier `_imports.ps1` sert à définir tous les modules externes qu'on souhaite charger dans chaque session PowerShell.

```powershell
# _imports.ps1

Import-Module posh-git
# et ainsi de suite...
```

_ATTENTION, certains modules peuvent prendre beaucoup de temps à s'initialiser (oui c'est toi que je regarde [Carbon](http://get-carbon.org/)), alors il est important de bien réfléchir à ce qu'on veut exécuter à chaque démarrage de session._

## Les couleurs

Les différentes couleurs d'affichage sont configurées dans le fichier `_ui.ps1`

```powershell
# _ui.ps1

$console = $host.UI.RawUI

$console.ForegroundColor = "white"
$console.BackgroundColor = "black"

$colors = $host.PrivateData
$colors.VerboseForegroundColor = "white"
$colors.VerboseBackgroundColor = "blue"
$colors.WarningForegroundColor = "yellow"
$colors.WarningBackgroundColor = "darkgreen"
$colors.ErrorForegroundColor = "white"
$colors.ErrorBackgroundColor = "red"

Clear-Host
```

## La fonction Prompt

Prompt est une fonction spéciale de PowerShell qui est exécutée pour générer la zone de saisie de commande. Lorsqu'elle est définie dans le script de profil, elle nous donne entièrement le contrôle sur le rendu de cette zone. Voici ma fonction Prompt :

```powershell
# _prompt.ps1

function Prompt {    
    $realLASTEXITCODE = $LASTEXITCODE

    $Host.UI.RawUI.ForegroundColor = $GitPromptSettings.DefaultForegroundColor

    Write-Host
    Write-Host "$ENV:USERNAME@" -NoNewline -ForegroundColor DarkYellow
    Write-Host "$ENV:COMPUTERNAME" -ForegroundColor Magenta -NoNewline
    Write-Host (Get-Date -Format " | MM/dd/yyyy | HH:MM:ss") -ForegroundColor DarkGray

    Write-Host $($(Get-Location) -replace ($env:USERPROFILE).Replace('\','\\'), "~") -NoNewline -ForegroundColor Gray

    Write-VcsStatus

    Write-Host

    $global:LASTEXITCODE = $realLASTEXITCODE

    return ""
}
```

Et voici un aperçu du résultat dans le terminal :

![Aperçu de la zone de saisie du terminal](https://4.bp.blogspot.com/-okseuC5zGMw/WSQZLLHGO6I/AAAAAAAAADE/IUAV-9_sxMcVmFn3p640XzbVguBBozNvgCKgB/s640/prompt_sample.png)

## Exécuter les scripts de profil au démarrage des sessions

Si tu es comme moi, tu aimes avoir tout ton code à la même place sur ton poste. Donc en principe ce répertoire Git nouvellement créé se trouvera à un endroit comme `c:\users\moi\dev\powershell-dev-environment`

Ce qu'on veut, c'est que PowerShell exécute notre script au démarrage des sessions. Par défaut, une variable `$profile` est définie et elle a pour valeur le chemin vers le script de profil de la session uilisateur en cours. Sa valeur est typiquement quelque chose comme `C:\Users\fcote\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`.

Ce qu'on doit faire, c'est créer ce fichier s'il n'existe pas encore, et y ajouter la ligne suivante.

```
. "c:\users\moi\dev\powershell-dev-environment\src\profile.ps1"
```

À partir de maintenant, PowerShell exécutera notre script au début de chaque session.

## La touche finale : le script d'installation

L'étape précédente peut facilement être automatisée avec un petit script d'installation tout simple.

```powershell
# install.ps1

$installedHint = "#https://github.com/thatfrankdev/powershell-dev-environment"
$installed = $false

if(-not (Test-Path $profile)){
    New-Item $profile
}
else{    
    $installed = (Select-String -Path $profile -pattern $installedHint | Measure-Object).Count -gt 0
}

if(-not $installed){
    Add-Content $profile ""
    Add-Content $profile ". `"$PSScriptRoot\src\profile.ps1`" $installedHint"
    powershell.exe
}
```

Noter l'utilisation de la variable `$installedHint`. L'insertion d'un commentaire au bout de la ligne permet à ce script d'être roulé plusieurs fois sans multiplier l'initialisation de notre profil.

## La prochaine étape

On a maintenant une structure de base très intéressante pour construire un profil PowerShell réutilisable.

Dans un prochain article, quelques fonctions sur mesure très intéressantes à placer dans ton nouveau profil Powershell!