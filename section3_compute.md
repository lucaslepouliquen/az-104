## 3. Deploy and Manage Azure Compute Resources (20-25%)

### 3.1 Virtual Machines

#### VM Sizes et Families
- **General Purpose** (B, D, DC series) : Workloads équilibrés
- **Compute Optimized** (F series) : Applications CPU-intensive
- **Memory Optimized** (E, M, G series) : Bases de données, analytics
- **Storage Optimized** (L series) : Big data, SQL NoSQL
- **GPU** (N series) : IA, machine learning, rendering

#### Disques et Stockage

** Erreur fréquente identifiée :** Disques temporaires vs persistants

**Disque C: (OS Disk)**
- **Type** : Persistant (VHD dans Azure Storage)
- **Contenu** : OS, applications installées
- **Persistance** : Conservé lors redémarrages/arrêts
- **Usage** : Applications, données importantes

**Disque D: (Temporary Disk)**
- **Type** : Temporaire (SSD local hyperviseur)
- **Contenu** : Pagefile.sys par défaut
- **Persistance** : **PERDU** lors maintenances/redéploiements
- **Usage** : Cache, fichiers temporaires, TempDB

**Disques de Données (E:, F:, etc.)**
- **Type** : Persistants (Managed Disks)
- **Usage** : Données applicatives, bases de données

#### Types de Disques
- **Standard HDD** : Workloads peu fréquents, coût minimal
- **Standard SSD** : Workloads modérés, balance performance/coût
- **Premium SSD** : Workloads critiques, haute performance
- **Ultra Disk** : Workloads extrêmes, latence ultra-faible

#### Création de VM avec Data Disks

**⚠️ Erreur Courante QCM : Attacher plusieurs data disks dès la création**

**Scénario :** Créer une VM avec 3 disques de données (data disks) attachés dès la création.

**Solution : Azure CLI (Méthode la plus simple)**

```bash
# Créer une VM avec 3 data disks en une seule commande
az vm create \
  --resource-group myRG \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 256 512
  # Créera automatiquement 3 disques: 128GB, 256GB et 512GB
```

**Paramètres clés :**
- `--data-disk-sizes-gb` : Liste des tailles de disques séparées par espaces
- Les disques sont créés automatiquement et attachés
- LUN (Logical Unit Number) assignés automatiquement : 0, 1, 2, etc.

**Concepts Importants :**

**LUN (Logical Unit Number) :**
- **Identifiant unique** pour chaque disque attaché à une VM
- **Numérotation** : Commence à 0 (LUN 0, LUN 1, LUN 2, etc.)
- **Maximum** : Dépend de la taille de VM
  - Standard_D2s_v3 : Max 8 data disks (LUN 0-7)
  - Standard_D16s_v3 : Max 32 data disks (LUN 0-31)

**Persistance des Data Disks :**
- ✅ **PERSISTANTS** : Les données sont conservées lors arrêts/redémarrages
- ✅ **Managed Disks** : Gestion automatique par Azure
- ✅ **Snapshots** : Possibilité de créer des snapshots pour backup
- ❌ **Différent du disque temporaire (D:)** qui est volatile

**Scénarios d'Examen :**

| Question | Réponse | Raison |
|----------|---------|--------|
| Créer VM avec 3 disques de données | `--data-disk-sizes-gb 128 256 512` | Azure CLI option la plus simple |
| Maximum de data disks sur Standard_D2s_v3 | 8 disks | Dépend de la taille de VM |
| Les data disks sont-ils persistants ? | Oui | Contrairement au disque temporaire D: |
| Identifier un data disk dans la VM | Par LUN (0, 1, 2...) | Logical Unit Number |

#### Accès Externe aux VMs

**⚠️ Erreur Courante QCM : Minimize Administrative Effort**

**Scénario :** Une VM est accessible uniquement depuis le réseau interne. Vous devez permettre l'accès depuis Internet avec **effort administratif minimal**.

**❌ FAUX :** Configurer un VPN Site-to-Site
**✅ CORRECT :** **Ajouter une adresse IP publique** à la VM

**Comparaison des Solutions :**

| Solution | Complexité | Coût | Temps | Effort Admin |
|----------|------------|------|-------|--------------|
| **IP Publique** | ✅ Très faible | ~$3/mois | 2 min | ✅ Minimal |
| **Azure Bastion** | Moyenne | ~$150/mois | 15 min | Moyen |
| **VPN Site-to-Site** | ❌ Élevée | ~$25-150/mois | 1-2h | ❌ Élevé |

**Solution Recommandée - Ajouter une IP Publique :**

```bash
# 1. Créer une IP publique
az network public-ip create \
  --resource-group myRG \
  --name myVM-PublicIP \
  --sku Standard \
  --allocation-method Static

# 2. Associer l'IP publique à la NIC de la VM
az network nic ip-config update \
  --resource-group myRG \
  --nic-name myVM-NIC \
  --name ipconfig1 \
  --public-ip-address myVM-PublicIP
```

**⚠️ Sécurité - NSG Recommandé :**

```bash
# Limiter l'accès SSH/RDP à votre IP uniquement
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myVM-NSG \
  --name Allow-SSH-MyIP \
  --priority 100 \
  --source-address-prefixes 203.0.113.50/32 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp
```

**Pourquoi PAS VPN Site-to-Site ?**
- ❌ **Complexe** : Local Network Gateway, Virtual Network Gateway, Connection
- ❌ **Temps** : 1-2 heures de configuration
- ❌ **Coût** : Gateway ~$25-150/mois
- ⚠️ **Use case** : Connectivité réseau entier, pas une seule VM

