# Fiche 4 — MFA & Conditional Access

<aside>
🎯

**Objectif de cette fiche** : maîtriser les méthodes MFA d'Entra ID, comprendre Conditional Access comme moteur de décision Zero Trust, et savoir troubleshooter les blocages d'accès. Contenu 2026-ready avec passkeys FIDO2, Authentication Strengths et Mandatory MFA Phase 2.

</aside>

## 1. Pourquoi la MFA est le pilier de l'IAM moderne

Le vol de mot de passe reste **le vecteur numéro 1** des compromissions d'identités en 2026. Selon Microsoft, activer la MFA bloque **plus de 99,2 %** des attaques par compromission de compte.

**Principe :** exiger au moins 2 facteurs parmi 3 catégories différentes pour prouver son identité.

<aside>
⚠️

**Piège fréquent :** un mot de passe + une question secrète = **PAS de MFA**. Les deux appartiennent à la même catégorie (« ce que vous savez »). La vraie MFA combine **au moins 2 catégories différentes**.

</aside>

## 2. Les 3 facteurs d'authentification

| **Catégorie** | **Exemples** | **Force** | **Faiblesses** |
| --- | --- | --- | --- |
| Ce que vous **SAVEZ** | Mot de passe, PIN, question secrète | 🔴 Faible | Phishing, brute-force, réutilisation |
| Ce que vous **AVEZ** | Téléphone, Yubikey FIDO2, TOTP app, passkey | 🟢 Forte | Perte physique, SIM swap (SMS uniquement) |
| Ce que vous **ÊTES** | Empreinte digitale, Face ID, reconnaissance vocale | 🟢 Très forte | Difficile à révoquer si compromis |

## 3. Méthodes MFA supportées par Entra ID (2026)

| **Méthode** | **Sécurité** | **UX** | **Statut 2026** |
| --- | --- | --- | --- |
| **Passkeys FIDO2** (device-bound ou synced) | 🟢🟢🟢 Phishing-resistant | Excellent (biométrie) | ✨ Recommandé Microsoft #1 |
| **Clés FIDO2 physiques** (Yubikey, Feitian) | 🟢🟢🟢 Phishing-resistant | Bon (branchement USB/NFC) | ✅ Recommandé pour admins |
| **Windows Hello for Business** | 🟢🟢🟢 Phishing-resistant | Excellent (natif Windows) | ✅ Recommandé sur postes Windows |
| **Certificate-based auth (CBA)** | 🟢🟢🟢 Phishing-resistant | Transparent | ✅ Grandes entreprises / secteur public |
| **Microsoft Authenticator** (push + number matching) | 🟢🟢 Résistant MFA fatigue | Très bon | ✅ Standard entreprises |
| **OATH TOTP** (Google/Microsoft Authenticator) | 🟢 Correct | Bon (code à 6 chiffres) | ✅ Fallback fiable |
| **Temporary Access Pass (TAP)** | 🟡 Temporaire | Simple (code jetable) | ✅ Onboarding / récupération |
| **SMS / Voice call** | 🔴 Phishable (SIM swap) | Simple | ⚠️ Fallback ultime uniquement |

<aside>
📢

**Direction Microsoft 2026 :** aller vers le **passwordless** (passkeys + Windows Hello + FIDO2). Le mot de passe disparaît progressivement au profit de méthodes phishing-resistant.

</aside>

## 4. Passkeys FIDO2 — le futur de l'authentification

Depuis 2024, Entra ID supporte nativement les **passkeys FIDO2** (device-bound ET synced), et en **mars 2026** Microsoft a ajouté les **passkey profiles** et le **support Linux SSO** avec MFA phishing-resistant.

### 4.1 Qu'est-ce qu'une passkey ?

Une passkey est une **paire de clés cryptographiques** (publique/privée) liée à un service :

- La clé **publique** est stockée chez le fournisseur (Entra ID)
- La clé **privée** ne quitte JAMAIS votre appareil (téléphone, PC)
- L'authentification se fait par **signature cryptographique** + biométrie locale

