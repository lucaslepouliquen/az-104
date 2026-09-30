# 🗺️ Guide de Navigation AZ-104

## Accès Rapide par Section

### 📁 [Section 1: Identities & Governance](./section1/) (20-25%)

| Fichier | Sujet | Points Clés |
|---------|-------|-------------|
| [1.1](./section1/1.1_azure_active_directory.md) | Azure AD | Tenants, Users, Groups, Conditional Access |
| [1.2](./section1/1.2_managed_identities.md) | Managed Identities | System-Assigned vs User-Assigned |
| [1.3](./section1/1.3_rbac.md) | RBAC | Roles, Scopes, Assignments |
| [1.4](./section1/1.4_azure_policy.md) | Azure Policy | Effects (Deny, Audit, DeployIfNotExists) |
| [1.5](./section1/1.5_management_groups.md) | Management Groups | Hierarchy, Resource Locks |
| [1.6](./section1/1.6_azure_blueprints.md) | Gouvernance historique | Blueprints et alternatives actuelles |

### 💾 [Section 2: Storage](./section2/) (15-20%)

| Fichier | Sujet | Points Clés |
|---------|-------|-------------|
| [2.1](./section2/2.1_storage_accounts.md) | Storage Accounts | Types, Replication (LRS, GRS, RA-GRS) |
| [2.2](./section2/2.2_blob_storage.md) | Blob Storage | Tiers (Hot/Cool/Archive), Lifecycle |
| [2.3](./section2/2.3_azure_files.md) | Azure Files | **Kerberos (IAM)**, File Sync |
| [2.4](./section2/2.4_data_lake_gen2.md) | Data Lake Gen2 | Hierarchical Namespace, ACLs |
| [2.5](./section2/2.5_data_transfer.md) | Data Transfer | Import/Export, AzCopy |
| [2.6](./section2/2.6_sas.md) | SAS | User Delegation (IAM) vs Service/Account |

### 🖥️ [Section 3: Compute](./section3/) (20-25%)

| Fichier | Sujet | Points Clés |
|---------|-------|-------------|
| [3.1](./section3/3.1_virtual_machines.md) | Virtual Machines | Sizes, Disks, Availability Sets/Zones |
| [3.2](./section3/3.2_vmss.md) | VMSS | Auto-scaling, Update management |
| [3.3](./section3/3.3_app_services.md) | App Services | **Scale Up vs Scale Out**, Autoscaling |
| [3.4](./section3/3.4_aci.md) | ACI | Serverless, Pay-per-second, Container Groups |
| [3.5](./section3/3.5_iac.md) | IaC | **ARM, Bicep, Terraform** |
| [3.6](./section3/3.6_aks.md) | AKS | Control Plane, Node Pools, Autoscaling |

### 🌐 [Section 4: Networking](./section4/) (15-20%)

| Fichier | Sujet | Points Clés |
|---------|-------|-------------|
| [4.1](./section4/4.1_virtual_networks.md) | VNet | Subnets, Peering, Endpoints |
| [4.2](./section4/4.2_nsg.md) | NSG | Rules, Priorities, ASG |
| [4.3](./section4/4.3_load_balancing.md) | Load Balancing | LB, App Gateway, Front Door, Traffic Mgr |
| [4.4](./section4/4.4_network_watcher.md) | Network Watcher | Diagnostic tools (IP Flow, Next Hop) |
| [4.5](./section4/4.5_vpn_expressroute.md) | VPN & ExpressRoute | Site-to-Site, Point-to-Site |
| [4.6](./section4/4.6_azure_bastion.md) | Azure Bastion | Secure RDP/SSH access |

### 📊 [Section 5: Monitoring & Backup](./section5/) (10-15%)

| Fichier | Sujet | Points Clés |
|---------|-------|-------------|
| [5.1](./section5/5.1_azure_monitor.md) | Azure Monitor | Metrics, Logs, KQL, Alerts |
| [5.2](./section5/5.2_azure_backup.md) | Azure Backup | Recovery Services Vault, Policies |
| [5.3](./section5/5.3_azure_site_recovery.md) | Site Recovery | DR, Failover, Replication |

---

## 🎯 Parcours d'Apprentissage Recommandés

