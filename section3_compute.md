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

### 3.5 ARM Templates

#### Structure de Base
```json
{
    "$schema": "...",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {}
}
```

** Point identifié :** Déploiement depuis template
- **Resource Group** : Seul paramètre configurable lors du déploiement
- Toutes autres configurations définies dans le template
- Template = Infrastructure as Code

---
