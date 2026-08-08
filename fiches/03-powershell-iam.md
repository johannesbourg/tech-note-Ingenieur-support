# Fiche 3 — PowerShell pour IAM (Entra ID & Active Directory)

# Fiche 3 — PowerShell pour IAM

<aside>
🎯

**Pourquoi cette fiche ?**

Les offres 1 (Spécialiste Support IT L2), 4 (Expert M365/Entra ID) et 5 (Ingénieur Système AD/Entra ID Harmonie Mutuelle) exigent la maîtrise de **PowerShell** pour gérer utilisateurs, groupes, licences et politiques d'accès. Cette fiche synthétise l'essentiel, avec un plan d'entraînement **100 % local et gratuit** sur votre PC Windows.

</aside>

**Ce que vous saurez faire après cette fiche :**

- Comprendre la syntaxe PowerShell (verbes, cmdlets, pipeline)
- Créer, modifier, désactiver des utilisateurs locaux Windows
- Gérer les groupes et l'appartenance aux groupes
- Utiliser les modules `Microsoft.Graph` (Entra ID / M365) et `ActiveDirectory`
- Faire des opérations en masse (bulk) via CSV
- Automatiser des tâches IAM récurrentes

---

## 💻 1. Pourquoi PowerShell domine l'IAM Microsoft

- **Automatisation** : créer 100 utilisateurs à partir d'un CSV en une commande, au lieu de 100 clics dans l'interface
- **Reporting** : extraire la liste des comptes inactifs, des licences expirées, des membres d'un groupe
- **Audit** : traçabilité des opérations via journalisation
- **Standard Microsoft** : même syntaxe pour AD, Entra ID, Exchange, SharePoint, Teams, Intune
- **Portable** : PowerShell 7 tourne aussi sur Linux et macOS depuis 2018

<aside>
💡

**Bon à savoir :** Windows PowerShell 5.1 (natif Windows) est distinct de PowerShell 7 (multiplateforme). En 2026, on privilégie **PowerShell 7** pour tout nouveau projet. Les cmdlets IAM modernes (Microsoft.Graph) sont optimisées pour PowerShell 7.

</aside>

---

## 🔧 2. Installer PowerShell 7 sur votre PC (5 min, gratuit)

### Option A — Installateur MSI (le plus simple)