### 🏃 Parcours Rapide (Révision - 3-4 heures)
Focus sur les concepts clés marqués **en gras** dans chaque section :
1. Section 2.3 - **Kerberos authentication** (IAM)
2. Section 3.3 - **Scale Up vs Scale Out**
3. Section 3.5 - **ARM/Bicep/Terraform**
4. Section 4.3 - **Load Balancing** (comparaison)
5. Section 1.2 - **Managed Identities**

### 🚶 Parcours Complet (Apprentissage - 15-20 heures)
Ordre recommandé pour comprendre les dépendances :
1. **Section 1** - Identities & Governance (base IAM)
2. **Section 4** - Networking (infrastructure réseau)
3. **Section 2** - Storage (stockage de données)
4. **Section 3** - Compute (calcul et applications)
5. **Section 5** - Monitoring & Backup (supervision)

### 🎓 Parcours Examen (Révision intensive - 8-10 heures)
Par ordre de poids dans l'examen :
1. **Section 4** (15-20%) - 3 heures
2. **Section 3** (20-25%) - 2.5 heures
3. **Section 1** (20-25%) - 2 heures
4. **Section 2** (15-20%) - 2 heures
5. **Section 5** (10-15%) - 1.5 heure

---

## 🔍 Recherche par Concept Clé

### Identity-Based Access (IAM)
- [2.3 Azure Files - Kerberos](./section2/2.3_azure_files.md) ⭐ **Point d'examen critique**
- [2.6 SAS - User Delegation SAS](./section2/2.6_sas.md)
- [1.2 Managed Identities](./section1/1.2_managed_identities.md)

### Scaling
- [3.3 App Services - Autoscaling](./section3/3.3_app_services.md)
- [3.2 VMSS](./section3/3.2_vmss.md)
- [3.6 AKS - Cluster Autoscaler & HPA](./section3/3.6_aks.md)

### High Availability
- [3.1 VMs - Availability Sets/Zones](./section3/3.1_virtual_machines.md)
- [4.3 Load Balancing](./section4/4.3_load_balancing.md)
- [5.3 Site Recovery](./section5/5.3_azure_site_recovery.md)

### Networking
- [4.1 VNet - Peering & Endpoints](./section4/4.1_virtual_networks.md)
- [4.2 NSG](./section4/4.2_nsg.md)
- [4.4 Network Watcher](./section4/4.4_network_watcher.md)

### Infrastructure as Code
- [3.5 IaC - ARM, Bicep, Terraform](./section3/3.5_iac.md)
- [1.6 Blueprints](./section1/1.6_azure_blueprints.md)
- [1.4 Azure Policy](./section1/1.4_azure_policy.md)

### Security & Governance
- [1.1 Azure AD](./section1/1.1_azure_active_directory.md)
- [1.3 RBAC](./section1/1.3_rbac.md)
- [1.5 Management Groups - Resource Locks](./section1/1.5_management_groups.md)

---

## 📌 Accès Direct aux Scénarios d'Examen

Chaque fichier contient des **scénarios d'examen** marqués avec **⚠️**. Points critiques :

### Questions Pièges Fréquentes
1. **"Which storage data service can be configured to use identity-based access?"**
   - 👉 [2.3 Azure Files](./section2/2.3_azure_files.md) - **Réponse : file shares (Kerberos)**

2. **"Minimize administrative effort to provide external access to a VM"**
   - 👉 [3.1 VMs](./section3/3.1_virtual_machines.md) - **Réponse : Add Public IP**

3. **"Scale Up vs Scale Out for App Service"**
   - 👉 [3.3 App Services](./section3/3.3_app_services.md) - **Concepts détaillés**

4. **"ACI vs AKS for simple 2-container app"**
   - 👉 [3.4 ACI](./section3/3.4_aci.md) & [3.6 AKS](./section3/3.6_aks.md) - **Comparaison**

5. **"Kerberos = IAM ou Credential?"**
   - 👉 [2.3 Azure Files](./section2/2.3_azure_files.md) - **Réponse : IAM**

---

## 💡 Conseils de Navigation

- **📝 README.md** : Vue d'ensemble de chaque section
- **⚠️** : Points critiques pour l'examen
- **✅/❌** : À faire / À éviter
- **Tableaux comparatifs** : Pour décisions rapides
- **Commandes CLI/PowerShell** : Pour pratique

---

**Bon apprentissage et bonne chance pour l'examen AZ-104 !** 🚀