**Scénarios d'Examen :**
- **"Minimal administrative effort"** → Add Public IP
- **"Secure access without exposing ports"** → Azure Bastion
- **"Connect entire on-premises network"** → VPN Site-to-Site

#### High Availability

**Availability Sets**

**Définition :**
- **Groupement logique** de VMs pour assurer la haute disponibilité
- **Protection** contre pannes matérielles ET maintenances planifiées
- **Placement** : VMs distribuées sur plusieurs racks physiques
- **Scope** : Même datacenter (single region, single datacenter)
- **SLA** : 99.95% (au moins 1 VM disponible)
- **Gratuit** : Aucun coût supplémentaire pour l'availability set lui-même

**Composants Clés :**

**1. Fault Domains (FD) - Domaines de Défaillance**
- **Définition** : Représente un rack physique dans le datacenter
- **Contenu** : Serveurs, switch réseau, source d'alimentation partagés
- **Maximum** : 3 Fault Domains par availability set
- **Protection** : Panne matérielle (hardware failure, power outage, network switch failure)
- **Distribution** : Azure répartit automatiquement vos VMs sur les FDs

**Visualisation des Fault Domains :**
```
Datacenter Azure
├── Rack 1 (FD 0) - Alimentation A, Switch A
│   ├── VM1
│   └── VM4
├── Rack 2 (FD 1) - Alimentation B, Switch B
│   ├── VM2
│   └── VM5
└── Rack 3 (FD 2) - Alimentation C, Switch C
    ├── VM3
    └── VM6
```

**Scénario de panne FD :**
- **Problème** : Rack 1 perd l'alimentation électrique
- **Impact** : Seules VM1 et VM4 sont affectées
- **Résultat** : VM2, VM3, VM5, VM6 continuent de fonctionner
- **Disponibilité** : Au moins 2/3 des VMs restent disponibles

**2. Update Domains (UD) - Domaines de Mise à Jour**
- **Définition** : Groupe logique de VMs redémarrées ensemble lors de maintenances
- **Par défaut** : 5 Update Domains (si non spécifié à la création)
- **Configurable** : De 1 à 20 Update Domains maximum
- **Important** : Configuration définie à la création, non modifiable après
- **Protection** : Maintenance planifiée Azure (host OS updates, hardware maintenance)
- **Processus** : Azure redémarre un seul UD à la fois
- **Délai** : 30 minutes entre chaque UD redémarré

**Visualisation des Update Domains :**
```
Update Domain 0: VM1, VM6
Update Domain 1: VM2, VM7
Update Domain 2: VM3, VM8
Update Domain 3: VM4, VM9
Update Domain 4: VM5, VM10

Maintenance planifiée :
Étape 1: Redémarre UD0 → VM1, VM6 offline, autres OK
[Attend 30 min]
Étape 2: Redémarre UD1 → VM2, VM7 offline, autres OK
[Attend 30 min]
Étape 3: Redémarre UD2 → VM3, VM8 offline, autres OK
...
```

**Scénario de maintenance UD :**
- **Événement** : Maintenance planifiée Azure
- **Processus** : Azure redémarre UD par UD (jamais simultanément)
- **Impact** : Maximum 1/5 (20%) des VMs offline à un moment donné
- **Garantie** : Au moins 80% de capacité maintenue

**3. Combinaison FD + UD - Protection Double**

**Matrice de Distribution (Exemple : 6 VMs, 3 FD, 3 UD) :**
```
              FD 0 (Rack 1)  FD 1 (Rack 2)  FD 2 (Rack 3)
UD 0            VM1             VM2             VM3
UD 1            VM4             VM5             VM6
UD 2            VM7             VM8             VM9
```

**Protection simultanée :**
- **Panne FD 1** : Perd VM2, VM5, VM8 → Garde VM1,3,4,6,7,9
- **Maintenance UD 0** : Redémarre VM1, VM2, VM3 → Garde VM4,5,6,7,8,9
- **Combiné** : Jamais plus d'un FD ET un UD affecté en même temps

**Configuration et Contraintes :**

**Création :**
- **Moment** : Doit être configuré AVANT ou PENDANT la création de la VM
- **Modification** : IMPOSSIBLE de changer l'availability set d'une VM existante
- **Solution** : Recréer la VM si changement nécessaire

**Limitations importantes :**
- **Zone géographique** : Limité à un seul datacenter
- **Maximum VMs** : Limite pratique (recommandé : < 200 VMs)
- **Famille de VM** : Toutes les VMs doivent être de tailles compatibles
- **Managed Disks** : Fortement recommandé (Aligned ou non)
- **Incompatibilité** : Ne peut PAS combiner avec Availability Zones

**Configuration PowerShell :**
```powershell
# Créer l'Availability Set
New-AzAvailabilitySet `
  -ResourceGroupName "myRG" `
  -Name "myAvailabilitySet" `
  -Location "East US" `
  -PlatformFaultDomainCount 3 `
  -PlatformUpdateDomainCount 5 `
  -Sku Aligned

# Créer VM dans l'Availability Set
New-AzVM `
  -ResourceGroupName "myRG" `
  -Name "myVM" `
  -AvailabilitySetName "myAvailabilitySet" `
  ...
```

**Configuration Azure CLI :**
```bash
# Créer l'Availability Set
az vm availability-set create \
  --resource-group myRG \
  --name myAvailabilitySet \
  --platform-fault-domain-count 3 \
  --platform-update-domain-count 5

# Créer VM dans l'Availability Set
az vm create \
  --resource-group myRG \
  --name myVM \
  --availability-set myAvailabilitySet \
  ...
```

**Best Practices :**

