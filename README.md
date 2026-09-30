# Guide Complet AZ-104 : Microsoft Azure Administrator

## 📋 Vue d'ensemble de la certification

L'examen AZ-104 valide vos compétences en tant qu'administrateur Azure. Il couvre la gestion des abonnements, la sécurité, le stockage, la mise en réseau et les ressources de calcul.

**Durée :** 150 minutes  
**Questions :** 40-60 questions  
**Score de réussite :** 700/1000  
**Coût :** $165 USD

---

## 🗂️ Navigation du Guide

Ce guide est organisé en **5 sections principales**, chacune découpée en **sous-fichiers** pour faciliter la navigation :

- **📁 [STRUCTURE.md](STRUCTURE.md)** - Vue d'ensemble de la structure des dossiers
- **🗺️ [NAVIGATION.md](NAVIGATION.md)** - Guide de navigation avec parcours d'apprentissage recommandés
- **📖 Fichiers originaux** - Les 5 fichiers de sections consolidés sont toujours disponibles

### Accès Rapide par Dossier

| Section | Dossier | Fichiers | Poids Examen |
|---------|---------|----------|--------------|
| [Identities & Governance](./section1/) | `section1/` | 6 fichiers | 20-25% |
| [Storage](./section2/) | `section2/` | 6 fichiers | 15-20% |
| [Compute](./section3/) | `section3/` | 6 fichiers | 20-25% |
| [Networking](./section4/) | `section4/` | 6 fichiers | 15-20% |
| [Monitoring & Backup](./section5/) | `section5/` | 3 fichiers | 10-15% |

Chaque dossier contient un **README.md** avec la liste des sous-fichiers et leur contenu.

---

## 🎯 Domaines d'examen (Répartition des poids)