### 4.2 Pourquoi c'est révolutionnaire

- **Impossible à phisher** : aucun secret n'est jamais tapé ou envoyé
- **Impossible à réutiliser** : chaque passkey est unique à un service
- **UX excellente** : déverrouillage biométrique (Face ID, empreinte)
- **Cross-device** avec les synced passkeys (iCloud Keychain, Google Password Manager)

### 4.3 En pratique dans Entra ID

<aside>
💡

Pour activer les passkeys dans Entra ID : **Entra Admin Center → Protection → Authentication methods → FIDO2 security key → Enable**. Puis dans le portail utilisateur `myaccount.microsoft.com`, l'utilisateur peut enregistrer une passkey.

</aside>

## 5. Number Matching — fin des MFA fatigue attacks

**Depuis mai 2023** (imposé par Microsoft), les notifications push Microsoft Authenticator exigent le **number matching** :

- L'écran de connexion affiche un nombre à 2 chiffres (ex : `27`)
- L'utilisateur doit **saisir ce nombre** dans son app Authenticator
- Un simple « Approuver » ne suffit plus

### Pourquoi c'est critique

Avant number matching, les attaquants pouvaient spammer des push notifications jusqu'à ce que l'utilisateur, agacé ou distrait, appuie sur « Approuver ». Résultat : compromission silencieuse.

Le number matching **force la vérification consciente** de la localisation de l'app qui demande l'auth.

<aside>
🎓

**Point d'entretien :** *« Le number matching est la contre-mesure de Microsoft contre les MFA fatigue attacks : au lieu de simplement approuver un push, l'utilisateur doit saisir un nombre affiché à l'écran, ce qui empêche l'approbation automatique par réflexe. »*

</aside>

## 6. Authentication Strengths — le nouveau paradigme

Avant : Conditional Access exigeait simplement « MFA required » sans préciser la méthode. Un utilisateur pouvait donc satisfaire la policy avec un SMS (faible) alors qu'on voulait une clé FIDO2.

**Authentication Strengths (GA 2023)** résout ce problème : une policy CA peut exiger une **combinaison spécifique** de méthodes d'auth.

### 6.1 Les 3 strengths built-in

| **Strength** | **Méthodes acceptées** | **Cas d'usage** |
| --- | --- | --- |
| **Multifactor authentication** | Toute combinaison de 2 facteurs | Baseline utilisateurs standards |
| **Passwordless MFA** | Windows Hello, passkeys, FIDO2, MS Authenticator (mode passwordless) | Applications sensibles |
| **Phishing-resistant MFA** | Passkeys FIDO2, Windows Hello, CBA uniquement | Admins, données critiques |

### 6.2 Custom Authentication Strengths

On peut créer jusqu'à **15 strengths personnalisées** pour matcher des politiques internes précises (ex : « Authenticator + biométrie uniquement »).

## 7. Mandatory MFA Azure — Phase 2 (depuis oct. 2025)

<aside>
🚨

**Depuis le 1er octobre 2025**, Microsoft impose la MFA pour **toutes les opérations Create / Update / Delete** sur Azure Resource Manager, quel que soit le client utilisé : Azure CLI, Azure PowerShell, Azure mobile app, IaC (Terraform, Bicep), REST API. Les opérations Read n'exigent pas MFA.

</aside>

### Impact concret

- **DevOps pipelines** : les service principals doivent migrer vers des workload identities
- **Scripts d'automation** : les comptes utilisateurs utilisés comme service accounts doivent être remplacés par des managed identities
- **Admins mobiles** : Azure Mobile App requiert MFA à chaque action
- **Postponement possible jusqu'au 1er sept. 2025** (délai maintenant expiré)

### À savoir pour L2

- Version minimale : **Azure CLI 2.76** et **Azure PowerShell 14.3**
- Les vieilles versions échouent avec des erreurs MFA obscures

## 8. Conditional Access — Fondamentaux

Conditional Access (CA) est le **moteur de décision Zero Trust** d'Entra ID. Structure logique :

### 8.1 Structure d'une policy CA