1. **Minimum 2 VMs par tier** : Pour bénéficier du SLA 99.95%
2. **Utiliser Managed Disks** : Meilleure résilience (Storage Fault Domains)
3. **Load Balancer** : Combiner avec Azure Load Balancer pour distribution trafic
4. **Même fonction** : Regrouper uniquement VMs ayant le même rôle
5. **Planification** : Configurer dès la création, impossible à modifier après

**Cas d'usage typiques :**
- **Web tier** : Serveurs web dans un availability set
- **App tier** : Serveurs applicatifs dans un autre availability set
- **Data tier** : Serveurs de base de données dans un troisième availability set
- **Séparation** : Un availability set différent par tier applicatif

**Comparaison rapide :**
| Caractéristique | Availability Sets | Availability Zones |
|-----------------|-------------------|-------------------|
| **Protection** | Rack failure | Datacenter failure |
| **Fault Domains** | Max 3 | 3 zones |
| **Scope** | Single datacenter | Multiple datacenters |
| **SLA** | 99.95% | 99.99% |
| **Latence** | Minimale | Légèrement plus élevée |
| **Coût** | Gratuit | Frais de transfert inter-zones |

**Availability Zones**
- **Protection** : Datacenters physiquement séparés
- **SLA** : 99.99%
- **Scope** : Région Azure

** Prérequis identifiés pour Availability Zones :**
- **Managed Disks** : Obligatoire
- **Availability Options** : Doit être configuré à la création

#### Accès Externe aux VMs
** Cas d'usage identifié :** Accès externe avec effort administratif minimal
- **Scénario** : VM interne accessible uniquement depuis le réseau interne, besoin d'accès externe
- **Solution optimale** : Ajouter une adresse IP publique à la VM
- **Avantage** : Configuration simple et directe, effort administratif minimal

** Erreur fréquente identifiée :** Complexité inutile pour accès externe simple
- ** Erreur** : Configurer un VPN Site-to-Site pour un accès externe simple
- ** Correct** : Utiliser une adresse IP publique pour minimiser l'effort administratif
- **Raison** : VPN S2S = Configuration complexe (Local Network Gateway, Connection, etc.)
- **Alternative** : IP publique = Configuration simple et directe

### 3.2 Virtual Machine Scale Sets (VMSS)

#### Concepts Clés
- **Scaling automatique** : Basé sur métriques ou planning
- **Load balancing** : Intégré avec Azure Load Balancer
- **Update management** : Rolling updates avec availability

** Cas d'usage identifié :** Maintenance Azure
- Pour garantir **8 VMs minimum** pendant maintenance
- Créer un **VMSS avec 10 instances**
- Azure maintient une partie, les autres restent disponibles

#### Scaling Policies
- **Manual** : Contrôle manuel du nombre d'instances
- **Automatic** : Basé sur CPU, mémoire, métriques custom
- **Scheduled** : Scaling programmé

### 3.3 App Services

#### Service Plans - Tiers Détaillés

**⚠️ Erreur Courante QCM : Capacités de scaling par tier**

**Comparaison Complète des Tiers :**

| Tier | SKUs | Max Instances | Autoscale | Deployment Slots | Custom Domain | SSL | Prix/mois |
|------|------|---------------|-----------|------------------|---------------|-----|-----------|
| **Free** | F1 | 1 (partagé) | ❌ Non | ❌ Non | ❌ Non | ❌ Non | Gratuit |
| **Shared** | D1 | 1 (partagé) | ❌ Non | ❌ Non | ✅ Oui | ❌ Non | ~$10 |
| **Basic** | B1-B3 | 3 | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui | ~$55-220 |
| **Standard** | S1-S3 | 10 | ✅ Oui | ✅ 5 slots | ✅ Oui | ✅ Oui | ~$70-280 |
| **Premium** | P1v2-P3v2 | 30 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$150-600 |
| **PremiumV3** | P1v3-P3v3 | 30 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$200-800 |
| **Isolated** | I1-I3 | 100 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$650+ |

**Caractéristiques par Tier :**

**Free / Shared (Development) :**
- **Infrastructure** : Partagée avec autres clients
- **Limitations** : Quotas CPU (60 min/jour pour Free)
- **Downtime** : Possible si app inactive > 20 min
- **Use case** : Développement, POC uniquement
- ❌ **Production** : Non recommandé

**Basic (Small Production) :**
- **Infrastructure** : VMs dédiées
- **Scaling** : Manuel uniquement (1-3 instances)
- **Limitations** : Pas d'autoscale, pas de slots
- **Use case** : Petites apps production, budget limité
- **SLA** : 99.95%

**Standard (Production) :**
- **Infrastructure** : VMs dédiées
- **Scaling** : Manuel + Autoscale (1-10 instances)
- **Features** : Deployment slots (5), Traffic Manager
- **Use case** : Apps production standard
- **SLA** : 99.95%

**Premium (High Performance) :**
- **Infrastructure** : VMs dédiées plus puissantes
- **Scaling** : Manuel + Autoscale (1-30 instances)
- **Features** : Deployment slots (20), VNet integration
- **Use case** : Apps critiques, haute performance
- **SLA** : 99.95%

**Isolated (Enterprise) :**
- **Infrastructure** : App Service Environment (ASE) dédié
- **Isolation** : Réseau isolé, pas de partage
- **Scaling** : Jusqu'à 100 instances
- **Use case** : Conformité, sécurité maximale
- **SLA** : 99.95%

#### App Service Plan Scaling - Détaillé

**⚠️ Erreur Courante QCM : Scale Up vs Scale Out**

**Types de Scaling Disponibles :**

**1. Scale Up (Vertical Scaling) - Changer de Tier/SKU**

**Définition :** Augmenter les ressources (CPU, RAM, Storage) de chaque instance

**Quand utiliser :**
- Application nécessite plus de CPU/RAM
- Atteinte des limites de l'instance actuelle
- Besoin de nouvelles fonctionnalités (ex: deployment slots)

