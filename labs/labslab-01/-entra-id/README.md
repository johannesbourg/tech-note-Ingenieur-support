# Lab 1 — Entra ID depuis zéro (Cloud-first)

<aside>
🎯

**Objectif du lab :** Maîtriser Microsoft Entra ID (ex-Azure AD) de A à Z **sans installer quoi que ce soit sur mon PC**. Ce lab me prépare directement aux offres **Spécialiste Support IT L2 (Réseau & IAM)**, **Expert Microsoft 365 / Entra ID** et **Ingénieur Système AD / Entra ID**.

</aside>

**Durée estimée :** 4 à 6 heures (étalables sur plusieurs sessions)

**Coût :** 0 €

**Prérequis matériel :** juste un navigateur web (Chrome, Edge, Firefox)

---

## 📋 Sommaire

1. Contexte & objectifs
2. Environnement technique
3. Étape 0 — Création du tenant Microsoft 365 Developer
4. Étape 1 — Découverte du portail Entra ID
5. Étape 2 — Structure organisationnelle
6. Étape 3 — Cycle de vie des utilisateurs
7. Étape 4 — Groupes de sécurité et licences
8. Étape 5 — MFA (Multi-Factor Authentication)
9. Étape 6 — Accès conditionnel
10. Étape 7 — SSPR (Self-Service Password Reset)
11. Étape 8 — Journalisation & audit
12. Livrables du lab
13. Ce que ce lab prouve aux recruteurs
14. Prochaines étapes

---

## 1. 🎯 Contexte & objectifs

Microsoft Entra ID (anciennement Azure AD) est **le service d'identité cloud le plus utilisé en entreprise**. Toutes les offres cibles le mentionnent explicitement. Ce lab me permet de :

- Créer et gérer un tenant Entra ID de bout en bout
- Comprendre la structure des identités cloud (utilisateurs, groupes, licences)
- Configurer les fonctionnalités de sécurité avancées (MFA, accès conditionnel, SSPR)
- Produire une **documentation professionnelle** utilisable directement en entretien

---

## 2. 🛠️ Environnement technique

- **Tenant :** Microsoft 365 Developer (gratuit à vie, renouvelé automatiquement)
- **Licences :** 25 licences Microsoft 365 E5 (valeur ~ 1500 €/mois offertes)
- **Portails utilisés :**
    - Microsoft Entra Admin Center — `entra.microsoft.com`
    - Microsoft 365 Admin Center — `admin.microsoft.com`
    - Portail Azure — `portal.azure.com`

---

## 3. 🚀 Étape 0 — Création du tenant Microsoft 365 Developer

<aside>
⏱️

Durée : 15 minutes

</aside>

### Marche à suivre

1. Ouvrir `https://developer.microsoft.com/microsoft-365/dev-program`
2. Cliquer sur **"Join Now"** (ou "Rejoindre maintenant")
3. Se connecter avec un compte Microsoft personnel (ou en créer un nouveau avec une adresse type `daniel.assou.dev@outlook.com`)
4. Remplir le formulaire :
    - Pays : Côte d'Ivoire
    - Langue : Français
    - Company : *"Personal Learning Lab"*
    - Focus areas : cocher **Microsoft Entra ID**, **Microsoft Graph**, **Microsoft 365 Apps**
5. Choisir **"Instant sandbox"** (recommandé — plus rapide que "Configurable sandbox")
6. Définir un mot de passe fort pour l'administrateur global (à conserver dans un gestionnaire de mots de passe)
7. Noter le **domaine du tenant** : ce sera quelque chose comme `xxxxxxxx.onmicrosoft.com`

### ✅ Résultat attendu

- Un tenant Microsoft 365 actif
- Un compte administrateur global : `admin@xxxxxxxx.onmicrosoft.com`
- 25 utilisateurs de démonstration précréés avec licences E5

### 📸 Captures à prendre

