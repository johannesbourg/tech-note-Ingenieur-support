
# Lab 3 â€” PowerShell IAM en pratique

<aside>
ðŸŽ¯

**Objectif de ce lab :** dÃ©montrer une maÃ®trise pratique de **PowerShell pour l'IAM** (Identity and Access Management) â€” gestion d'utilisateurs Windows locaux, automation par lot via CSV, et exploration du SDK **Microsoft Graph** pour Entra ID. Environnement 100 % local reproductible, sans dÃ©pendance Ã  un tenant M365 payant.

</aside>

## ðŸ–¥ï¸ Environnement de test

- **OS** : Windows 10 Pro
- **Shells utilisÃ©s** :
    - **Windows PowerShell 5.1** (icÃ´ne bleue) â€” pour LocalAccounts et RSAT
    - **PowerShell 7.x** (icÃ´ne noire) â€” pour Microsoft.Graph SDK moderne
- **Droits** : session **Administrateur**
- **PrÃ©requis financiers** : aucun. Pas de tenant M365 payant, pas d'Azure.

<aside>
ðŸ“š

**Fiche thÃ©orique associÃ©e :** *Fiche 3 â€” PowerShell pour l'IAM* dans le hub Fiches Techniques. Cette fiche couvre les concepts (verbe-nom, pipeline, modules, Microsoft.Graph, ActiveDirectory RSAT) que ce lab met en pratique.

</aside>

---

## ðŸ§ª Lab A â€” DÃ©couverte des cmdlets PowerShell

**Objectif :** se familiariser avec la structure **verbe-nom**, l'aide intÃ©grÃ©e `Get-Help`, la dÃ©couverte via `Get-Command`, et le **pipeline** (`|`).

### Commandes exÃ©cutÃ©es

```powershell
# DÃ©couvrir les cmdlets liÃ©es aux utilisateurs locaux
Get-Command -Verb Get -Noun *User* -Module Microsoft.PowerShell.LocalAccounts

# Aide contextuelle avec exemples
Get-Help Get-LocalUser -Examples

# Exemple de pipeline : top 5 processus par CPU
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name, CPU, Id
```

![Capture A1a â€” $PSVersionTable en PowerShell 7.6.4 + Get-Help System (aide intÃ©grÃ©e)](5cba2866-70b6-4801-b2f6-a615b0444f1c.png)

Capture A1a â€” $PSVersionTable en PowerShell 7.6.4 + Get-Help System (aide intÃ©grÃ©e)