**Exemple - Azure CLI :**
```bash
# Upgrader de B1 vers S1
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --sku S1

# Upgrader vers Premium
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --sku P1V2
```

**Impact :**
- ⏱️ **Downtime** : Bref redémarrage (quelques secondes)
- 💰 **Coût** : Augmente selon le nouveau tier
- 🎯 **Performance** : Chaque instance plus puissante

**2. Scale Out (Horizontal Scaling) - Ajouter des Instances**

**Définition :** Augmenter le nombre d'instances (VMs) qui exécutent l'application

**Quand utiliser :**
- Trafic élevé, besoin de plus de capacité
- Haute disponibilité requise
- Distribution de charge

**Exemple - Azure CLI :**
```bash
# Scale Out manuel - Ajouter 5 instances
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --number-of-workers 5
```

**Impact :**
- ⏱️ **Downtime** : ✅ Aucun
- 💰 **Coût** : Proportionnel au nombre d'instances
- 🎯 **Performance** : Capacité totale multipliée

**3. Autoscale (Automatic Horizontal Scaling)**

**Disponible à partir de :** Standard (S1) et supérieur

**Configuration Autoscale :**
```bash
# Créer une règle d'autoscale
az monitor autoscale create \
  --resource-group myRG \
  --resource myAppServicePlan \
  --resource-type Microsoft.Web/serverFarms \
  --name myAutoscaleRule \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Règle basée sur CPU (Scale Out)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscaleRule \
  --condition "CpuPercentage > 75 avg 5m" \
  --scale out 1

# Règle basée sur CPU (Scale In)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscaleRule \
  --condition "CpuPercentage < 25 avg 5m" \
  --scale in 1
```

**Métriques disponibles pour Autoscale :**
- **CPU Percentage** : % utilisation CPU
- **Memory Percentage** : % utilisation RAM
- **Disk Queue Length** : File d'attente disque
- **Http Queue Length** : File d'attente requêtes HTTP
- **Data In/Out** : Bande passante réseau
- **Custom metrics** : Application Insights

**Configuration via Portal :**
```
App Service Plan → Scale out (App Service plan) → Autoscale
→ Add a rule → Metric (CPU Percentage > 70)
→ Operation (Increase count by 1)
→ Cool down (5 minutes)
```

**Best Practices Autoscale :**
- ✅ **Minimum 2 instances** pour haute disponibilité
- ✅ **Cool down period** : 5-10 minutes (éviter flapping)
- ✅ **Margins** : Scale out à 70%, scale in à 30%
- ⚠️ **Limites** : Définir max instances pour contrôler coûts

**Matrice de Décision - Scaling :**

| Symptôme | Solution | Commande |
|----------|----------|----------|
| **CPU/RAM saturé** sur instances | Scale Up | Change SKU (B1→S1) |
| **Trafic élevé**, instances saturées | Scale Out | Augmenter instances |
| **Trafic variable** (pics) | Autoscale | Règles basées métriques |
| **Besoin de deployment slots** | Scale Up | Upgrade vers S1+ |
| **Latence élevée** | Scale Out | Plus d'instances |
| **Coûts trop élevés** | Scale Down/In | Réduire tier ou instances |

**Limitations par Tier :**

| Tier | Scale Out Manuel | Autoscale | Max Instances |
|------|-----------------|-----------|---------------|
| Free/Shared | ❌ Non | ❌ Non | 1 (partagé) |
| Basic | ✅ Oui | ❌ Non | 3 |
| Standard | ✅ Oui | ✅ Oui | 10 |
| Premium | ✅ Oui | ✅ Oui | 30 |
| Isolated | ✅ Oui | ✅ Oui | 100 |

**Scénarios d'Examen :**

| Question | Réponse | Raison |
|----------|---------|--------|
| App nécessite plus de RAM | **Scale Up** | Augmenter ressources par instance |
| Trafic web élevé, pic de charge | **Scale Out** | Ajouter instances pour capacité |
| Besoin deployment slots | **Scale Up to Standard** | Slots disponibles à partir de S1 |
| Trafic variable jour/nuit | **Autoscale** | Ajustement automatique |
| Minimiser downtime pendant scaling | **Scale Out** | Pas de redémarrage |

**PowerShell - Gestion Complète :**
```powershell
# Scale Up
Set-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" `
  -Tier "Standard" `
  -WorkerSize "Medium"

# Scale Out
Set-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" `
  -NumberofWorkers 5

# Get current scale
Get-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" | 
  Select-Object Name, Sku, NumberOfWorkers