- [ ]  Confirmation d'inscription au Developer Program
- [ ]  Dashboard du portail admin M365 à la première connexion
- [ ]  Liste des utilisateurs de démonstration

---

## 4. 🔍 Étape 1 — Découverte du portail Entra ID

<aside>
⏱️

Durée : 30 minutes

</aside>

### Marche à suivre

1. Se connecter à `https://entra.microsoft.com` avec le compte admin
2. Explorer les sections principales dans le menu de gauche :
    - **Users** (Utilisateurs)
    - **Groups** (Groupes)
    - **Roles & admins** (Rôles et administrateurs)
    - **Protection** (MFA, accès conditionnel, Identity Protection)
    - **Monitoring & health** (Journaux, rapports)
    - **Enterprise apps** (Applications d'entreprise)
3. Aller dans **Overview** et noter :
    - Le **Tenant ID** (GUID)
    - Le **Primary domain** (`xxx.onmicrosoft.com`)
    - L'édition Entra ID (Free / P1 / P2)

### 📸 Captures à prendre

- [ ]  Page d'accueil Entra Admin Center
- [ ]  Overview du tenant avec Tenant ID visible
- [ ]  Menu de navigation déplié

---

## 5. 🏢 Étape 2 — Structure organisationnelle

<aside>
⏱️

Durée : 20 minutes

</aside>

### Scénario

Je simule une entreprise fictive **"Global Trade Corp"** avec 4 départements : IT, Finance, RH, Ventes.

### Marche à suivre

1. Aller dans **Users → All users**
2. Compléter les profils des 25 utilisateurs de démo avec des attributs réalistes :
    - **Department** : IT, Finance, HR, Sales
    - **Job title** : ex. "Support Engineer", "CFO", "HR Manager"
    - **City** : Paris, Abidjan, Casablanca, Dakar
    - **Manager** : désigner un manager pour chaque utilisateur
3. Créer 3 utilisateurs supplémentaires manuellement (voir Étape 3)

### 📸 Captures à prendre

- [ ]  Profil utilisateur enrichi avec tous les attributs
- [ ]  Liste des utilisateurs filtrée par département

---

## 6. 🔄 Étape 3 — Cycle de vie des utilisateurs

<aside>
⏱️

Durée : 45 minutes · **Étape critique** pour les entretiens

</aside>

### Scénarios à jouer

**Scénario A — Onboarding**

1. Créer un nouvel utilisateur : `nouveau.collab@xxx.onmicrosoft.com`
2. Remplir tous les attributs (nom, prénom, département, manager, téléphone)
3. Assigner une licence Microsoft 365 E5
4. Ajouter aux groupes appropriés (voir Étape 4)
5. Se connecter avec ce nouveau compte dans un onglet privé pour vérifier

**Scénario B — Mobilité interne**

1. Prendre un utilisateur existant
2. Changer son département (ex. Finance → IT)
3. Modifier son manager
4. Révoquer les anciens accès (retirer des groupes Finance)
5. Attribuer les nouveaux accès (ajouter aux groupes IT)

**Scénario C — Départ (offboarding)**

1. Choisir un utilisateur
2. **Bloquer la connexion** (Block sign-in)
3. **Révoquer toutes les sessions** (Revoke sessions)
4. **Réinitialiser le mot de passe** avec un mot de passe aléatoire
5. **Retirer toutes les licences**
6. **Convertir la boîte mail en boite partagée** (via M365 Admin Center)
7. Après 30 jours simulés : suppression définitive

### 📸 Captures à prendre

- [ ]  Écran de création d'utilisateur avec tous les champs remplis
- [ ]  Utilisateur mobilité (avant/après les changements de groupes)
- [ ]  Utilisateur bloqué (sign-in blocked) avec licences retirées

### 📝 Livrable à produire

Créer un document Word **"Procédure de gestion du cycle de vie des utilisateurs"** avec les 3 scénarios documentés pas à pas. Ce document sera **directement citable en entretien** comme livrable professionnel.

---

## 7. 👥 Étape 4 — Groupes de sécurité et licences

<aside>
⏱️

Durée : 30 minutes

</aside>

### Marche à suivre

1. Aller dans **Groups → All groups → New group**
2. Créer les groupes suivants avec **"Membership type: Dynamic User"** :
    - `SG-Department-IT` : règle `user.department -eq "IT"`
    - `SG-Department-Finance` : règle `user.department -eq "Finance"`
    - `SG-Department-HR` : règle `user.department -eq "HR"`
    - `SG-Department-Sales` : règle `user.department -eq "Sales"`
3. Créer des groupes "Assigned" pour les accès manuels :
    - `SG-VPN-Users`
    - `SG-Admins-M365`
    - `SG-External-Guests`
4. Attribuer une licence E5 au groupe `SG-Department-IT` → tous les utilisateurs IT reçoivent la licence automatiquement (**Group-Based Licensing**)
5. Vérifier que les utilisateurs IT ont bien la licence attribuée via le groupe

### 📸 Captures à prendre

- [ ]  Création d'un groupe dynamique avec la règle visible
- [ ]  Onglet **Members** montrant les utilisateurs automatiquement inclus
- [ ]  Attribution de licence via groupe

---

## 8. 🔐 Étape 5 — MFA (Multi-Factor Authentication)

<aside>
⏱️

Durée : 30 minutes

</aside>

### Marche à suivre

1. Aller dans **Protection → Authentication methods**
2. Configurer les méthodes activées :
    - **Microsoft Authenticator** → All users
    - **SMS** → All users
    - **Email OTP** → pour les invités uniquement
    - **FIDO2 security keys** → pour les admins uniquement
3. Se connecter avec un compte test dans un onglet privé
4. Suivre le prompt d'enrôlement MFA (télécharger l'app Microsoft Authenticator sur le téléphone)
5. Documenter chaque étape de l'expérience utilisateur avec captures