![Capture A1b â€” Get-Command : exploration de l'Ã©cosystÃ¨me complet de cmdlets, alias et functions disponibles](15729e3a-bd62-4077-b1b0-8aff13ff5724.png)

Capture A1b â€” Get-Command : exploration de l'Ã©cosystÃ¨me complet de cmdlets, alias et functions disponibles

---

## ðŸ§ª Lab B â€” Gestion d'utilisateurs Windows locaux

**Objectif :** crÃ©er, lister, supprimer des utilisateurs Windows locaux en batch via PowerShell.

<aside>
âš ï¸

**Insight majeur â€” Gotcha PS 7 vs PS 5.1 :** le module `Microsoft.PowerShell.LocalAccounts` (qui contient `New-LocalUser`, `Get-LocalUser`, etc.) est **historique de Windows PowerShell 5.1**. En PS 7, ces cmdlets ne se chargent pas automatiquement et retournent : *Â« The term 'New-LocalUser' is not recognized as a name of a cmdlet Â»*.

**Solutions :**

- **Simple** â€” utiliser Windows PowerShell 5.1 (icÃ´ne bleue) en admin
- **AvancÃ©e (rester en PS 7)** â€” `Import-Module Microsoft.PowerShell.LocalAccounts -UseWindowsPowerShell`
</aside>

### Commandes exÃ©cutÃ©es (Windows PowerShell 5.1 admin)

```powershell
# CrÃ©ation batch de 3 utilisateurs de test
1..3 | ForEach-Object {
    $pwd = ConvertTo-SecureString "Test2026!$_" -AsPlainText -Force
    New-LocalUser -Name "test.user$_" -Password $pwd -FullName "Test Utilisateur $_"
}

# VÃ©rification
Get-LocalUser | Where-Object { $_.Name -like "test.user*" } | Format-Table -AutoSize

# Nettoyage (important : ne pas laisser traÃ®ner des comptes de test)
"test.user1","test.user2","test.user3" | ForEach-Object { Remove-LocalUser -Name $_ }
```

![Capture B1 â€” CrÃ©ation batch rÃ©ussie de test.user1, test.user2, test.user3 en Windows PowerShell 5.1 admin](a0a480da-53ac-4a38-b779-10038ca8ee1c.png)

Capture B1 â€” CrÃ©ation batch rÃ©ussie de test.user1, test.user2, test.user3 en Windows PowerShell 5.1 admin

![Capture B2 â€” DÃ©monstration du workflow disable-puis-remove : Disable-LocalUser test.user2 (dÃ©sactivation) puis Remove-LocalUser test.user3 (suppression)](75467381-58ac-46e9-8d21-8b48fd004512.png)

Capture B2 â€” DÃ©monstration du workflow disable-puis-remove : Disable-LocalUser test.user2 (dÃ©sactivation) puis Remove-LocalUser test.user3 (suppression)

![Capture B3 â€” Ã‰tat final aprÃ¨s opÃ©rations : test.user2 conservÃ© mais Enabled=False (dÃ©sactivÃ©). DiffÃ©rence clÃ© entre Disable (compte inactif) et Remove (compte supprimÃ©)](d0567c52-0c55-456a-9856-263226e99198.png)

Capture B3 â€” Ã‰tat final aprÃ¨s opÃ©rations : test.user2 conservÃ© mais Enabled=False (dÃ©sactivÃ©). DiffÃ©rence clÃ© entre Disable (compte inactif) et Remove (compte supprimÃ©)

---

## ðŸ§ª Lab C â€” Import CSV â†’ crÃ©ation batch d'utilisateurs

**Objectif :** automatiser la crÃ©ation d'utilisateurs depuis un fichier CSV. **C'est LE pattern support L2 le plus frÃ©quent** pour onboarding en masse.

### Ã‰tape 1 â€” CrÃ©ation du CSV via here-string

```powershell
@"
Nom,Prenom,Login,Departement,Poste
Martin,Alice,amartin,IT,Support L2
Dupont,Bob,bdupont,RH,Assistant RH
Garcia,Carla,cgarcia,Finance,Comptable
"@ | Out-File -FilePath .\lab-c-users.csv -Encoding UTF8

# VÃ©rifier le contenu
Get-Content .\lab-c-users.csv
```

![Capture C1 â€” CrÃ©ation du CSV via here-string @"..."@ + vÃ©rification avec Get-Content : header + 3 lignes utilisateurs Martin/Dupont/Garcia](c48ea148-732f-4af6-984b-f285891a8573.png)

Capture C1 â€” CrÃ©ation du CSV via here-string @"..."@ + vÃ©rification avec Get-Content : header + 3 lignes utilisateurs Martin/Dupont/Garcia

### Ã‰tape 2 â€” Import du CSV et crÃ©ation des utilisateurs

```powershell
Import-Csv -Path .\lab-c-users.csv | ForEach-Object {
    $pwd = ConvertTo-SecureString "Bienvenue2026!" -AsPlainText -Force
    New-LocalUser -Name $_.Login `
        -Password $pwd `
        -FullName "$($_.Prenom) $($_.Nom)" `
        -Description "$($_.Departement) - $($_.Poste)"
    Write-Host "âœ… CrÃ©Ã© : $($_.Login)" -ForegroundColor Green
}