```

#### Runtime Stacks et OS
** Point clé identifié :** Un App Service = Un runtime
- Chaque App Service configuré pour **un seul runtime**
- Python OU Java OU C# - pas de mix possible
- Un Plan App Service peut héberger plusieurs App Services du même OS

#### Deployment Slots

** Workflow optimal identifié :**
1. **Deploy** → Déployer l'update sur staging slot
2. **Test** → Tester l'application sur staging
3. **Swap** → Basculer staging vers production

**Avantages du Slot Swapping :**
- **Zero downtime** : Aucune interruption
- **Warm-up automatique** : Instances préchauffées
- **Rollback instant** : Re-swap pour annuler
- **Configuration preservation** : Settings spécifiques aux slots

#### Authentication et Authorization
** Configuration identifiée :** Désactiver l'accès anonyme
- Configurer **Authentication** dans App Service
- Ajouter identity providers : Microsoft, Google, Facebook, Twitter
- **Anonymous access** est une méthode d'authentification

### 3.4 Azure Container Instances (ACI)

#### Caractéristiques
- **Serverless containers** : Pas de gestion d'infrastructure
- **Billing per second** : Facturation à la seconde
- **Quick start** : Démarrage en secondes
- **Support** : Linux et Windows containers

#### Use Cases
- **Burst workloads** : Scaling rapide
- **Build agents** : CI/CD pipelines
- **Data processing** : Jobs batch
- **Development/testing** : Environnements temporaires

### 3.5 Infrastructure as Code (IaC) - ARM, Bicep, Terraform

#### ARM Templates (Azure Resource Manager Templates)

**⚠️ Fondamental pour l'Examen AZ-104**

**Définition :**
ARM Templates sont des fichiers JSON déclaratifs qui définissent l'infrastructure Azure de manière programmatique (Infrastructure as Code).

**Avantages :**
- ✅ **Déclaratif** : Décrivez "quoi" déployer, pas "comment"
- ✅ **Idempotent** : Même résultat à chaque exécution
- ✅ **Orchestration** : Déploie ressources dans le bon ordre
- ✅ **Validation** : Test avant déploiement
- ✅ **Modularité** : Templates réutilisables
- ✅ **Versioning** : Intégration Git/DevOps

**Structure Complète ARM Template :**

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "metadata": {
    "description": "Déploiement VM Windows avec disques et réseau"
  },
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "myVM",
      "metadata": {
        "description": "Nom de la machine virtuelle"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3",
      "allowedValues": [
        "Standard_B2s",
        "Standard_D2s_v3",
        "Standard_D4s_v3"
      ],
      "metadata": {
        "description": "Taille de la VM"
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Nom d'utilisateur administrateur"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Mot de passe administrateur"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Localisation des ressources"
      }
    }
  },
  "variables": {
    "vnetName": "[concat(parameters('vmName'), '-vnet')]",
    "subnetName": "default",
    "nicName": "[concat(parameters('vmName'), '-nic')]",
    "publicIPName": "[concat(parameters('vmName'), '-pip')]",
    "nsgName": "[concat(parameters('vmName'), '-nsg')]",
    "osDiskName": "[concat(parameters('vmName'), '-osdisk')]",
    "addressPrefix": "10.0.0.0/16",
    "subnetPrefix": "10.0.1.0/24"
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2023-04-01",
      "name": "[variables('vnetName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[variables('addressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "[variables('subnetPrefix')]"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2023-04-01",
      "name": "[variables('publicIPName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2023-04-01",
      "name": "[variables('nsgName')]",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "RDP",
            "properties": {
              "priority": 100,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "3389"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2023-04-01",
      "name": "[variables('nicName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]",
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]",
        "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
              },
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))]"
              }
            }
          }
        ],
        "networkSecurityGroup": {
          "id": "[resourceId('Microsoft.Network/networkSecurityGroups', variables('nsgName'))]"
        }
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2023-03-01",
      "name": "[parameters('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "osProfile": {
          "computerName": "[parameters('vmName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2022-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "name": "[variables('osDiskName')]",
            "createOption": "FromImage",
            "managedDisk": {
              "storageAccountType": "Premium_LRS"
            }
          },
          "dataDisks": [
            {
              "lun": 0,
              "name": "[concat(parameters('vmName'), '-datadisk1')]",
              "createOption": "Empty",
              "diskSizeGB": 128,
              "managedDisk": {
                "storageAccountType": "Premium_LRS"
              }
            }
          ]
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "vmName": {
      "type": "string",
      "value": "[parameters('vmName')]"
    },
    "publicIP": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))).ipAddress]"
    },
    "resourceId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
    }
  }
}
```

**Sections du Template ARM :**

**1. `$schema` et `contentVersion` :**
- **$schema** : Définit le schéma de validation
- **contentVersion** : Version du template (pour tracking)

**2. `parameters` - Valeurs Configurables :**
```json
"parameters": {
  "vmName": {
    "type": "string",
    "defaultValue": "myVM",
    "minLength": 1,
    "maxLength": 15,
    "metadata": {
      "description": "Nom de la VM"
    }
  },
  "adminPassword": {
    "type": "securestring"
  },
  "vmSize": {
    "type": "string",
    "allowedValues": ["Standard_B2s", "Standard_D2s_v3"]
  }
}
```

**Types de paramètres :**
- `string` : Texte
- `securestring` : Mot de passe (non affiché dans logs)
- `int` : Nombre entier
- `bool` : Boolean
- `object` : Objet JSON
- `array` : Tableau
- `secureObject` : Objet sécurisé

**3. `variables` - Valeurs Calculées :**
```json
"variables": {
  "nicName": "[concat(parameters('vmName'), '-nic')]",
  "subnetRef": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]",
  "isProduction": "[equals(parameters('environment'), 'prod')]"
}
```

**Fonctions ARM utiles :**
- `concat()` : Concaténation de chaînes
- `resourceId()` : Obtenir l'ID d'une ressource
- `reference()` : Obtenir les propriétés d'une ressource
- `parameters()` : Accéder aux paramètres
- `variables()` : Accéder aux variables
- `resourceGroup()` : Info sur le resource group
- `subscription()` : Info sur la subscription
- `uniqueString()` : Générer chaîne unique
- `equals()`, `greater()`, `less()` : Comparaisons
- `if()` : Condition ternaire

**4. `resources` - Ressources Azure :**
```json
"resources": [
  {
    "type": "Microsoft.Compute/virtualMachines",
    "apiVersion": "2023-03-01",
    "name": "[parameters('vmName')]",
    "location": "[parameters('location')]",
    "dependsOn": [
      "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
    ],
    "properties": {
      // Configuration de la ressource
    }
  }
]
```

**`dependsOn` - Ordre de Déploiement :**
- Spécifie les dépendances entre ressources
- Azure déploie les ressources dans le bon ordre
- Utilise `resourceId()` pour référencer