### 📸 Captures à prendre

- [ ]  Page de configuration des Authentication methods
- [ ]  Prompt de premier login MFA (QR code)
- [ ]  Confirmation d'enrôlement réussi

---

## 9. 🎛️ Étape 6 — Accès conditionnel

<aside>
⏱️

Durée : 45 minutes · **Étape phare pour se démarquer**

</aside>

### Marche à suivre

1. Aller dans **Protection → Conditional Access → Policies**
2. Créer 3 politiques classiques :

**Politique 1 — MFA obligatoire pour les admins**

- Users : `Directory roles → Global Administrator`, `Security Administrator`
- Cloud apps : All cloud apps
- Conditions : n/a
- Grant : Require MFA
- State : **Report-only** d'abord, puis On

**Politique 2 — Bloquer les géolocalisations à risque**

- Users : All users (exclure les admins pour éviter le lockout !)
- Cloud apps : All cloud apps
- Conditions : Locations → Any location, exclude Trusted
- Grant : Block access
- State : **Report-only**

**Politique 3 — Forcer un appareil conforme pour SharePoint**

- Users : `SG-Department-IT`
- Cloud apps : Office 365 SharePoint Online
- Grant : Require compliant device (mode découverte)
- State : **Report-only**

### ⚠️ Règle d'or

**Toujours exclure un compte admin de secours (Break Glass) de toutes les politiques d'accès conditionnel.** Documentez ce compte, c'est une bonne pratique attendue en entretien.

### 📸 Captures à prendre

- [ ]  Liste des 3 politiques créées
- [ ]  Détail d'une politique (users, apps, conditions, grant)
- [ ]  Onglet **Sign-in logs** montrant l'évaluation des politiques

---

## 10. 🔑 Étape 7 — SSPR (Self-Service Password Reset)

<aside>
⏱️

Durée : 20 minutes

</aside>

### Marche à suivre