1. Ouvrir : [https://github.com/PowerShell/PowerShell/releases/latest](https://github.com/PowerShell/PowerShell/releases/latest)
2. Télécharger `PowerShell-7.X.X-win-x64.msi` (~110 Mo)
3. Double-cliquer, suivre l'assistant (Next, Next, Install)
4. ✅ Lancer via **Démarrer > PowerShell 7** (icône noire, distincte du bleu de PS 5.1)

### Option B — Via `winget` (Windows 10/11)

```powershell
winget install --id Microsoft.PowerShell --source winget
```

### Vérifier l'installation

```powershell
$PSVersionTable.PSVersion    # Doit afficher 7.X.X
```

<aside>
⚙️

**Astuce productivité :** Installer aussi **Windows Terminal** (via Microsoft Store, gratuit) : onglets, thèmes, exécution côte à côte de PS 5.1, PS 7, CMD et WSL. Indispensable pour un support qui jongle entre environnements.

⚠️ **À savoir dès le début :** PowerShell 7 est idéal pour Microsoft.Graph (Entra ID / M365), mais certaines cmdlets Windows historiques (comme `*-LocalUser`) sont natives en Windows PowerShell 5.1. Pour les labs sur les utilisateurs locaux (section 4), utilisez **Windows PowerShell 5.1** (icône bleue, pré-installé sur Windows) ou faites un `Import-Module -UseWindowsPowerShell` en PS 7.

</aside>

---

## 📖 3. Concepts fondamentaux

### 3.1 La convention **Verbe-Nom**

Chaque cmdlet PowerShell suit la syntaxe `Verbe-Nom` :

- `Get-Process` : lister les processus
- `Stop-Service` : arrêter un service
- `New-LocalUser` : créer un utilisateur local
- `Remove-ADUser` : supprimer un utilisateur AD

**Verbes standardisés** (liste courte des plus utilisés en IAM) :

| Verbe | Usage | Exemple IAM |
| --- | --- | --- |
| `Get` | Lire / lister | `Get-ADUser` |
| `Set` | Modifier | `Set-ADUser -Enabled $false` |
| `New` | Créer | `New-MgUser` |
| `Remove` | Supprimer | `Remove-ADGroup` |
| `Add` | Ajouter à une collection | `Add-ADGroupMember` |
| `Enable`/`Disable` | Activer/Désactiver | `Disable-ADAccount` |

### 3.2 Le pipeline (`|`)

La sortie d'une commande devient l'entrée de la suivante. En PowerShell, on passe des **objets**, pas juste du texte.

```powershell
Get-Process | Where-Object { $_.CPU -gt 100 } | Sort-Object CPU -Descending | Select-Object -First 5
```

Traduction : liste les processus → filtre ceux qui consomment > 100s CPU → trie par CPU décroissant → garde les 5 premiers.

### 3.3 Aide intégrée (réflexe permanent)

```powershell
Get-Help New-LocalUser              # Aide de base
Get-Help New-LocalUser -Examples    # Voir des exemples concrets
Get-Help New-LocalUser -Full        # Documentation complète
Get-Command *user*                  # Trouver toutes les cmdlets contenant "user"
Get-Member                          # Voir toutes les propriétés/méthodes d'un objet
```

<aside>
💡

**Réflexe support L2 :** avant Google, tapez `Get-Help <cmdlet> -Examples`. Réponse immédiate, même hors ligne. C'est le réflexe qui distingue un vrai admin PowerShell.

</aside>

### 3.4 Variables et objets

```powershell
$user = Get-LocalUser -Name "johannes"
$user.Enabled            # Accéder à une propriété
$user | Get-Member       # Découvrir toutes les propriétés disponibles
```

---

## 👤 4. Gestion des utilisateurs locaux Windows (testable immédiatement)

Ces commandes fonctionnent **directement sur votre PC** sans aucun serveur AD. C'est votre premier terrain d'entraînement.

⚠️ **Pré-requis :** lancer PowerShell **en tant qu'administrateur** (clic droit sur l'icône → Exécuter en tant qu'administrateur).

<aside>
⚠️

**Gotcha PowerShell 7 (fréquent en entretien) :** Les cmdlets `*-LocalUser` et `*-LocalGroup` viennent du module `Microsoft.PowerShell.LocalAccounts`, historiquement Windows PowerShell 5.1. En PowerShell 7 (icône noire), l'erreur *"The term 'New-LocalUser' is not recognized"* signifie que ce module n'est pas chargé. **Deux solutions :**

1. **La plus simple :** ouvrir directement **Windows PowerShell** (icône bleue, PS 5.1) en admin → les cmdlets `*-LocalUser` fonctionnent nativement.
2. **En PS 7 :** `Import-Module Microsoft.PowerShell.LocalAccounts -UseWindowsPowerShell` avant vos commandes (charge le module via une session de compatibilité).

💡 **Bon réflexe support L2 :** savoir diagnostiquer ce type d'erreur "cmdlet non reconnue" = problème de module non chargé. Réflexe : `Get-Module -ListAvailable | Where-Object Name -like "*<mot-clé>*"`.

</aside>

### 4.1 Lister

```powershell
Get-LocalUser                                # Tous les utilisateurs locaux
Get-LocalUser | Where-Object { $_.Enabled }  # Uniquement les actifs
Get-LocalGroup                               # Tous les groupes locaux
Get-LocalGroupMember Administrators          # Membres du groupe Administrators
```

### 4.2 Créer, modifier, supprimer

```powershell
# Créer un mot de passe sécurisé
$password = Read-Host -AsSecureString "Mot de passe pour test.user"

# Créer l'utilisateur
New-LocalUser -Name "test.user" -Password $password -FullName "Utilisateur de test" -Description "Compte créé en lab"

# L'ajouter au groupe Users
Add-LocalGroupMember -Group "Users" -Member "test.user"

# Désactiver
Disable-LocalUser -Name "test.user"

# Réactiver
Enable-LocalUser -Name "test.user"

# Changer le mot de passe
$newPwd = Read-Host -AsSecureString "Nouveau mot de passe"
Set-LocalUser -Name "test.user" -Password $newPwd

# Supprimer définitivement
Remove-LocalUser -Name "test.user"
```

### 4.3 Rapport rapide : comptes inactifs

```powershell
Get-LocalUser | Where-Object { -not $_.Enabled } | 
    Select-Object Name, LastLogon, Description | 
    Format-Table -AutoSize
```

---

## ☁️ 5. Module Microsoft.Graph pour Entra ID / M365

Microsoft.Graph est **la** bibliothèque officielle pour piloter Entra ID, M365, Teams, SharePoint via PowerShell. Elle remplace les anciens modules `AzureAD` et `MSOnline` (dépréciés en 2024).

### 5.1 Installation (locale, sans compte payant)

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force
Get-Module Microsoft.Graph -ListAvailable    # Vérifier l'installation
```

⚠️ Le module fait ~600 Mo (nombreuses sous-bibliothèques). L'installation prend 3-5 min. C'est normal.

### 5.2 Se connecter à un tenant Entra ID

```powershell
Connect-MgGraph -Scopes "User.Read.All", "Group.Read.All"
```

Un navigateur s'ouvre → authentification MFA → consentement du scope demandé. Sans tenant valide, la connexion échoue mais **le module reste utilisable en dry-run pour explorer les cmdlets**.

### 5.3 Cmdlets IAM essentielles

```powershell
# Utilisateurs
Get-MgUser -Top 10                                            # 10 premiers users
Get-MgUser -Filter "accountEnabled eq true" -All              # Users actifs
Get-MgUser -UserId "johannes@entreprise.com"                  # Un user précis
New-MgUser -DisplayName "Alice Martin" -UserPrincipalName "alice@entreprise.com" `
    -MailNickname "alice" -AccountEnabled `
    -PasswordProfile @{Password="Temp2026!"; ForceChangePasswordNextSignIn=$true}
Update-MgUser -UserId "alice@entreprise.com" -Department "IT" -JobTitle "Support L2"
Update-MgUser -UserId "alice@entreprise.com" -AccountEnabled:$false   # Désactiver
Remove-MgUser -UserId "alice@entreprise.com"                          # Supprimer (⚠️ corbeille 30j)

# Groupes
Get-MgGroup -Top 20
New-MgGroup -DisplayName "IT-Support-L2" -MailEnabled:$false -MailNickname "itsupportl2" -SecurityEnabled
Add-MgGroupMember -GroupId "<id-du-groupe>" -DirectoryObjectId "<id-user>"
Get-MgGroupMember -GroupId "<id-du-groupe>"
Remove-MgGroupMemberByRef -GroupId "<id-du-groupe>" -DirectoryObjectId "<id-user>"

# Licences
Get-MgSubscribedSku                                          # Voir les licences dispo dans le tenant
Set-MgUserLicense -UserId "alice@entreprise.com" `
    -AddLicenses @{SkuId = "<sku-id>"} -RemoveLicenses @()   # Assigner une licence

# Déconnexion propre
Disconnect-MgGraph
```

### 5.4 Exemples de rapports courants

```powershell
# Users créés ces 30 derniers jours
Get-MgUser -All -Property CreatedDateTime,DisplayName,UserPrincipalName | 
    Where-Object { $_.CreatedDateTime -gt (Get-Date).AddDays(-30) } | 
    Select-Object DisplayName, UserPrincipalName, CreatedDateTime

# Users sans MFA activé (approximation via authentication methods)
Get-MgUser -All | ForEach-Object {
    $methods = Get-MgUserAuthenticationMethod -UserId $_.Id
    if ($methods.Count -le 1) {
        [pscustomobject]@{
            User = $_.UserPrincipalName
            MethodCount = $methods.Count
        }
    }
}
```

---

## 🏢 6. Module ActiveDirectory pour AD DS on-premise

Ce module gère les utilisateurs et groupes d'un contrôleur de domaine AD (environnement entreprise on-premise).

### 6.1 Installation (Windows 10/11 Pro/Entreprise)

```powershell
# Installer les RSAT (Remote Server Administration Tools)
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
Import-Module ActiveDirectory
Get-Command -Module ActiveDirectory   # 150+ cmdlets disponibles
```

⚠️ Sans domaine AD accessible, on ne peut pas exécuter les cmdlets contre un vrai DC. **Mais on peut lire la syntaxe, l'aide, et pratiquer sur des exemples secs.**

### 6.2 Cmdlets AD essentielles

```powershell
# Utilisateurs
Get-ADUser -Filter *                                          # Tous les users
Get-ADUser -Identity "johannes" -Properties *                 # Un user + toutes ses propriétés
Get-ADUser -Filter { LastLogonDate -lt (Get-Date).AddDays(-90) } -Properties LastLogonDate    # Inactifs
New-ADUser -Name "Alice Martin" -SamAccountName "amartin" `
    -UserPrincipalName "amartin@corp.local" -Enabled $true `
    -AccountPassword (ConvertTo-SecureString "Temp2026!" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true
Set-ADUser -Identity "amartin" -Department "IT" -Title "Support L2"
Disable-ADAccount -Identity "amartin"
Unlock-ADAccount -Identity "amartin"                          # Déverrouiller (LOCK par mots de passe ratés)
Remove-ADUser -Identity "amartin"

# Groupes
Get-ADGroup -Filter *
New-ADGroup -Name "IT-Support-L2" -GroupCategory Security -GroupScope Global
Add-ADGroupMember -Identity "IT-Support-L2" -Members "amartin", "bmartin"
Get-ADGroupMember -Identity "IT-Support-L2"
Remove-ADGroupMember -Identity "IT-Support-L2" -Members "amartin" -Confirm:$false

# Comptes verrouillés (cas support ultra fréquent)
Search-ADAccount -LockedOut
```

### 6.3 Cas support réel : déverrouiller un compte

```powershell
# Un user appelle : "je n'arrive plus à me connecter"
Get-ADUser -Identity "johannes" -Properties LockedOut, LastBadPasswordAttempt, BadPwdCount
# S'il est verrouillé :
Unlock-ADAccount -Identity "johannes"
# Puis réinitialiser le mot de passe si besoin :
Set-ADAccountPassword -Identity "johannes" -Reset -NewPassword (Read-Host -AsSecureString)
Set-ADUser -Identity "johannes" -ChangePasswordAtLogon $true
```

---

## 📊 7. Opérations en masse via CSV (compétence à fort impact)

### 7.1 Créer 100 utilisateurs à partir d'un CSV

**Fichier `nouveaux-users.csv` :**

```jsx
Nom,Prenom,Login,Departement,Poste
Martin,Alice,amartin,IT,Support L2
Dupont,Bob,bdupont,RH,Assistant RH
Garcia,Carla,cgarcia,Finance,Comptable
```

**Script d'import :**

```powershell
Import-Csv -Path ".\nouveaux-users.csv" | ForEach-Object {
    $pwd = ConvertTo-SecureString "Bienvenue2026!" -AsPlainText -Force
    New-LocalUser -Name $_.Login `
        -Password $pwd `
        -FullName "$($_.Prenom) $($_.Nom)" `
        -Description "$($_.Departement) - $($_.Poste)"
    Write-Host "✅ Créé : $($_.Login)" -ForegroundColor Green
}
```

### 7.2 Export d'un annuaire vers CSV (pour audit ou reporting)

```powershell
Get-LocalUser | 
    Select-Object Name, Enabled, LastLogon, Description | 
    Export-Csv -Path ".\audit-comptes-locaux.csv" -NoTypeInformation -Encoding UTF8