**5. `outputs` - Valeurs Retournées :**
```json
"outputs": {
  "publicIP": {
    "type": "string",
    "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('publicIPName'))).ipAddress]"
  },
  "vmId": {
    "type": "string",
    "value": "[resourceId('Microsoft.Compute/virtualMachines', parameters('vmName'))]"
  }
}
```

**Déploiement ARM Template :**

**Via Azure CLI :**
```bash
# Déployer template avec paramètres inline
az deployment group create \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters vmName=myVM adminUsername=azureuser adminPassword='P@ssw0rd123!'

# Déployer avec fichier de paramètres
az deployment group create \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json

# Valider avant déploiement (What-If)
az deployment group validate \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json

# What-If - Prévisualiser changements
az deployment group what-if \
  --resource-group myRG \
  --template-file azuredeploy.json \
  --parameters @azuredeploy.parameters.json
```

**Via PowerShell :**
```powershell
# Déployer template
New-AzResourceGroupDeployment `
  -ResourceGroupName "myRG" `
  -TemplateFile "azuredeploy.json" `
  -TemplateParameterFile "azuredeploy.parameters.json" `
  -Verbose

# Test deployment (validation)
Test-AzResourceGroupDeployment `
  -ResourceGroupName "myRG" `
  -TemplateFile "azuredeploy.json" `
  -TemplateParameterFile "azuredeploy.parameters.json"

# What-If
New-AzResourceGroupDeployment `
  -ResourceGroupName "myRG" `
  -TemplateFile "azuredeploy.json" `
  -WhatIf
```

**Fichier de Paramètres (azuredeploy.parameters.json) :**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "value": "myVM"
    },
    "vmSize": {
      "value": "Standard_D2s_v3"
    },
    "adminUsername": {
      "value": "azureuser"
    },
    "adminPassword": {
      "reference": {
        "keyVault": {
          "id": "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myKeyVault"
        },
        "secretName": "vmAdminPassword"
      }
    }
  }
}
```

**Linked Templates - Modularité :**
```json
{
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2021-04-01",
  "name": "linkedDeployment",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "https://mystorageaccount.blob.core.windows.net/templates/network.json",
      "contentVersion": "1.0.0.0"
    },
    "parameters": {
      "vnetName": {
        "value": "[variables('vnetName')]"
      }
    }
  }
}
```

**Deployment Modes :**

| Mode | Comportement | Use Case |
|------|--------------|----------|
| **Incremental** (Default) | Ajoute/modifie ressources, ne supprime pas | Ajout progressif de ressources |
| **Complete** | Supprime ressources non dans template | Déploiement complet contrôlé |

**Best Practices ARM Templates :**

✅ **À FAIRE :**
- Utiliser parameters files pour différents environnements
- Stocker passwords dans Azure Key Vault
- Utiliser `dependsOn` explicitement
- Valider avec `az deployment group validate`
- Tester avec `what-if` avant production
- Versionner templates dans Git
- Utiliser linked templates pour modularité
- Documenter avec `metadata.description`

❌ **À ÉVITER :**
- Hardcoder passwords dans templates
- Templates monolithiques (> 1000 lignes)
- Complete mode sans tester (risque suppression)
- Ignorer les erreurs de validation

#### Bicep - Alternative Moderne aux ARM Templates

**⚠️ Important pour l'Examen AZ-104**

**Définition :**
Bicep est un langage déclaratif pour déployer des ressources Azure. C'est une **abstraction simplifiée** d'ARM Templates JSON.

**Avantages Bicep vs ARM JSON :**
- ✅ **Syntaxe simplifiée** : Moins verbose (50% moins de code)
- ✅ **Type safety** : Validation au moment de l'écriture
- ✅ **Intellisense** : Support IDE (VS Code extension)
- ✅ **Modules** : Réutilisation facile
- ✅ **No state file** : Contrairement à Terraform
- ✅ **Transpilation** : Bicep → ARM JSON automatique

**Même VM en Bicep (comparaison) :**

```bicep
// Paramètres
@description('Nom de la VM')
@minLength(1)
@maxLength(15)
param vmName string = 'myVM'

@description('Taille de la VM')
@allowed([
  'Standard_B2s'
  'Standard_D2s_v3'
  'Standard_D4s_v3'
])
param vmSize string = 'Standard_D2s_v3'

@description('Nom d\'utilisateur administrateur')
param adminUsername string

@secure()
@description('Mot de passe administrateur')
param adminPassword string

param location string = resourceGroup().location

// Variables
var vnetName = '${vmName}-vnet'
var subnetName = 'default'
var nicName = '${vmName}-nic'
var publicIPName = '${vmName}-pip'
var nsgName = '${vmName}-nsg'
var osDiskName = '${vmName}-osdisk'
var addressPrefix = '10.0.0.0/16'
var subnetPrefix = '10.0.1.0/24'

// Ressources
resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [
        addressPrefix
      ]
    }
    subnets: [
      {
        name: subnetName
        properties: {
          addressPrefix: subnetPrefix
        }
      }
    ]
  }
}

resource publicIP 'Microsoft.Network/publicIPAddresses@2023-04-01' = {
  name: publicIPName
  location: location
  sku: {
    name: 'Standard'
  }
  properties: {
    publicIPAllocationMethod: 'Static'
  }
}

resource nsg 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
  name: nsgName
  location: location
  properties: {
    securityRules: [
      {
        name: 'RDP'
        properties: {
          priority: 100
          protocol: 'Tcp'
          access: 'Allow'
          direction: 'Inbound'
          sourceAddressPrefix: '*'
          sourcePortRange: '*'
          destinationAddressPrefix: '*'
          destinationPortRange: '3389'
        }
      }
    ]
  }
}