1. Aller dans **Protection → Password reset**
2. Activer SSPR pour **All users**
3. Configurer les méthodes d'authentification requises (2 sur 3)
4. Se déconnecter, aller sur `https://passwordreset.microsoftonline.com`
5. Effectuer un vrai reset de mot de passe avec un compte test
6. Vérifier dans les Audit logs que l'événement est bien tracé

### 📸 Captures à prendre

- [ ]  Configuration SSPR activée
- [ ]  Page publique de reset SSPR
- [ ]  Audit log de l'événement de reset

---

## 11. 📊 Étape 8 — Journalisation & audit

<aside>
⏱️

Durée : 30 minutes

</aside>

### Marche à suivre

1. Aller dans **Monitoring & health**
2. Explorer :
    - **Sign-in logs** : filtrer par utilisateur, par échec, par politique CA appliquée
    - **Audit logs** : filtrer par catégorie (User Management, Group Management)
    - **Usage & insights** : quels apps sont utilisées, quelles méthodes d'auth
3. Exporter un rapport CSV des sign-ins des 7 derniers jours
4. Identifier une "connexion suspecte" simulée (échec MFA répété)

### 📸 Captures à prendre

- [ ]  Sign-in logs avec filtres
- [ ]  Un log détaillé montrant les politiques CA évaluées
- [ ]  Audit log d'une modification de groupe

---

## 12. 📦 Livrables du lab

À la fin du lab, je dois avoir produit :

- [ ]  **Document PDF "Prise en main d'Entra ID"** — 6 à 10 pages avec captures annotées
- [ ]  **Procédure onboarding/mobilité/offboarding** — 3 pages réutilisables en entreprise
- [ ]  **Schéma d'architecture** : tenant Entra ID + groupes dynamiques + politiques CA ([Draw.io](http://Draw.io) gratuit)
- [ ]  **Article LinkedIn** : *"J'ai monté un environnement Entra ID complet en un week-end — voici ce que j'ai appris"* (500-800 mots)
- [ ]  **Page portfolio** : synthèse avec captures et lien vers l'article

---

## 13. 🎯 Ce que ce lab prouve aux recruteurs

<aside>
💬

Pour l'offre **Spécialiste Support IT L2 (Réseau & IAM)** :

> *"J'ai construit et documenté un environnement Entra ID complet couvrant la gestion des comptes, MFA et accès conditionnel. Je suis opérationnel immédiatement sur les demandes IAM du poste."*
> 
</aside>

<aside>
💬

Pour l'offre **Expert Microsoft 365 / Entra ID** :

> *"J'ai réalisé le déploiement d'un tenant M365 avec licences dynamiques par groupes, MFA obligatoire pour les admins, et 3 politiques d'accès conditionnel en mode report-only. Je maîtrise les bonnes pratiques comme le compte Break Glass."*
> 
</aside>

<aside>
💬

Pour l'offre **Ingénieur Système AD / Entra ID (Harmonie Mutuelle)** :

> *"J'ai constitué un environnement de test cloud reproductible que j'utilise pour valider mes procédures avant application en production. Cela couvre l'aspect exploitation et documentation attendu du poste."*
> 
</aside>

---

## 14. ➡️ Prochaines étapes

Une fois ce lab terminé :

- ✅ Mettre à jour la section "Projets" du portfolio hub
- ✅ Publier l'article LinkedIn
- ✅ Ajouter "Microsoft Entra ID (Free tier)" et "MFA / Conditional Access" à la section Compétences du CV
- ➡️ Enchainer sur **Lab 2 — Hybrid Identity** (AD Connect via Azure Free Trial, sans surcharger le PC)
- ➡️ En parallèle : commencer les modules SC-300 sur Microsoft Learn

---

## 📝 Journal de bord

<aside>
📖

Je note ici mes observations, difficultés, apprentissages au fur et à mesure. Ce journal servira à rédiger l'article LinkedIn final.

</aside>

**Session 1 — *(date)***

- *(à remplir : ce que j'ai fait, difficultés, temps passé)*

**Session 2 — *(date)***

- *(à remplir)*