```
SI (Assignments) ALORS (Access Controls)
```

**Assignments — QUI et QUOI :**

- **Users or workload identities** : quels utilisateurs / groupes / rôles
- **Cloud apps or actions** : quelles applications (M365, Azure, apps custom)
- **Conditions** :
    - **User risk** (compromis, sign-in from anonymous IP...)
    - **Sign-in risk** (leaked credentials, impossible travel...)
    - **Device platforms** (Windows, iOS, Android...)
    - **Locations** (pays, IP ranges, trusted networks)
    - **Client apps** (browser, mobile app, legacy protocols)
    - **Device state** (compliant, hybrid joined)

**Access Controls — QUE FAIRE :**

- **Grant** :
    - Require MFA / Require authentication strength
    - Require device to be marked as compliant
    - Require Hybrid Azure AD joined device
    - Require approved client app / app protection policy
    - Require password change
- **Block** : refus pur et simple
- **Session** : limiter la durée de session, mode navigation restreinte, App-Enforced Restrictions

### 8.2 Ordre d'évaluation

<aside>
⚠️

**Règle critique :** toutes les policies CA applicables sont évaluées et **combinées avec un ET logique**. Si UNE policy dit « Block », l'accès est refusé même si les autres autorisent. Toujours prévoir des exclusions cohérentes (break-glass !).

</aside>

## 9. Templates CA recommandés Microsoft (baseline Zero Trust)

Microsoft fournit des **templates** dans le portail. À déployer par ordre de priorité :

1. **🔒 Require MFA for admins** — obligatoire, tous les rôles privilégiés
2. **🔒 Block legacy authentication** — POP, IMAP, SMTP basic auth bypasse MFA → à bloquer absolument
3. **🔒 Require MFA for all users** — Zero Trust baseline
4. **📱 Require compliant device** — combiner avec Intune
5. **🌍 Block access from unsupported locations** — pays où l'entreprise n'opère pas
6. **⚠️ Risk-based policies** :
    - Sign-in risk **medium+** → require MFA
    - User risk **high** → require password change + MFA
7. **🔐 Require phishing-resistant MFA for admins** — évolution logique, passkeys/FIDO2 obligatoires pour rôles Global Admin

<aside>
💡

**Astuce déploiement :** toujours démarrer une policy en mode **Report-only** pour observer l'impact avant de basculer en `On`. Le portail Entra fournit un « CA What-If » pour tester avec un utilisateur donné.

</aside>

## 10. Break-glass accounts — la règle d'or

<aside>
🚨

**Non négociable :** toute tenant Entra ID doit avoir **2 comptes break-glass** (« bris de glace »).

</aside>

### Caractéristiques

- Rôle **Global Administrator** permanent
- **Exclus de TOUTES les policies MFA et Conditional Access**
- Mot de passe **très long** (25+ caractères), stocké physiquement en **coffre-fort** (2 endroits distincts)
- **Cloud-only** (pas synchronisés depuis AD on-premise)
- **Nommage neutre** (ex : `emergency-01@tenant.onmicrosoft.com`)
- **Monitorés** : alerte Sentinel / Log Analytics à chaque sign-in

### Pourquoi ils sauvent la vie

Scénarios réels où ils servent :

- Le service MFA Microsoft est down (rare mais arrive)
- Une policy CA mal configurée verrouille tous les admins
- Le fournisseur d'identité fédéré (ADFS, IdP tiers) est en panne
- L'unique admin part et supprime les autres comptes admin

<aside>
🎓

**Point d'entretien majeur :** *« La règle non négociable en Entra ID, c'est d'avoir 2 comptes break-glass exclus de toutes les policies Conditional Access et MFA, avec un mot de passe long stocké en coffre et une alerte de sign-in monitorée. Sans eux, un incident MFA peut verrouiller définitivement une tenant. »*

</aside>

## 11. Scénarios support L2 concrets

### 🎫 Scénario 1 : « J'ai perdu mon téléphone MFA »

**Diagnostic :**