resource nic 'Microsoft.Network/networkInterfaces@2023-04-01' = {
  name: nicName
  location: location
  properties: {
    ipConfigurations: [
      {
        name: 'ipconfig1'
        properties: {
          subnet: {
            id: vnet.properties.subnets[0].id
          }
          publicIPAddress: {
            id: publicIP.id
          }
        }
      }
    ]
    networkSecurityGroup: {
      id: nsg.id
    }
  }
}

resource vm 'Microsoft.Compute/virtualMachines@2023-03-01' = {
  name: vmName
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: vmName
      adminUsername: adminUsername
      adminPassword: adminPassword
    }
    storageProfile: {
      imageReference: {
        publisher: 'MicrosoftWindowsServer'
        offer: 'WindowsServer'
        sku: '2022-Datacenter'
        version: 'latest'
      }
      osDisk: {
        name: osDiskName
        createOption: 'FromImage'
        managedDisk: {
          storageAccountType: 'Premium_LRS'
        }
      }
      dataDisks: [
        {
          lun: 0
          name: '${vmName}-datadisk1'
          createOption: 'Empty'
          diskSizeGB: 128
          managedDisk: {
            storageAccountType: 'Premium_LRS'
          }
        }
      ]
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nic.id
        }
      ]
    }
  }
}

// Outputs
output vmName string = vmName
output publicIP string = publicIP.properties.ipAddress
output vmId string = vm.id
```

**Différences Clés Bicep vs ARM JSON :**

| Feature | ARM JSON | Bicep |
|---------|----------|-------|
| **Syntaxe** | Verbeux, brackets | Concis, naturel |
| **Variables** | `[variables('name')]` | `varName` |
| **Parameters** | `[parameters('name')]` | `paramName` |
| **String concat** | `[concat('a', 'b')]` | `'${a}${b}'` |
| **Resource ID** | `[resourceId('...')]` | `resource.id` |
| **Properties** | `[reference('...').prop]` | `resource.properties.prop` |
| **Dependencies** | `dependsOn` explicite | Automatique (référence symbolique) |

**Bicep Modules - Réutilisation :**

**Module : storage.bicep**
```bicep
param storageAccountName string
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageAccountId string = storageAccount.id
output primaryEndpoint string = storageAccount.properties.primaryEndpoints.blob
```

**Main template : main.bicep**
```bicep
module storage 'storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: 'mystorageaccount'
    location: 'eastus'
  }
}

output storageId string = storage.outputs.storageAccountId
```

**Déploiement Bicep :**

```bash
# Installer Bicep CLI (si pas déjà fait)
az bicep install

# Upgrader Bicep
az bicep upgrade

# Déployer Bicep template
az deployment group create \
  --resource-group myRG \
  --template-file main.bicep \
  --parameters vmName=myVM adminUsername=azureuser adminPassword='P@ssw0rd123!'

# Build Bicep vers ARM JSON (pour inspection)
az bicep build --file main.bicep

# Decompile ARM JSON vers Bicep
az bicep decompile --file azuredeploy.json

# Valider Bicep
az deployment group validate \
  --resource-group myRG \
  --template-file main.bicep

# What-If avec Bicep
az deployment group what-if \
  --resource-group myRG \
  --template-file main.bicep
```

**Bicep Loops - Itération :**
```bicep
param vmCount int = 3

resource vms 'Microsoft.Compute/virtualMachines@2023-03-01' = [for i in range(0, vmCount): {
  name: 'vm-${i}'
  location: resourceGroup().location
  properties: {
    hardwareProfile: {
      vmSize: 'Standard_B2s'
    }
    // ... autres propriétés
  }
}]
```

**Bicep Conditions :**
```bicep
param deployPublicIP bool = true

resource publicIP 'Microsoft.Network/publicIPAddresses@2023-04-01' = if (deployPublicIP) {
  name: 'myPublicIP'
  location: resourceGroup().location
  properties: {
    publicIPAllocationMethod: 'Static'
  }
}
```

#### Terraform pour Azure

**⚠️ Moins Important pour AZ-104 mais Bon à Connaître**

**Définition :**
Terraform est un outil IaC **multi-cloud** de HashiCorp. Utilise HCL (HashiCorp Configuration Language).

**Terraform vs ARM/Bicep :**

| Feature | ARM/Bicep | Terraform |
|---------|-----------|-----------|
| **Scope** | Azure uniquement | Multi-cloud (Azure, AWS, GCP) |
| **Language** | JSON / Bicep | HCL |
| **State** | Pas de state file | State file requis |
| **Provider** | Natif Azure | AzureRM provider |
| **Modules** | Bicep modules | Terraform modules |
| **What-If** | `az deployment what-if` | `terraform plan` |
| **Validation** | Intégrée Azure | Locale |

**Même VM en Terraform :**

```hcl
# Provider configuration
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

provider "azurerm" {
  features {}
}

# Variables
variable "vm_name" {
  description = "Nom de la VM"
  type        = string
  default     = "myVM"
}

variable "vm_size" {
  description = "Taille de la VM"
  type        = string
  default     = "Standard_D2s_v3"
}

variable "admin_username" {
  description = "Username administrateur"
  type        = string
}

variable "admin_password" {
  description = "Password administrateur"
  type        = string
  sensitive   = true
}