```

### 7.3 Même logique en Entra ID

```powershell
Import-Csv -Path ".\new-cloud-users.csv" | ForEach-Object {
    $upn = "$($_.Login)@entreprise.com"
    New-MgUser -DisplayName "$($_.Prenom) $($_.Nom)" `
        -UserPrincipalName $upn -MailNickname $_.Login `
        -AccountEnabled -Department $_.Departement `
        -PasswordProfile @{
            Password = "Bienvenue2026!"
            ForceChangePasswordNextSignIn = $true
        }
    Write-Host "✅ Créé dans Entra ID : $upn" -ForegroundColor Green
}
```

---

## 🔍 8. Scénarios support L2 concrets

### Scénario 1 — "Je ne peux plus me connecter au VPN"

```powershell
# Est-ce que le compte AD est verrouillé ou désactivé ?
Get-ADUser -Identity "johannes" -Properties LockedOut, Enabled, LastLogonDate, PasswordExpired
# Si LockedOut = True → Unlock-ADAccount
# Si Enabled = False → Enable-ADAccount
# Si PasswordExpired = True → Set-ADAccountPassword
```

### Scénario 2 — "J'ai perdu l'accès à une boîte partagée"

```powershell
# Vérifier l'appartenance aux groupes
Get-ADPrincipalGroupMembership -Identity "johannes" | Select-Object Name
# Ajouter au groupe qui donne accès
Add-ADGroupMember -Identity "BAL-Support-Prod" -Members "johannes"
```

### Scénario 3 — "Onboarding d'un nouvel employé"

Création utilisateur AD + Entra ID + affectation groupes + licence M365 en un script :

```powershell
$newUser = @{
    Name = "Alice Martin"
    SamAccountName = "amartin"
    UPN = "amartin@entreprise.com"
    Department = "IT"
    Title = "Support L2"
    Groups = @("IT-Support-L2", "VPN-Users", "M365-E3")
}

# 1. AD DS
New-ADUser -Name $newUser.Name -SamAccountName $newUser.SamAccountName `
    -UserPrincipalName $newUser.UPN -Enabled $true `
    -Department $newUser.Department -Title $newUser.Title `
    -AccountPassword (ConvertTo-SecureString "Bienvenue2026!" -AsPlainText -Force) `
    -ChangePasswordAtLogon $true

# 2. Groupes
foreach ($grp in $newUser.Groups) {
    Add-ADGroupMember -Identity $grp -Members $newUser.SamAccountName
}

# 3. Attendre la synchro Entra ID (AAD Connect → généralement 30 min)
# Puis assigner licence M365
Set-MgUserLicense -UserId $newUser.UPN -AddLicenses @{SkuId = "<sku-e3>"} -RemoveLicenses @()
```

### Scénario 4 — "Offboarding d'un départ"

```powershell
# 1. Désactiver le compte AD (ne pas supprimer immédiatement)
Disable-ADAccount -Identity "bmartin"
# 2. Réinitialiser le mot de passe (éviter tentative de reconnexion)
Set-ADAccountPassword -Identity "bmartin" -Reset -NewPassword (ConvertTo-SecureString ([System.Guid]::NewGuid().ToString()) -AsPlainText -Force)
# 3. Retirer de tous les groupes
Get-ADPrincipalGroupMembership "bmartin" | Where-Object { $_.Name -ne "Domain Users" } | 
    ForEach-Object { Remove-ADGroupMember -Identity $_ -Members "bmartin" -Confirm:$false }
# 4. Déplacer vers OU "Départ"
Move-ADObject -Identity (Get-ADUser "bmartin").DistinguishedName -TargetPath "OU=Departs,DC=corp,DC=local"
# 5. Révoquer les licences M365
Set-MgUserLicense -UserId "bmartin@corp.com" -AddLicenses @() -RemoveLicenses @("<sku-e3>")
# 6. Conserver 30 jours puis supprimer (règle RGPD interne)
```

---

## 🏗️ 9. Plan d'entraînement local (aucun tenant requis)

### Lab A — Découverte PowerShell 7 (30 min)

**Ouvrir PowerShell 7 en admin, puis :**

```powershell
$PSVersionTable                        # Vérifier la version
Get-Help                               # L'aide
Get-Command | Measure-Object           # Combien de cmdlets dispo ?
Get-Command -Verb Get -Noun *user*     # Toutes les cmdlets Get-*User*
Get-Alias                              # Voir les raccourcis (ls, cat, etc.)
```

📸 **Captures suggestées :** `A1-version-help.png`, `A2-command-explore.png`

### Lab B — Utilisateurs locaux Windows (45 min)

```powershell
# Créer 3 utilisateurs de test
1..3 | ForEach-Object {
    $pwd = ConvertTo-SecureString "Test2026!$_" -AsPlainText -Force
    New-LocalUser -Name "test.user$_" -Password $pwd -FullName "Test Utilisateur $_"
}
# Les lister
Get-LocalUser | Where-Object { $_.Name -like "test.user*" } | Format-Table -AutoSize

# En désactiver 1, en supprimer 1
Disable-LocalUser -Name "test.user2"
Remove-LocalUser -Name "test.user3"

# Vérifier
Get-LocalUser | Where-Object { $_.Name -like "test.user*" }

# Nettoyage complet
Get-LocalUser | Where-Object { $_.Name -like "test.user*" } | Remove-LocalUser
```

📸 **Captures suggestées :** `B1-creation-3-users.png`, `B2-desactivation.png`, `B3-nettoyage.png`

### Lab C — CSV : import / export en masse (30 min)

1. Créer un fichier `.\lab-c-users.csv` (voir template section 7.1)
2. Importer avec le script fourni
3. Vérifier avec `Get-LocalUser | Export-Csv...`
4. Nettoyer

📸 **Captures suggestées :** `C1-import-csv.png`, `C2-export-audit.png`

### Lab D — Microsoft.Graph en dry-run (45 min)

```powershell
Install-Module Microsoft.Graph -Scope CurrentUser -Force
Get-Command -Module Microsoft.Graph.Users | Measure-Object    # ~200 cmdlets pour Users
Get-Help New-MgUser -Examples                                 # Explorer la syntaxe
Get-Help Get-MgUser -Detailed                                 # Parcourir les paramètres
```

📸 **Captures suggestées :** `D1-install-mggraph.png`, `D2-help-newmguser.png`

### Lab E — RSAT ActiveDirectory syntaxe (30 min)

(Windows 10/11 Pro uniquement)

```powershell
Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
Import-Module ActiveDirectory
Get-Command -Module ActiveDirectory | Measure-Object
Get-Help New-ADUser -Examples
Get-Help Search-ADAccount -Full     # Comprendre les filtres
```

📸 **Captures suggestées :** `E1-rsat-install.png`, `E2-help-newaduser.png`

---

## 🎓 10. Questions d'entretien fréquentes

- **Q1 — Différence entre AzureAD, MSOnline et Microsoft.Graph ?**
    - **MSOnline** (le plus ancien) : module hérité pour Office 365, déprécié depuis 2024.
    - **AzureAD** : module intermédiaire pour Azure AD, aussi déprécié depuis 2024.
    - **Microsoft.Graph** : le module officiel actuel, unifié pour Entra ID + M365 + Teams + SharePoint + Intune. **C'est celui à utiliser** en 2026.
    
    En entretien : mentionner qu'on utilise Microsoft.Graph, tout en sachant lire un ancien script en AzureAD s'il faut le migrer.
    
- **Q2 — Comment créer 100 utilisateurs à la fois ?**
    1. Préparer un fichier CSV avec les colonnes nécessaires (login, nom, département, etc.)
    2. `Import-Csv` pour charger le CSV
    3. Pipeline `| ForEach-Object { New-MgUser ... }` pour créer chaque ligne
    4. Toujours logger le résultat (succès / échec) et faire un dry-run sur 2-3 lignes avant le lot complet
- **Q3 — Comment identifier les comptes inactifs depuis 90 jours ?**
    
    **AD DS :**
    
    ```powershell
    Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 -UsersOnly | 
        Select-Object Name, LastLogonDate
    ```
    
    **Entra ID (via Microsoft.Graph) :**
    
    ```powershell
    Get-MgUser -All -Property SignInActivity | 
        Where-Object { $_.SignInActivity.LastSignInDateTime -lt (Get-Date).AddDays(-90) }
    ```
    
    Cas support : nettoyer périodiquement pour libérer des licences et réduire la surface d'attaque.
    
- **Q4 — Quelle différence entre `Disable-ADAccount` et `Remove-ADUser` ?**
    - `Disable-ADAccount` : désactive le compte mais le conserve. Réversible.
    - `Remove-ADUser` : supprime définitivement. En AD DS, c'est irréversible sans restauration depuis une sauvegarde AD ou depuis l'AD Recycle Bin (si activé).
    
    Bonne pratique en offboarding : **désactiver d'abord**, attendre 30-90 jours (rétention légale + éviter un rappel immédiat), puis supprimer.
    
- **Q5 — Comment gérer les secrets (mots de passe, tokens) dans un script PowerShell ?**
    - ❌ **JAMAIS en clair** dans le script (`$pwd = "Motdepasse123"`)
    - ✅ Demander interactivement : `Read-Host -AsSecureString`
    - ✅ Stocker dans un **coffre-fort** : SecretManagement module + backend (Azure Key Vault, Windows Credential Manager)
    - ✅ Pour Entra ID : utiliser une **app registration** avec certificat, pas un secret
- **Q6 — Comment tester un script sans risque avant production ?**
    - Utiliser le paramètre `-WhatIf` (dry-run) : `New-MgUser -WhatIf` montre ce qui serait fait sans le faire
    - Le paramètre `-Confirm` demande confirmation avant chaque opération
    - Toujours tester sur un environnement de dev/lab avant d'exécuter en prod
    - Versionner les scripts (Git), documenter les changements, faire relire par un pair

---

## 🔐 11. Bonnes pratiques sécurité

<aside>
🚨

**Règles d'or PowerShell IAM en production :**

- Toujours **tester avec `-WhatIf`** avant une opération destructive
- **Journaliser** toutes les actions (`Start-Transcript`)
- Ne jamais stocker de mot de passe en clair dans un script
- Privilégier les **comptes de service** à permissions minimales (least privilege)
- Pour Entra ID : **App Registration + certificat** plutôt que compte admin nominatif
- **Signer** les scripts en production (Set-AuthenticodeSignature)
- Fixer l'ExecutionPolicy : `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`
</aside>

---

## 📚 12. Sources officielles Microsoft

- **PowerShell** : [https://learn.microsoft.com/en-us/powershell/](https://learn.microsoft.com/en-us/powershell/)
- **Microsoft.Graph PowerShell** : [https://learn.microsoft.com/en-us/powershell/microsoftgraph/](https://learn.microsoft.com/en-us/powershell/microsoftgraph/)
- **Active Directory cmdlets** : [https://learn.microsoft.com/en-us/powershell/module/activedirectory/](https://learn.microsoft.com/en-us/powershell/module/activedirectory/)
- **Local Accounts cmdlets** : [https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/)
- **Windows Terminal** : [https://learn.microsoft.com/en-us/windows/terminal/](https://learn.microsoft.com/en-us/windows/terminal/)
- **PowerShell Gallery** (modules communautaires) : [https://www.powershellgallery.com/](https://www.powershellgallery.com/)

---

<aside>
➡️

**Prochaine étape pour VOUS :**

1. Installer PowerShell 7 sur votre PC (section 2)
2. Faire le **Lab A** (découverte) et le **Lab B** (utilisateurs locaux) : 100 % gratuit, 100 % local, aucun compte payant
3. Revenir avec les captures : on créera le **Lab 3 — PowerShell IAM en pratique** dans Notion, même modèle que le Lab 2 Linux
</aside>