- Vérifier l'identité de l'utilisateur (question de sécurité / contact manager)
- Consulter `Security → Authentication methods` du compte utilisateur dans le portail Entra

**Action :**

1. Révoquer la méthode Microsoft Authenticator compromise
2. Générer un **Temporary Access Pass (TAP)** valide 1-24h
3. Communiquer le TAP à l'utilisateur par canal sécurisé (téléphone pro, physiquement)
4. Utilisateur se reconnecte avec le TAP et réenregistre une nouvelle méthode MFA
5. Vérifier les logs sign-in pour détecter tout usage frauduleux

### 🎫 Scénario 2 : « Je pars en voyage à l'étranger »

**Diagnostic :**

- Existe-t-il une CA « Block untrusted locations » ?
- Le pays de destination est-il dans la liste des trusted locations ?

**Action :**

1. Ajouter une exception temporaire ciblée à la CA (via groupe temporaire)
2. OU utiliser une CA « Require phishing-resistant MFA if outside trusted locations » (plus élégant)
3. Documenter dans le ticket la date de fin
4. Prévoir la révocation automatique via **Access Reviews** ou script

### 🎫 Scénario 3 : « Mon MFA demande un numéro que je ne vois pas »

**Diagnostic :** confusion avec le **number matching**.

**Action :**

1. Expliquer : l'écran de sign-in affiche un nombre (ex : `27`), il faut le taper dans l'app Authenticator
2. Si l'utilisateur ne voit pas le nombre → vérifier que l'écran n'est pas caché derrière une autre fenêtre
3. Si app Authenticator ne répond pas → vérifier connexion internet du téléphone + version app à jour
4. Fallback : générer un TAP

### 🎫 Scénario 4 : « Je suis bloqué par une policy CA, je ne comprends pas »

**Diagnostic :** utilisation du **Sign-in Log** dans le portail Entra.

1. **Entra Admin Center → Monitoring → Sign-in logs**
2. Filtrer sur l'UPN utilisateur
3. Ouvrir le sign-in en échec → onglet **Conditional Access**
4. Identifier la policy qui bloque (`Failure` en rouge)
5. Onglet **Report-only** : voir si de nouvelles policies bloqueraient prochainement

**Action :**

- Si policy légitime → guider l'utilisateur pour se conformer (installer app, changer localisation, activer MFA...)
- Si policy erronée → escalader vers l'équipe sécurité pour ajustement
- Utiliser **CA What-If tool** pour tester avant modification

## 12. Questions d'entretien — MFA & Conditional Access

### Q1 : Quelle est la différence entre MFA et 2FA ?

> Techniquement, 2FA (two-factor authentication) est un sous-ensemble de MFA (multi-factor authentication) où on exige exactement 2 facteurs. MFA peut demander 2 facteurs ou plus. Dans le langage courant, les deux sont souvent employés de manière interchangeable pour désigner une authentification à plusieurs facteurs.
> 

### Q2 : Qu'est-ce que le number matching et pourquoi c'est important ?

> Le number matching, imposé par Microsoft depuis mai 2023, exige que l'utilisateur saisisse un nombre affiché à l'écran de sign-in dans son app Microsoft Authenticator. C'est la contre-mesure aux **MFA fatigue attacks**, où un attaquant spammait des push notifications espérant que l'utilisateur finisse par approuver par réflexe. Avec number matching, l'approbation devient consciente.
> 

### Q3 : Comment fonctionne une policy Conditional Access ?

> Une policy CA suit la logique « Si → Alors ». Les **Assignments** définissent QUI (utilisateurs, groupes, rôles) accède à QUOI (cloud apps) et sous quelles **Conditions** (localisation, plateforme, risque, état du device). Les **Access Controls** définissent ensuite l'action : Grant (autoriser avec exigences comme MFA ou device compliant), Block, ou Session (limitations). Toutes les policies applicables sont combinées en ET logique.
> 

### Q4 : Qu'est-ce qu'un compte break-glass et pourquoi c'est essentiel ?