# VÃ©rification
Get-LocalUser | Where-Object { $_.Name -in @("amartin","bdupont","cgarcia") } | Format-Table -AutoSize
```

![Capture C2 â€” Import-Csv en pipeline vers ForEach-Object : les 3 utilisateurs amartin, bdupont, cgarcia crÃ©Ã©s avec confirmation Â« âœ… CrÃ©Ã© Â» en vert et tableau rÃ©capitulatif](d862f62f-2a34-42dc-84d7-04b89b64da13.png)

Capture C2 â€” Import-Csv en pipeline vers ForEach-Object : les 3 utilisateurs amartin, bdupont, cgarcia crÃ©Ã©s avec confirmation Â« âœ… CrÃ©Ã© Â» en vert et tableau rÃ©capitulatif

### Ã‰tape 3 â€” Nettoyage

```powershell
"amartin","bdupont","cgarcia" | ForEach-Object { Remove-LocalUser -Name $_ }
Get-LocalUser | Where-Object { $_.Name -in @("amartin","bdupont","cgarcia") }  # doit Ãªtre vide
```

![Capture C3a â€” VÃ©rification Get-LocalUser (les 3 users prÃ©sents et Enabled=True) puis lancement de Remove-LocalUser en pipeline](5e1aae22-c511-45e2-bd3b-abef03060043.png)

Capture C3a â€” VÃ©rification Get-LocalUser (les 3 users prÃ©sents et Enabled=True) puis lancement de Remove-LocalUser en pipeline

![Capture C3b â€” Nettoyage terminÃ© : Remove-LocalUser exÃ©cutÃ© sans erreur, retour Ã  l'invite propre (aucun message = succÃ¨s silencieux)](75635864-7902-46e1-bbfc-bf5e1d1e4601.png)

Capture C3b â€” Nettoyage terminÃ© : Remove-LocalUser exÃ©cutÃ© sans erreur, retour Ã  l'invite propre (aucun message = succÃ¨s silencieux)

---

## ðŸ§ª Lab D â€” Microsoft Graph SDK pour Entra ID

**Objectif :** installer et explorer le SDK **Microsoft.Graph** â€” la porte d'entrÃ©e vers l'administration Entra ID / M365 depuis PowerShell.

<aside>
âš ï¸

**Insight majeur #1 â€” TLS 1.2 requis pour PowerShell Gallery :** depuis 2020, PowerShell Gallery exige TLS 1.2. Or Windows PowerShell 5.1 utilise par dÃ©faut TLS 1.0 / 1.1 (obsolÃ¨tes). RÃ©sultat : `Install-Module` Ã©choue avec *Â« NuGet provider is required. VÃ©rifiez votre connexion Internet Â»* alors que l'internet fonctionne parfaitement.

**Fix :** activer TLS 1.2 dans la session PowerShell avant l'install.

</aside>

### Ã‰tape 1 â€” Fix TLS 1.2 + installation (en Windows PowerShell 5.1 admin)

```powershell
# Activer TLS 1.2 pour cette session
[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor [Net.SecurityProtocolType]::Tls12

# Installer Microsoft.Graph (~600 Mo, ~40 sous-modules)
Install-Module Microsoft.Graph -Scope CurrentUser -Force -AllowClobber
```

![Capture D1 â€” RÃ©sultat aprÃ¨s tout le parcours d'installation : $PSVersionTable.PSVersion confirme PowerShell 7.6.4 + Get-Module Microsoft.Graph -ListAvailable montre la version 2.39.0 installÃ©e avec succÃ¨s](da9cc616-6f79-448d-b963-84ed8fc55bba.png)

Capture D1 â€” RÃ©sultat aprÃ¨s tout le parcours d'installation : $PSVersionTable.PSVersion confirme PowerShell 7.6.4 + Get-Module Microsoft.Graph -ListAvailable montre la version 2.39.0 installÃ©e avec succÃ¨s

<aside>
âš ï¸

**Insight majeur #2 â€” Microsoft.Graph 2.x exige PowerShell 7 :** le SDK Microsoft.Graph v2.x dÃ©pend de **.NET Standard 2.0**, disponible nativement en PowerShell 7 mais pas en Windows PowerShell 5.1. En PS 5.1, `Import-Module Microsoft.Graph.Users` Ã©choue avec *Â« Impossible de charger l'assembly 'netstandard, Version=2.0.0.0' Â»*.

**Fix :** basculer sur PowerShell 7 (icÃ´ne noire) et rÃ©installer le module lÃ -bas (chemins de modules distincts entre PS 5.1 et PS 7).

</aside>

### Ã‰tape 2 â€” RÃ©installation en PowerShell 7 + exploration

```powershell
# VÃ©rifier qu'on est bien en PS 7
$PSVersionTable.PSVersion  # Doit afficher Major : 7

# RÃ©installer dans le path de PS 7
Install-Module Microsoft.Graph -Scope CurrentUser -Force -AllowClobber

# Ajuster l'ExecutionPolicy si prompts Ã©diteur non approuvÃ©
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force

# Charger le sous-module Users
Import-Module Microsoft.Graph.Users

# Compter les cmdlets disponibles
Get-Command -Module Microsoft.Graph.Users | Measure-Object  # â†’ Count : 212

# Explorer la syntaxe (sans se connecter Ã  un tenant)
Get-Help New-MgUser -Examples
```

![Capture D2 â€” Import-Module Microsoft.Graph.Users puis Get-Command | Measure-Object : Count 212 cmdlets disponibles pour manipuler les utilisateurs Entra ID depuis PowerShell 7](542862ce-0e06-424a-b073-834e8c9ade40.png)

Capture D2 â€” Import-Module Microsoft.Graph.Users puis Get-Command | Measure-Object : Count 212 cmdlets disponibles pour manipuler les utilisateurs Entra ID depuis PowerShell 7

![Capture D3 â€” Get-Help New-MgUser -Examples : documentation intÃ©grÃ©e avec exemple complet (utilisateur Rene Magi, PasswordProfile, AccountEnabled, MailNickName, UserPrincipalName). PrÃªt Ã  Ãªtre utilisÃ© aprÃ¨s connexion Ã  un tenant Entra ID via Connect-MgGraph](b848908b-8c11-4363-855b-881f1a17a5ca.png)

Capture D3 â€” Get-Help New-MgUser -Examples : documentation intÃ©grÃ©e avec exemple complet (utilisateur Rene Magi, PasswordProfile, AccountEnabled, MailNickName, UserPrincipalName). PrÃªt Ã  Ãªtre utilisÃ© aprÃ¨s connexion Ã  un tenant Entra ID via Connect-MgGraph

---

## ðŸ§ª Lab E â€” RSAT ActiveDirectory (analyse honnÃªte)

**Objectif tentÃ© :** installer le module PowerShell RSAT pour manipuler Active Directory depuis Windows.

**Constat de terrain :** sur mon poste Windows 10 Pro, l'installation via `Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'` **Ã©choue silencieusement** â€” la commande retourne `Path : / Online : True / RestartNeeded : False` (faux succÃ¨s) mais `Get-WindowsCapability` continue de renvoyer `State : NotPresent`.

### DÃ©marche diagnostique

| **Ã‰tape** | **Commande** | **RÃ©sultat** |
| --- | --- | --- |
| 1. VÃ©rifier l'Ã©tat rÃ©el du composant | `Get-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.*'` | `State : NotPresent` |
| 2. VÃ©rifier la prÃ©sence des fichiers | `Test-Path "C:\Windows\System32\WindowsPowerShell\v1.0\Modules\ActiveDirectory"` | `False` |
| 3. VÃ©rifier le service Windows Update | `Get-Service wuauserv` | `Status : Stopped` ðŸš¨ |
| 4. DÃ©marrer le service et rÃ©essayer | `Start-Service wuauserv` puis relance | Toujours `State : NotPresent` âŒ |

### Conclusion diagnostique

Silent failure de Windows Update, probablement due Ã  :

- Une politique WSUS orphelin ou politique locale interdisant les sources Microsoft directes
- Un Ã©tat Windows Update corrompu nÃ©cessitant `DISM /Online /Cleanup-Image /RestoreHealth` puis `sfc /scannow`
- Un pare-feu / antivirus interceptant les requÃªtes `windowsupdate.microsoft.com`

<aside>
âš ï¸

**Capture terminale non disponible** â€” au moment du diagnostic Lab E, les nombreuses tentatives d'installation successives n'ont pas Ã©tÃ© capturÃ©es visuellement. Le comportement observÃ© (`State : NotPresent` malgrÃ© `Add-WindowsCapability` renvoyant `Online : True / RestartNeeded : False`) est intÃ©gralement retranscrit dans le tableau de diagnostic ci-dessus. Cette absence de capture est cohÃ©rente avec le nature mÃªme du bug : une silent failure de Windows Update qui, prÃ©cisÃ©ment, n'affiche rien d'anormal.

</aside>

<aside>
ðŸŽ“

**Ce que ce lab dÃ©montre en entretien :** la capacitÃ© Ã  **diagnostiquer mÃ©thodiquement une silent failure Windows** (vÃ©rification de l'Ã©tat rÃ©el vs message de retour, chaine des dÃ©pendances systÃ¨me), la **connaissance des commandes de rÃ©paration WU** (`DISM /RestoreHealth`, `sfc /scannow`), et la maÃ®trise de **quand escalader** un ticket vers l'Ã©quipe infra pour investigation des logs `CBS.log` et `WindowsUpdate.log`.

</aside>

---

## ðŸŽ“ CompÃ©tences pratiques dÃ©montrÃ©es

- âœ… DÃ©couverte de l'Ã©cosystÃ¨me PowerShell (structure verbe-nom, pipeline, aide intÃ©grÃ©e)
- âœ… Gestion d'utilisateurs Windows locaux via module `LocalAccounts`
- âœ… **Automation par lot depuis CSV** (pattern support L2 le plus frÃ©quent)
- âœ… **Configuration de l'environnement PowerShell** : `TLS 1.2`, `ExecutionPolicy RemoteSigned`, `PSModulePath`
- âœ… **ComprÃ©hension du paradoxe PS 5.1 vs PS 7** : LocalAccounts (5.1) vs Microsoft.Graph (7)
- âœ… Manipulation du **SDK Microsoft.Graph** (porte Entra ID / M365)
- âœ… **Diagnostic mÃ©thodique** d'une silent failure Windows Update (Lab E)
- âœ… Documentation technique honnÃªte : reconnaÃ®tre ce qui n'a pas fonctionnÃ© et pourquoi

---

## ðŸ”— Ressources associÃ©es

- **Fiche 3 â€” PowerShell pour l'IAM** : couvre la thÃ©orie complÃ¨te (verbes, pipeline, Microsoft.Graph, RSAT, best practices sÃ©curitÃ©)
- **Fiche 1 â€” Entra ID cycle de vie** : contexte cloud IAM Microsoft
- **Fiche 2 â€” Active Directory** : contexte on-premise pour comprendre l'AD que RSAT vise Ã  gÃ©rer
- **Fiche 4 â€” MFA & Conditional Access** : la couche sÃ©curitÃ© au-dessus des identitÃ©s crÃ©Ã©es dans ce lab

---

<aside>
ðŸŽ¯

**Bilan :** 4 labs pratiques validÃ©s sur environnement local + 1 lab d'analyse honnÃªte = un socle de compÃ©tences PowerShell IAM concret et vÃ©rifiable, sans dÃ©pendance Ã  un abonnement M365 payant. Chaque insight (TLS 1.2, PS 5.1 vs 7, silent failure WU) est un point d'entretien exploitable.

</aside>
