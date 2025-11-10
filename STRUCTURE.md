# Structure du Guide AZ-104

Ce guide est organisé en **5 sections principales**, chacune découpée en **sous-fichiers** pour faciliter la navigation et l'apprentissage.

## 📁 Structure des Dossiers

```
az-104/
├── section1/          # Identities & Governance (15-20%)
│   ├── README.md
│   ├── 1.1_azure_active_directory.md
│   ├── 1.2_managed_identities.md
│   ├── 1.3_rbac.md
│   ├── 1.4_azure_policy.md
│   ├── 1.5_management_groups.md
│   └── 1.6_azure_blueprints.md
│
├── section2/          # Storage (15-20%)
│   ├── README.md
│   ├── 2.1_storage_accounts.md
│   ├── 2.2_blob_storage.md
│   ├── 2.3_azure_files.md
│   ├── 2.4_data_lake_gen2.md
│   ├── 2.5_data_transfer.md
│   └── 2.6_sas.md
│
├── section3/          # Compute (20-25%)
│   ├── README.md
│   ├── 3.1_virtual_machines.md
│   ├── 3.2_vmss.md
│   ├── 3.3_app_services.md
│   ├── 3.4_aci.md
│   ├── 3.5_iac.md
│   └── 3.6_aks.md
│
├── section4/          # Networking (25-30%)
│   ├── README.md
│   ├── 4.1_virtual_networks.md
│   ├── 4.2_nsg.md
│   ├── 4.3_load_balancing.md
│   ├── 4.4_network_watcher.md
│   ├── 4.5_vpn_expressroute.md
│   └── 4.6_azure_bastion.md
│
└── section5/          # Monitoring & Backup (10-15%)
    ├── README.md
    ├── 5.1_azure_monitor.md
    ├── 5.2_azure_backup.md
    └── 5.3_azure_site_recovery.md
```

## 📊 Statistiques

### Total
- **5 sections principales**
- **27 sous-fichiers** (fichiers markdown de contenu)
- **5 fichiers README** (un par section)
- **~12,000 lignes de contenu**

### Par Section

| Section | Fichiers | Lignes | Poids examen |
|---------|----------|--------|--------------|
| Section 1: Identities & Governance | 6 | ~1,700 | 15-20% |
| Section 2: Storage | 6 | ~4,100 | 15-20% |
| Section 3: Compute | 6 | ~2,700 | 20-25% |
| Section 4: Networking | 6 | ~2,500 | 25-30% |
| Section 5: Monitoring & Backup | 3 | ~2,100 | 10-15% |

## 🎯 Navigation Recommandée

### Pour un apprentissage complet (ordre recommandé)
1. **Section 1** - Identities & Governance (comprendre qui peut faire quoi)
2. **Section 4** - Networking (base pour tout le reste)
3. **Section 2** - Storage (stockage des données)
4. **Section 3** - Compute (calcul et applications)
5. **Section 5** - Monitoring & Backup (supervision et protection)

### Pour une révision rapide (par poids d'examen)
1. **Section 4** - Networking (25-30%) ⭐⭐⭐
2. **Section 3** - Compute (20-25%) ⭐⭐⭐
3. **Section 1** - Identities & Governance (15-20%) ⭐⭐
4. **Section 2** - Storage (15-20%) ⭐⭐
5. **Section 5** - Monitoring & Backup (10-15%) ⭐

## 📝 Points Clés par Section

### Section 1: Identities & Governance
- **Kerberos** = Identity-based access pour **Azure Files uniquement**
- **Managed Identities** (System vs User)
- **RBAC** (roles, scopes)
- **Azure Policy** (Deny, Audit, DeployIfNotExists)
- **Blueprints** vs ARM vs Policy

### Section 2: Storage
- **Storage Account types** (GPv2, Premium, etc.)
- **Blob tiers** (Hot, Cool, Archive)
- **Azure Files** avec **Kerberos authentication** (IAM)
- **SAS types** (User Delegation = IAM, Service/Account = Credential)
- **Différence IAM vs Credential**

### Section 3: Compute
- **VMs** (sizes, disks, availability)
- **App Service Plans** (Scale Up vs Scale Out)
- **ACI** (serverless, pay-per-second)
- **AKS** (Control Plane managed, Node Pools)
- **IaC** (ARM, Bicep, Terraform)

### Section 4: Networking
- **VNet** (subnets, peering, endpoints)
- **NSG** (rules, priorities, ASG)
- **Load Balancing** (Load Balancer, App Gateway, Front Door, Traffic Manager)
- **Network Watcher** (diagnostic tools)
- **VPN** vs **ExpressRoute**

### Section 5: Monitoring & Backup
- **Azure Monitor** (metrics, logs, KQL, alerts)
- **Azure Backup** (Recovery Services Vault, policies)
- **Site Recovery** (disaster recovery, failover)

## 🔍 Comment Utiliser cette Structure

### 1. Apprentissage Initial
- Commencez par le **README.md** de chaque section
- Lisez les fichiers dans l'ordre numérique
- Prenez des notes sur les **⚠️ Concepts Clés pour AZ-104**

### 2. Révision Ciblée
- Utilisez les **README.md** pour identifier les sujets à revoir
- Ouvrez directement le fichier concerné
- Concentrez-vous sur les tableaux comparatifs et scénarios d'examen

### 3. Préparation Examen
- Relisez tous les encadrés **⚠️ Concept Clé pour AZ-104**
- Pratiquez les commandes CLI/PowerShell
- Faites les scénarios d'examen de chaque fichier

## 📌 Fichiers Originaux

Les fichiers originaux consolidés sont toujours disponibles :
- `section1_identities_governance.md`
- `section2_storage.md`
- `section3_compute.md`
- `section4_networking.md`
- `section5_monitoring_backup.md`

## 🚀 Prochaines Étapes

1. **Lire le README** de la section qui vous intéresse
2. **Explorer les sous-fichiers** dans l'ordre
3. **Pratiquer** avec Azure Portal, CLI, et PowerShell
4. **Réviser** les scénarios d'examen

---

**Bonne chance pour l'examen AZ-104 !** 🎓