> Un compte break-glass est un compte Global Administrator d'urgence, exclu de toutes les policies MFA et Conditional Access, avec un mot de passe long stocké physiquement en coffre. Chaque tenant doit en avoir **2**. Ils permettent de récupérer l'accès si le service MFA Microsoft est indisponible, si une policy CA verrouille tous les admins, ou en cas de panne du fournisseur d'identité fédéré. C'est une règle **non négociable** de sécurité IAM.
> 

### Q5 : Différence entre « Require MFA » simple et Authentication Strengths ?

> `Require MFA` accepte n'importe quelle combinaison de 2 facteurs, y compris le SMS qui est phishable. **Authentication Strengths** permet d'exiger des méthodes spécifiques : par exemple, `Phishing-resistant MFA` limite à passkeys, FIDO2 keys, Windows Hello ou CBA. C'est beaucoup plus précis et sécurisé pour protéger les ressources sensibles ou les rôles admin.
> 

### Q6 : Comment débugger un utilisateur bloqué par une policy CA ?

> Direction **Entra Admin Center → Monitoring → Sign-in logs**. On filtre sur l'UPN, on ouvre le sign-in en échec, et on va dans l'onglet **Conditional Access** qui liste toutes les policies évaluées avec leur statut (Success / Failure / Not applied). La policy en `Failure` (rouge) est celle qui bloque. On peut aussi utiliser le **CA What-If tool** pour simuler l'impact d'une modification avant de la déployer.
> 

### Q7 : Qu'est-ce qu'une passkey et pourquoi Microsoft les pousse ?

> Une passkey est une paire de clés cryptographiques (publique/privée) liée à un service, où la clé privée ne quitte jamais l'appareil. L'authentification se fait par signature cryptographique + biométrie locale (Face ID, empreinte). Microsoft les pousse car elles sont **phishing-resistant par nature** : aucun secret n'est jamais tapé ou envoyé, donc impossible à intercepter ou à réutiliser sur un site malveillant. C'est l'avenir du passwordless.
> 

## 13. Best practices sécurité — checklist L2

<aside>
✅

**Baseline minimale d'une tenant Entra ID sécurisée (2026)**

</aside>

- [ ]  2 comptes break-glass configurés et monitorés
- [ ]  MFA activée pour **100 %** des utilisateurs
- [ ]  Legacy authentication (POP, IMAP, SMTP basic) **bloquée** via CA
- [ ]  Authentication method policy configurée : Authenticator + FIDO2 activés, SMS en fallback
- [ ]  Number matching activé (par défaut depuis 2023)
- [ ]  Policy CA « Require MFA for admins » avec `Phishing-resistant MFA` strength
- [ ]  Policy CA « Block untrusted locations » pour pays hors zone d'opération
- [ ]  Risk-based policies actives (sign-in risk medium+ → MFA)
- [ ]  Sign-in logs exportés vers SIEM (Sentinel, Splunk...)
- [ ]  Access Reviews trimestriels pour comptes admin
- [ ]  Temporary Access Pass activé pour onboarding / recovery

## 14. Sources officielles Microsoft

- [Overview of Conditional Access](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview) — Entra Docs
- [Conditional Access Authentication Strengths](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths) — Entra Docs
- [Plan for mandatory MFA](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-mandatory-multifactor-authentication) — Entra Docs
- [FIDO2 passkeys in Entra ID](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passwordless) — Entra Docs
- [Authentication methods policy](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods-manage) — Entra Docs
- [Break-glass accounts best practices](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/security-emergency-access) — Entra Docs
- [Microsoft Entra whats-new](https://learn.microsoft.com/en-us/entra/fundamentals/whats-new) — mises à jour mensuelles

---

<aside>
🏆

**Vous avez terminé la Fiche 4.** Combinée aux Fiches 1 (Entra ID lifecycle), 2 (Active Directory) et 3 (PowerShell IAM), vous couvrez maintenant **l'intégralité du périmètre IAM d'un Support L2 Microsoft** : identités, authentification, autorisation, automation. C'est un socle de connaissance solide pour vos entretiens. 🎯

</aside>