### 1. [Gérer les identités et la gouvernance Azure (20-25%)](section1_identities_governance.md)
- [Azure Active Directory (Azure AD / Entra ID)](section1_identities_governance.md#11-azure-active-directory-azure-ad)
  - [Concepts Fondamentaux](section1_identities_governance.md#concepts-fondamentaux)
  - [Utilisateurs et Groupes](section1_identities_governance.md#utilisateurs-et-groupes)
  - [Custom Domains](section1_identities_governance.md#custom-domains)
  - [Licensing et Dynamic Groups](section1_identities_governance.md#licensing-et-dynamic-groups)
  - [B2B Collaboration](section1_identities_governance.md#b2b-collaboration)
  - [Azure AD Connect](section1_identities_governance.md#azure-ad-connect---synchronisation-hybrid)
- [Contrôle d'accès basé sur les rôles (RBAC)](section1_identities_governance.md#12-role-based-access-control-rbac)
  - [Rôles Built-in Essentiels](section1_identities_governance.md#rôles-built-in-essentiels)
  - [Rôles Administratifs Azure AD](section1_identities_governance.md#rôles-administratifs-azure-ad)
  - [Scopes d'assignation RBAC](section1_identities_governance.md#scopes-dassignation-rbac---détaillé)
- [Azure Policy et gouvernance](section1_identities_governance.md#13-azure-policy)
  - [Concepts Clés](section1_identities_governance.md#concepts-clés)
  - [Effects Principaux](section1_identities_governance.md#effects-principaux---détaillé)
- [Management Groups et abonnements](section1_identities_governance.md#14-management-groups)
  - [Hiérarchie](section1_identities_governance.md#hiérarchie)
  - [Resource Locks](section1_identities_governance.md#resource-locks)

### 2. [Implémenter et gérer le stockage (15-20%)](section2_storage.md)
- Comptes de stockage et types
- **Types de blobs** : Block, Page, Append
- Azure Files et partages de fichiers
- Sécurité et réplication du stockage

### 3. [Déployer et gérer les ressources de calcul Azure (20-25%)](section3_compute.md)
- Machines virtuelles et haute disponibilité
- App Service et déploiement d'applications
- Container Instances
- ARM Templates et Infrastructure as Code

### 4. [Configurer et gérer la mise en réseau virtuelle (15-20%)](section4_networking.md)
- Réseaux virtuels (VNets) et sous-réseaux
- Groupes de sécurité réseau (NSG)
- Équilibrage de charge et passerelles d'application
- Connectivité hybride (VPN, ExpressRoute)

### 5. [Surveiller et sauvegarder les ressources Azure (10-15%)](section5_monitoring_backup.md)
- Azure Monitor et alertes
- Log Analytics et KQL
- Azure Backup et récupération

---

## 📚 Contenu du guide

Ce référentiel contient :

### 📖 [Guide théorique complet](az104_theory_guide.md)
Un guide détaillé de 1100+ lignes couvrant tous les aspects de l'examen AZ-104 avec :
- **Points critiques identifiés** pour l'examen
- **Erreurs fréquentes** et leurs corrections
- **Conseils pratiques** basés sur des questions réelles
- **Matrices de décision** pour choisir les bonnes solutions
- **Exemples concrets** et cas d'usage

### 🔑 Points forts du guide

#### Stockage Azure - Types de Blobs
Le guide inclut une section détaillée sur les **3 types de blobs Azure** :

**Block Blobs**
- Usage : Fichiers standard (documents, images, vidéos)
- Taille max : 190.7 TB
- Cas d'usage : Sites web statiques, médias, sauvegardes

**Page Blobs**
- Usage : Disques de machines virtuelles (VHD/VHDX)
- Taille max : 8 TB
- Cas d'usage : Disques OS/données, bases de données

**Append Blobs**
- Usage : Données ajoutées séquentiellement (logs)
- Taille max : 195 GB
- Cas d'usage : Logs d'applications, données IoT

#### Réseaux virtuels - Points critiques
- **VNet Peering** : Plages d'adresses non-chevauchantes obligatoires
- **NSG Priorities** : Plus bas = plus prioritaire (100 < 200)
- **Session Persistence** : Client IP + Protocol pour sticky sessions

#### Machines virtuelles - Gestion des disques
- **Disque C:** Persistant (OS, applications)
- **Disque D:** Temporaire (PERDU lors maintenance)
- **Disques de données** : Persistants (E:, F:, etc.)

---

## 🎯 Conseils d'examen

### Stratégie de révision
1. **Comprendre les concepts** plutôt que mémoriser
2. **Pratiquer** avec Azure Portal et CLI/PowerShell
3. **Étudier les scénarios** réels d'utilisation
4. **Connaître les limites** des services Azure
5. **Focus sécurité** et bonnes pratiques

### Pièges fréquents identifiés
- **VNet Peering** : Confusion entre plages chevauchantes/non-chevauchantes
- **NSG** : Oublier l'évaluation en cascade Subnet → NIC
- **Load Balancer** : Confusion entre session persistence et NAT rules
- **Diagnostic réseau** : Utiliser les mauvais outils
- **Storage Account** : Types et limitations des comptes

### Checklist final
- [ ] Dynamic group rules syntax
- [ ] Custom domain DNS records
- [ ] FileStorage pour Premium files uniquement
- [ ] NSG sharing entre ressources
- [ ] Log Analytics Workspace comme target pour VM alerts
- [ ] Recovery Services Vault : même région obligatoire

---

## 📁 Structure du référentiel

```
az-104/
├── README.md                 # Ce fichier - Vue d'ensemble
├── az104_theory_guide.md     # Guide théorique complet (1100+ lignes)
├── DEPLOYMENT.md             # Instructions de déploiement
├── index.md                  # Page d'accueil du site
├── _config.yml               # Configuration Jekyll
└── LICENSE                   # Licence du projet
```

---

## 🚀 Comment utiliser ce guide

1. **Commencez par** le [guide théorique](az104_theory_guide.md) pour une vue complète
2. **Concentrez-vous** sur les sections marquées 🎯 (points critiques)
3. **Pratiquez** les commandes et scénarios décrits
4. **Utilisez** les matrices de décision pour les questions d'examen
5. **Révisez** les erreurs fréquentes avant l'examen

---

## 📊 Temps de préparation recommandé

**Total : 40-60 heures d'étude**
- Théorie : 20-30 heures
- Labs pratiques : 15-20 heures
- Examens blancs : 5-10 heures

---

## 📚 Ressources complémentaires

### Documentation officielle
- [Azure Architecture Center](https://docs.microsoft.com/azure/architecture/)
- [Azure Well-Architected Framework](https://docs.microsoft.com/azure/architecture/framework/)
- [Microsoft Learn AZ-104](https://docs.microsoft.com/learn/certifications/exams/az-104)

### Labs pratiques
- Compte gratuit Azure (12 mois)
- Microsoft Learn modules interactifs
- Labs hands-on recommandés

### Examens blancs
- MeasureUp practice tests
- Whizlabs Azure exams
- Tutorials Dojo practice tests

---

## 🤝 Contribution

Ce guide est basé sur l'expérience réelle de l'examen et est continuellement mis à jour. Les contributions sont les bienvenues !

---

## 📄 Licence

Ce projet est sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.

---

**Bonne chance pour votre certification AZ-104 ! 🎉**