variable "location" {
  description = "Localisation Azure"
  type        = string
  default     = "East US"
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = "${var.vm_name}-rg"
  location = var.location
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "${var.vm_name}-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

# Subnet
resource "azurerm_subnet" "main" {
  name                 = "default"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

# Public IP
resource "azurerm_public_ip" "main" {
  name                = "${var.vm_name}-pip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Network Security Group
resource "azurerm_network_security_group" "main" {
  name                = "${var.vm_name}-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "RDP"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "3389"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
}

# Network Interface
resource "azurerm_network_interface" "main" {
  name                = "${var.vm_name}-nic"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  ip_configuration {
    name                          = "ipconfig1"
    subnet_id                     = azurerm_subnet.main.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.main.id
  }
}

# Associate NSG with NIC
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.main.id
}

# Virtual Machine
resource "azurerm_windows_virtual_machine" "main" {
  name                = var.vm_name
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = var.vm_size
  admin_username      = var.admin_username
  admin_password      = var.admin_password

  network_interface_ids = [
    azurerm_network_interface.main.id,
  ]

  os_disk {
    name                 = "${var.vm_name}-osdisk"
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }
}

# Data Disk
resource "azurerm_managed_disk" "data" {
  name                 = "${var.vm_name}-datadisk1"
  location             = azurerm_resource_group.main.location
  resource_group_name  = azurerm_resource_group.main.name
  storage_account_type = "Premium_LRS"
  create_option        = "Empty"
  disk_size_gb         = 128
}

resource "azurerm_virtual_machine_data_disk_attachment" "data" {
  managed_disk_id    = azurerm_managed_disk.data.id
  virtual_machine_id = azurerm_windows_virtual_machine.main.id
  lun                = 0
  caching            = "ReadWrite"
}

# Outputs
output "vm_name" {
  value = azurerm_windows_virtual_machine.main.name
}

output "public_ip" {
  value = azurerm_public_ip.main.ip_address
}

output "vm_id" {
  value = azurerm_windows_virtual_machine.main.id
}
```

**Commandes Terraform :**

```bash
# Initialiser Terraform (télécharge providers)
terraform init

# Valider syntaxe
terraform validate

# Formater code
terraform fmt

# Prévisualiser changements (What-If)
terraform plan

# Appliquer changements
terraform apply

# Appliquer sans confirmation
terraform apply -auto-approve

# Détruire infrastructure
terraform destroy

# Afficher outputs
terraform output

# Afficher state
terraform show

# Import ressource existante
terraform import azurerm_resource_group.main /subscriptions/{sub-id}/resourceGroups/myRG
```

**Terraform State - Gestion :**

Le state file (`terraform.tfstate`) contient l'état actuel de l'infrastructure.

**Local State (default) :**
```hcl
# State stocké localement (terraform.tfstate)
# ⚠️ Ne pas commiter dans Git !
```

**Remote State (recommandé) :**
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "terraform.tfstate"
  }
}
```

**Terraform Modules :**
```hcl
# Module pour créer VMs
module "vm" {
  source = "./modules/vm"
  
  vm_name        = "myVM"
  vm_size        = "Standard_D2s_v3"
  admin_username = "azureuser"
  admin_password = var.admin_password
  location       = "East US"
}

output "vm_public_ip" {
  value = module.vm.public_ip
}
```

#### Comparaison Complète : ARM vs Bicep vs Terraform

| Critère | ARM Templates | Bicep | Terraform |
|---------|---------------|-------|-----------|
| **Format** | JSON | Bicep (DSL) | HCL |
| **Cloud** | Azure only | Azure only | Multi-cloud |
| **Verbosité** | Élevée | Faible | Moyenne |
| **Courbe apprentissage** | Élevée | Faible | Moyenne |
| **State management** | ❌ Aucun | ❌ Aucun | ✅ State file |
| **IDE Support** | Basique | ✅ Excellent (VS Code) | ✅ Excellent |
| **Modules** | Linked templates | ✅ Natif | ✅ Natif |
| **Community** | Microsoft | Microsoft | HashiCorp + Community |
| **Validation** | Azure | Azure | Locale |
| **What-If** | ✅ Oui | ✅ Oui | ✅ Plan |
| **Maturité** | +++++ | +++ | ++++ |
| **AZ-104 Exam** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

**Quand Utiliser Chaque Outil :**

| Scénario | Recommandation |
|----------|----------------|
| **Azure uniquement, nouveau projet** | **Bicep** |
| **Azure uniquement, legacy** | ARM Templates |
| **Multi-cloud (Azure + AWS + GCP)** | **Terraform** |
| **Équipe déjà Terraform** | Terraform |
| **Besoin validation native Azure** | ARM/Bicep |
| **Examen AZ-104** | ARM Templates + Bicep |

#### Scénarios d'Examen - IaC

**⚠️ Questions Typiques AZ-104**

| Question | Réponse | Justification |
|----------|---------|---------------|
| **Déployer 100 VMs identiques** | ARM Template avec copy loop | Automatisation, consistance |
| **Resource Group configurable au déploiement** | Parameter dans template | Seul paramètre runtime disponible |
| **Réutiliser template pour Dev/Staging/Prod** | Parameters files séparés | Même template, paramètres différents |
| **Valider template sans déployer** | `az deployment group validate` ou `what-if` | Test avant production |
| **Syntaxe simplifiée vs ARM JSON** | Bicep | 50% moins de code |
| **Multi-cloud Azure + AWS** | Terraform | Support multi-cloud |
| **Export configuration ressources existantes** | `az group export --name myRG` | Générer template depuis existant |

**Export Template depuis Ressources Existantes :**

```bash
# Exporter toutes ressources d'un Resource Group
az group export \
  --name myRG \
  --output json > exported-template.json

# Exporter template d'un déploiement
az deployment group export \
  --name myDeployment \
  --resource-group myRG \
  --output json > deployment-template.json
```

```powershell
# Export via PowerShell
Export-AzResourceGroup `
  -ResourceGroupName "myRG" `
  -Path "exported-template.json" `
  -Force
```

---
