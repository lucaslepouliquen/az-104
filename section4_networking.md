## 4. Configure and Manage Virtual Networking (25-30%)

### 4.1 Virtual Networks (VNet)

#### Concepts Fondamentaux
- **Address Space** : Plage CIDR privée 10.0.0.0/8 (16M d'adresses), 172.16.0.0/12 (1M d'adresses), 192.168.0.0/16 (65k adresses)
- **Subnets** : Subdivision du VNet
- **Network Security Groups** : Firewalls au niveau subnet/NIC

#### VNet Peering - Configuration Détaillée

**⚠️ Erreur Courante QCM : Configuration bidirectionnelle obligatoire**

**Vue d'ensemble :**
VNet Peering établit une connexion privée entre deux réseaux virtuels Azure, permettant aux ressources de communiquer comme si elles étaient sur le même réseau.

**Types de Peering :**

| Type | Portée | Latence | Coût | Use Case |
|------|--------|---------|------|----------|
| **Regional VNet Peering** | Même région Azure | Ultra-faible | Gratuit (ingress) | Apps multi-tier, partage de ressources |
| **Global VNet Peering** | Régions différentes | Faible | Facturé (ingress + egress) | Multi-région, DR, geo-distribution |

**Caractéristiques Clés :**
- **Traffic** : 100% privé, transit par backbone Azure (pas d'Internet)
- **Bande passante** : Identique à celle d'un VNet unique
- **Latence** : Minimale, équivalente à un VNet local
- **Sécurité** : Trafic chiffré par défaut sur le réseau Azure
- **Gateway Transit** : Partage de VPN/ExpressRoute Gateways possible

**Configuration Obligatoire - Bidirectionnelle**

**⚠️ POINT CRITIQUE pour l'EXAMEN :**
- **Peering = 2 opérations distinctes**
- VNet1 → VNet2 (créer peering depuis VNet1)
- VNet2 → VNet1 (créer peering depuis VNet2)
- **Les deux doivent être configurés** pour établir la communication

**Visualisation :**
```
VNet1 (10.0.0.0/16)          VNet2 (172.16.0.0/16)
       │                              │
       │ ─────── Peering 1 ────────→ │
       │                              │
       │ ←────── Peering 2 ───────── │
       │                              │
    ✅ Communication établie
```

**Sans configuration bidirectionnelle :**
```
VNet1 (10.0.0.0/16)          VNet2 (172.16.0.0/16)
       │                              │
       │ ─────── Peering 1 ────────→ │
       │                              │
       │         (pas de retour)      │
       │                              │
    ❌ Communication IMPOSSIBLE
```

**1. Règle d'Or : Plages d'adresses non-chevauchantes**

**⚠️ Prérequis OBLIGATOIRE :**
- **Principe** : Les plages d'adresses (CIDR) des deux VNets ne peuvent PAS se chevaucher
- **Vérification Azure** : Azure refuse automatiquement le peering si chevauchement détecté

**Exemples de Compatibilité :**

| VNet1 Address Space | VNet2 Address Space | Peering Possible ? | Raison |
|---------------------|---------------------|-------------------|--------|
| 10.0.0.0/16 | 172.16.0.0/16 | ✅ **OUI** | Plages complètement différentes |
| 10.0.0.0/16 | 10.1.0.0/16 | ✅ **OUI** | Pas de chevauchement |
| 192.168.0.0/24 | 192.168.1.0/24 | ✅ **OUI** | Sous-réseaux différents |
| 192.168.0.0/24 | 192.168.0.0/16 | ❌ **NON** | /24 inclus dans /16 |
| 10.0.0.0/16 | 10.0.0.0/24 | ❌ **NON** | /24 inclus dans /16 |
| 172.16.0.0/16 | 172.16.0.0/12 | ❌ **NON** | /16 inclus dans /12 |

**Erreur Fréquente en Examen :**
- **Question** : VNet1 (192.168.0.0/24) peut-il être peeré avec VNet3 (192.168.0.0/16) ?
- **Réponse correcte** : ❌ **NON**
- **Raison** : La plage 192.168.0.0/24 est **entièrement incluse** dans 192.168.0.0/16
- **Solution** : Utiliser des plages complètement distinctes (ex: 10.0.0.0/16 et 172.16.0.0/16)

**2. Configuration via Azure CLI - Étape par Étape**

**Étape 1 : Obtenir les Resource IDs**
```bash
# VNet1 ID
vnet1Id=$(az network vnet show \
  --resource-group RG1 \
  --name VNet1 \
  --query id --out tsv)

# VNet2 ID
vnet2Id=$(az network vnet show \
  --resource-group RG2 \
  --name VNet2 \
  --query id --out tsv)
```

**Étape 2 : Créer Peering VNet1 → VNet2**
```bash
az network vnet peering create \
  --name VNet1-to-VNet2 \
  --resource-group RG1 \
  --vnet-name VNet1 \
  --remote-vnet $vnet2Id \
  --allow-vnet-access
```

**Étape 3 : Créer Peering VNet2 → VNet1 (Obligatoire !)**
```bash
az network vnet peering create \
  --name VNet2-to-VNet1 \
  --resource-group RG2 \
  --vnet-name VNet2 \
  --remote-vnet $vnet1Id \
  --allow-vnet-access
```

**Étape 4 : Vérifier l'état du Peering**
```bash
# Vérifier depuis VNet1
az network vnet peering show \
  --resource-group RG1 \
  --vnet-name VNet1 \
  --name VNet1-to-VNet2 \
  --query peeringState

# Résultat attendu: "Connected"
```

**3. Configuration via PowerShell**

```powershell
# Créer Peering VNet1 → VNet2
Add-AzVirtualNetworkPeering `
  -Name "VNet1-to-VNet2" `
  -VirtualNetwork (Get-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "RG1") `
  -RemoteVirtualNetworkId "/subscriptions/{sub-id}/resourceGroups/RG2/providers/Microsoft.Network/virtualNetworks/VNet2"

# Créer Peering VNet2 → VNet1 (Obligatoire !)
Add-AzVirtualNetworkPeering `
  -Name "VNet2-to-VNet1" `
  -VirtualNetwork (Get-AzVirtualNetwork -Name "VNet2" -ResourceGroupName "RG2") `
  -RemoteVirtualNetworkId "/subscriptions/{sub-id}/resourceGroups/RG1/providers/Microsoft.Network/virtualNetworks/VNet1"

# Vérifier l'état
Get-AzVirtualNetworkPeering `
  -ResourceGroupName "RG1" `
  -VirtualNetworkName "VNet1" | 
  Select-Object Name, PeeringState
```

**4. Options de Configuration Avancées**

**Options Disponibles :**

| Option | Description | Use Case |
|--------|-------------|----------|
| **allow-vnet-access** | Autoriser trafic entre VNets | Par défaut, toujours activé |
| **allow-forwarded-traffic** | Autoriser trafic transitif via NVA | Hub-spoke avec firewall |
| **allow-gateway-transit** | Partager VPN/ExpressRoute Gateway | VNet hub avec gateway |
| **use-remote-gateways** | Utiliser gateway du VNet distant | VNet spoke sans gateway |

**Exemple - Configuration Hub-Spoke :**
```bash
# Hub VNet (avec VPN Gateway)
az network vnet peering create \
  --name Hub-to-Spoke \
  --resource-group HubRG \
  --vnet-name HubVNet \
  --remote-vnet $spokeVnetId \
  --allow-vnet-access \
  --allow-gateway-transit

# Spoke VNet (utilise gateway du Hub)
az network vnet peering create \
  --name Spoke-to-Hub \
  --resource-group SpokeRG \
  --vnet-name SpokeVNet \
  --remote-vnet $hubVnetId \
  --allow-vnet-access \
  --use-remote-gateways
```

**5. Caractéristiques Non-Transitives**

**⚠️ Important pour l'Examen :**
VNet Peering est **NON-TRANSITIF** par défaut

**Scénario :**
```
VNet A ←→ VNet B ←→ VNet C
```
- VNet A peut communiquer avec VNet B ✅
- VNet B peut communiquer avec VNet C ✅
- VNet A **NE PEUT PAS** communiquer avec VNet C ❌

**Solution pour rendre transitif :**
- Créer un peering direct A ←→ C
- OU utiliser une architecture Hub-Spoke avec **allow-forwarded-traffic** et une appliance réseau (NVA) dans le hub

**6. Performance et Latence**
- **Bande passante** : Identique à celle d'un VNet local (pas de limite imposée par le peering)
- **Latence** : Ultra-faible (< 1ms en regional, quelques ms en global)
- **Throughput** : Dépend uniquement des VM sizes
- **Coût Regional** : Ingress gratuit, Egress gratuit dans la même région
- **Coût Global** : Ingress et Egress facturés ($0.01-0.035/GB selon zones)

**7. Troubleshooting VNet Peering**

**Problèmes Courants :**

| Problème | Cause | Solution |
|----------|-------|----------|
| Peering en état "Initiated" | Peering bidirectionnel incomplet | Créer le peering retour |
| Communication impossible | NSG bloque le trafic | Vérifier règles NSG avec IP Flow Verify |
| Peering refusé | Adresses IP chevauchantes | Modifier les plages d'adresses |
| Gateway transit ne fonctionne pas | Options mal configurées | Vérifier allow-gateway-transit et use-remote-gateways |

**Commandes de Diagnostic :**
```bash
# Vérifier l'état du peering
az network vnet peering list \
  --resource-group myRG \
  --vnet-name myVNet \
  --output table

# Vérifier les plages d'adresses
az network vnet show \
  --resource-group myRG \
  --name myVNet \
  --query addressSpace.addressPrefixes

# Tester la connectivité (depuis une VM)
ping <private-ip-remote-vm>
```

**8. Configuration dans le Portail Azure**
- **Navigation** : Virtual Network → Peerings → + Add
- **Bidirectionnel** : Cocher "Configure peering settings on both VNets"
- **Validation automatique** : Azure vérifie la compatibilité des plages
- **État** : Doit afficher "Connected" des deux côtés

**9. Best Practices**

✅ **À FAIRE :**
- Planifier les plages d'adresses dès le début (éviter chevauchements)
- Toujours configurer le peering dans les deux sens
- Utiliser des conventions de nommage claires (VNet1-to-VNet2)
- Documenter la topologie réseau
- Tester la connectivité après création

❌ **À ÉVITER :**
- Utiliser des plages d'adresses chevauchantes
- Oublier le peering bidirectionnel
- Compter sur la transitivité par défaut
- Ignorer les NSG qui peuvent bloquer le trafic

#### DNS Resolution

** Point identifié :** DNS interne Azure
- **Format automatique** : `vm-name.internal.cloudapp.net`
- **Usage** : Résolution entre VMs dans VNet
- **Custom DNS** : Possibilité d'utiliser ses propres serveurs

### 4.1.1 Azure Private DNS Zones

#### Concepts et Fonctionnalités

**Azure Private DNS Zone**
- **Usage** : Résolution DNS privée pour ressources Azure dans un VNet
- **Avantages** : 
  - Noms personnalisés pour ressources internes
  - Résolution entre VNets connectés
  - Pas besoin de serveurs DNS personnalisés
- **Domaines** : Domaines personnalisés (ex: contoso.internal)

#### Virtual Network Links

**Processus d'association identifié :**
Pour associer un VNet à une Private DNS Zone, vous devez :
1. **Créer un Virtual Network Link** dans la zone DNS privée
2. **Sélectionner le VNet** à associer
3. **Configurer l'auto-registration** (optionnel)
4. **Valider** l'association

**Caractéristiques des Virtual Network Links :**
- **Fonction** : Lie un VNet à une zone DNS privée
- **Résolution** : Les VMs du VNet peuvent résoudre les enregistrements de la zone
- **Auto-registration** : Enregistrement automatique des VMs dans la zone DNS
- **Bi-directionnel** : Un VNet peut être lié à plusieurs zones DNS

#### Azure DNS Private Resolver

**Rôle et Utilisation :**
- **Fonction principale** : Proxy pour requêtes DNS entre environnements on-premises et Azure DNS
- **Scénario** : Connectivité hybride (on-premises <--> Azure)
- **Avantages** :
  - Résolution DNS bidirectionnelle
  - Intégration transparente avec Private DNS Zones
  - Pas de gestion de serveurs DNS
- **Architecture** : Service géré déployé dans un VNet

**Comparaison des Solutions DNS :**

**Virtual Network Link (Recommandé pour Azure-only)**
- **Usage** : Association VNet --> Private DNS Zone
- **Scope** : Azure uniquement
- **Configuration** : Simple, intégré
- **Coût** : Inclus avec Private DNS

**Azure DNS Private Resolver (Hybride)**
- **Usage** : Proxy DNS on-premises <--> Azure
- **Scope** : Environnements hybrides
- **Configuration** : Déploiement dans VNet requis
- **Coût** : Service facturé séparément

**Custom DNS Server (Non recommandé pour Private DNS)**
- **Usage** : Serveurs DNS personnalisés (VM ou appliance)
- **Limitation** : Configuration complexe, ne fonctionne pas nativement avec Private DNS Zones
- **Maintenance** : Gestion manuelle requise
- **Coût** : Infrastructure + maintenance

#### User-Defined Routes (UDR)
** Cas d'usage identifié :** Redirection de trafic vers appliances réseau
- **Objectif** : Forcer le trafic à passer par des appliances spécifiques (firewalls, appliances d'inspection)
- **Mécanisme** : Azure crée automatiquement une table de routage pour chaque sous-réseau avec des routes système par défaut
- **Override** : Les UDR permettent de remplacer certaines routes système Azure
- **Application** : Le trafic sortant d'un sous-réseau suit les routes de la table de routage du sous-réseau

** Erreur fréquente identifiée :** Confusion entre routes système et UDR
- ** Erreur** : Essayer de modifier les routes système par défaut
- ** Correct** : Créer des User-Defined Routes pour rediriger le trafic
- **Principe** : Les routes système sont gérées par Azure, les UDR permettent de surcharger le comportement

### 4.2 Network Security Groups (NSG)
Can be used with subnet or NIC

#### Règles de Sécurité
- **Priority** : 100-4096, plus bas = plus prioritaire
- **Direction** : Inbound, Outbound
- **Action** : Allow, Deny

** Optimisation identifiée :**
- **Un NSG peut être associé à plusieurs ressources**
- **5 VMs avec mêmes règles = 5 NICs + 1 NSG**
- Partage possible entre subnets et NICs

**Tips critiques identifiés :**

**1. Système de Priorités NSG**
- **Règle fondamentale** : Plus le numéro de priorité est bas, plus la règle est prioritaire
- **Exemple critique** : Priority 100 > Priority 200 (100 est plus prioritaire)
- **Impact** : Une règle Deny avec priorité élevée (100) bloque une règle Allow avec priorité faible (200)
- **Solution** : Ajuster les priorités ou modifier l'action de la règle

**2. Évaluation en Cascade : Subnet → NIC**
- **Ordre d'évaluation** : NSG du Subnet d'abord, puis NSG de la NIC
- **Principe** : Les deux niveaux doivent autoriser le trafic pour qu'il passe
- **Piège courant** : NSG Subnet Allow + NSG NIC Deny = Trafic bloqué
- **Optimisation** : Un seul NSG Deny à n'importe quel niveau bloque tout le trafic

**3. Stratégies de Résolution de Problèmes**
- **Diagnostic** : Vérifier les NSG aux deux niveaux (Subnet et NIC)
- **Priorités** : Identifier les règles conflictuelles par numéro de priorité
- **Actions** : Modifier la priorité OU changer l'action (Allow/Deny)
- **Test** : Utiliser Network Watcher pour valider les règles

#### Default Rules
**Inbound :**
- Allow VNet traffic
- Allow Azure Load Balancer
- Deny all other traffic

**Outbound :**
- Allow VNet traffic
- Allow Internet traffic
- Deny all other traffic

### 4.3 Load Balancing Solutions

#### Azure Load Balancer (Layer 4) - Guide DevOps Approfondi

**Architecture et Composants :**

**Types de Load Balancer :**
1. **Public Load Balancer**
   - **Frontend** : Adresse IP publique
   - **Usage** : Trafic Internet → VMs backend
   - **Cas d'usage** : Applications web, APIs publiques
   - **NAT** : Traduit IP publique → IP privées backend

2. **Internal Load Balancer (ILB)**
   - **Frontend** : Adresse IP privée du VNet
   - **Usage** : Trafic interne entre tiers applicatifs
   - **Cas d'usage** : App tier → Database tier, microservices
   - **Sécurité** : Jamais exposé à Internet

**SKUs - Différences Critiques :**

| Caractéristique | Basic | Standard |
|-----------------|-------|----------|
| **Backend pool size** | 300 VMs | 1000 VMs |
| **Health probes** | TCP, HTTP | TCP, HTTP, HTTPS |
| **Availability Zones** | ❌ Non | ✅ Oui |
| **SLA** | Aucun | 99.99% |
| **HA Ports** | ❌ Non | ✅ Oui |
| **Outbound rules** | Basiques | Avancées |
| **Sécurité** | Open par défaut | Fermé par défaut |
| **Coût** | Gratuit | ~$18/mois + data |
| **Production** | ⚠️ Non recommandé | ✅ Recommandé |

**⚠️ IMPORTANT DevOps** : Basic SKU sera déprécié en 2025. Toujours utiliser Standard SKU.

**Composants Techniques Détaillés :**

**1. Frontend IP Configuration**
```bash
# Public Load Balancer
az network lb create \
  --resource-group myRG \
  --name myPublicLB \
  --sku Standard \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontend

# Internal Load Balancer
az network lb create \
  --resource-group myRG \
  --name myInternalLB \
  --sku Standard \
  --vnet-name myVNet \
  --subnet mySubnet \
  --frontend-ip-name myFrontend \
  --private-ip-address 10.0.1.10
```

**2. Backend Pool - Options de Configuration**

**A. VM-based Backend Pool**
```bash
# Ajouter des VMs au backend pool
az network nic ip-config address-pool add \
  --resource-group myRG \
  --nic-name myNIC \
  --ip-config-name ipconfig1 \
  --lb-name myLB \
  --address-pool myBackendPool
```

**B. IP-based Backend Pool (Standard SKU only)**
```bash
# Créer backend pool avec IP addresses
az network lb address-pool create \
  --resource-group myRG \
  --lb-name myLB \
  --name myBackendPool

# Ajouter une IP au pool
az network lb address-pool address add \
  --resource-group myRG \
  --lb-name myLB \
  --pool-name myBackendPool \
  --name backend1 \
  --ip-address 10.0.1.4
```

**C. VMSS Backend Pool (Auto-scaling)**
```bash
# Associer VMSS au load balancer
az vmss create \
  --resource-group myRG \
  --name myVMSS \
  --image UbuntuLTS \
  --load-balancer myLB \
  --backend-pool-name myBackendPool
```

**3. Health Probes - Configuration Avancée**

**Types de Probes :**

**TCP Probe (Recommandé pour bases de données)**
```bash
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name tcpProbe \
  --protocol tcp \
  --port 1433 \
  --interval 15 \
  --threshold 2
```
- **Fonctionnement** : Vérifie si le port répond (TCP handshake)
- **Avantage** : Léger, rapide
- **Inconvénient** : Ne vérifie pas si l'application fonctionne

**HTTP/HTTPS Probe (Recommandé pour web apps)**
```bash
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name httpProbe \
  --protocol http \
  --port 80 \
  --path /health \
  --interval 15 \
  --threshold 2
```
- **Fonctionnement** : Attend HTTP 200 OK
- **Avantage** : Vérifie la santé applicative
- **Best practice** : Créer un endpoint `/health` ou `/healthz` dédié

**Endpoint Health Check - Pattern DevOps**
```python
# Flask example
@app.route('/health')
def health_check():
    try:
        # Vérifier DB connectivity
        db.session.execute('SELECT 1')
        # Vérifier dependencies
        redis_client.ping()
        return jsonify({"status": "healthy"}), 200
    except Exception as e:
        return jsonify({"status": "unhealthy", "error": str(e)}), 503
```

**Paramètres de Health Probe - Tuning Performance**
- **Interval** : Fréquence des checks (5-300 sec, défaut: 15)
- **Threshold** : Nombre d'échecs avant marquage unhealthy (défaut: 2)
- **Calcul downtime** : `interval × threshold = temps avant exclusion`
  - Exemple : 15s × 2 = 30 secondes avant que VM soit retirée
- **Trade-off** : Interval court = détection rapide mais plus de charge

**4. Load Balancing Rules - Distribution du Trafic**

**Rule Basique**
```bash
az network lb rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myHTTPRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontend \
  --backend-pool-name myBackendPool \
  --probe-name httpProbe \
  --disable-outbound-snat false \
  --idle-timeout 15 \
  --enable-tcp-reset true
```

**Distribution Algorithms (Session Affinity)**

**A. 5-Tuple Hash (Défaut) - Aucune session persistence**
```
Hash = (Source IP, Source Port, Dest IP, Dest Port, Protocol)
```
- **Comportement** : Distribution équitable
- **Usage** : Applications stateless
- **Avantage** : Meilleure répartition de charge

**B. 3-Tuple Hash (Source IP affinity)**
```
Hash = (Source IP, Dest IP, Protocol)
```
```bash
az network lb rule update \
  --resource-group myRG \
  --lb-name myLB \
  --name myHTTPRule \
  --load-distribution SourceIP
```
- **Comportement** : Même client → même backend
- **Usage** : Applications avec état (sessions)
- **Durée** : Pendant la durée de la session TCP

**C. 2-Tuple Hash (Source IP + Protocol affinity)**
```
Hash = (Source IP, Protocol)
```
```bash
az network lb rule update \
  --resource-group myRG \
  --lb-name myLB \
  --name myHTTPRule \
  --load-distribution SourceIPProtocol
```
- **Comportement** : Client → toujours même backend pour ce protocole
- **Usage** : FTP, RDP, applications legacy

**Pattern DevOps - Choix de Distribution**
| Application Type | Distribution | Raison |
|------------------|--------------|--------|
| **REST API stateless** | 5-tuple (défaut) | Pas de session, max performance |
| **E-commerce avec panier** | 3-tuple (SourceIP) | Maintenir sessions utilisateur |
| **WebSockets** | 3-tuple (SourceIP) | Connexion persistante |
| **Remote Desktop** | 2-tuple (SourceIPProtocol) | Session RDP stable |
| **Microservices** | 5-tuple (défaut) | Stateless, scalabilité max |

**5. HA Ports (Standard SKU Only) - Pattern Avancé**

**Configuration**
```bash
az network lb rule create \
  --resource-group myRG \
  --lb-name myInternalLB \
  --name HAPortsRule \
  --protocol All \
  --frontend-port 0 \
  --backend-port 0 \
  --frontend-ip-name myFrontend \
  --backend-pool-name myBackendPool
```

**Cas d'usage DevOps**
- **Network Virtual Appliances (NVA)** : Firewalls, IDS/IPS
- **Architecture Hub-Spoke** : Load balancing de tous les flux
- **Port forwarding dynamique** : Pas besoin de règles par port

**6. Outbound Rules - Contrôle du Trafic Sortant**

**Problème** : Par défaut, VMs derrière ILB ne peuvent pas accéder à Internet

**Solution A : Outbound Rule avec Public LB**
```bash
az network lb outbound-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myOutboundRule \
  --frontend-ip-configs myFrontend \
  --protocol All \
  --idle-timeout 15 \
  --outbound-ports 10000 \
  --address-pool myBackendPool
```

**Solution B : NAT Gateway (Recommandé pour production)**
```bash
az network nat gateway create \
  --resource-group myRG \
  --name myNATGateway \
  --public-ip-addresses myPublicIP \
  --idle-timeout 10

az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name mySubnet \
  --nat-gateway myNATGateway
```

**7. Inbound NAT Rules - Accès Direct aux VMs**

**Use Case** : SSH/RDP vers VMs spécifiques derrière LB
```bash
# Port 2221 → VM1:22
az network lb inbound-nat-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name SSH-VM1 \
  --protocol tcp \
  --frontend-port 2221 \
  --backend-port 22 \
  --frontend-ip-name myFrontend

# Port 2222 → VM2:22
az network lb inbound-nat-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name SSH-VM2 \
  --protocol tcp \
  --frontend-port 2222 \
  --backend-port 22 \
  --frontend-ip-name myFrontend

# Associer à la NIC
az network nic ip-config inbound-nat-rule add \
  --resource-group myRG \
  --nic-name myVM1-NIC \
  --ip-config-name ipconfig1 \
  --inbound-nat-rule SSH-VM1
```

**Pattern DevOps - Bastion Alternative**
```
Internet → LB Public IP:2221 → VM1:22
Internet → LB Public IP:2222 → VM2:22
Internet → LB Public IP:2223 → VM3:22
```

#### Troubleshooting Load Balancer - Standard SKU

**Problèmes de connectivité courants et résolutions :**

**1. Health Probe Configuration (Priorité haute)**
- **Symptôme** : VMs ne reçoivent pas de trafic
- **Cause** : Health probes mal configurés ou VMs ne répondent pas
- **Solution** : 
  - Vérifier la configuration des health probes (protocole, port, chemin)
  - S'assurer que les VMs répondent sur le port configuré
  - Tester manuellement le endpoint de health check
- **Impact** : Si les probes échouent, le Load Balancer n'envoie pas de trafic

**2. Port Response Verification (Priorité haute)**
- **Symptôme** : Trafic n'atteint pas les backends
- **Cause** : Application non démarrée ou port non écouté
- **Solution** :
  - Vérifier que l'application écoute sur le port configuré
  - Utiliser `netstat -an` pour confirmer les ports d'écoute
  - Redémarrer les services si nécessaire
- **Impact** : Health probes échouent si le port ne répond pas

**3. NSG Rules Validation (Priorité haute)**
- **Symptôme** : Connectivité bloquée malgré Load Balancer configuré
- **Cause** : NSG bloque le trafic inbound vers les VMs
- **Solution** :
  - Vérifier les règles NSG au niveau subnet ET NIC
  - S'assurer que les règles autorisent le trafic sur les ports requis
  - Autoriser le trafic depuis AzureLoadBalancer service tag
- **Impact** : NSG peut bloquer même si Load Balancer est correct

**Actions NON recommandées (Erreurs courantes) :**
- **Modifier session persistence** : Ne résout pas les problèmes de connectivité de base
- **Augmenter timeout settings** : N'aide pas si le trafic est bloqué
- **Redémarrer les VMs** : Peut masquer temporairement sans résoudre la cause

** Stratégie de troubleshooting identifiée :**
1. **Health probes** : Vérifier configuration et réponses
2. **Port listening** : Confirmer que les applications écoutent
3. **NSG rules** : Valider les autorisations de trafic
4. **Network Watcher** : Utiliser IP Flow Verify pour diagnostic approfondi

** Tips critiques identifiés :**

**1. Session Persistence (Sticky Sessions) - Concept Clé**
- **Problème résolu** : Maintenir l'utilisateur sur le même serveur backend
- **Cas d'usage critique** : Applications avec état (paniers e-commerce, sessions utilisateur)
- **Configuration** : Client IP + Protocol pour une persistance optimale
- **Alternative** : None = distribution aléatoire (pas de persistance)

**2. Différenciation des Options de Load Balancer**
- **Session Persistence** : Contrôle la distribution des sessions utilisateur
- **NAT Rules** : Redirection de trafic spécifique (différent de la session persistence)
- **Health Probes** : Vérification de l'état des backends
- **Load Balancing Rules** : Définition des pools et méthodes de distribution

**3. Stratégies de Configuration**
- **Client IP** : Persistance basée sur l'adresse IP source
- **Protocol** : Persistance basée sur le protocole (HTTP/HTTPS)
- **Combinaison** : Client IP + Protocol pour une persistance maximale
- **Performance** : Équilibrer entre persistance et répartition de charge

#### Application Gateway (Layer 7) - Guide DevOps Approfondi

**Architecture et Composants :**

Application Gateway est un **reverse proxy intelligent** opérant au Layer 7 (HTTP/HTTPS).

**SKUs et Versions :**

| Caractéristique | v1 Standard | v1 WAF | v2 Standard | v2 WAF |
|-----------------|-------------|--------|-------------|--------|
| **Autoscaling** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Availability Zones** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Static VIP** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **WAF** | ❌ Non | ✅ Oui | ❌ Non | ✅ Oui |
| **Performance** | 1000 Mbps | 1000 Mbps | Supérieure | Supérieure |
| **Rewrite HTTP** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Custom Error Pages** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Status** | ⚠️ Legacy | ⚠️ Legacy | ✅ Recommandé | ✅ Recommandé |
| **Coût/mois (estimé)** | ~$125 | ~$250 | ~$200+ | ~$300+ |

**⚠️ IMPORTANT DevOps** : v1 est en maintenance. Toujours déployer v2 pour nouvelles installations.

**Composants de l'Application Gateway :**

**1. Frontend IP Configuration**
- **Public IP** : Exposition Internet (obligatoire pour v2)
- **Private IP** : Exposition interne VNet (optionnel)
- **Both** : Combinaison public + private possible

```bash
# Créer Application Gateway v2 avec autoscaling
az network application-gateway create \
  --name myAppGateway \
  --resource-group myRG \
  --sku WAF_v2 \
  --capacity 2 \
  --min-capacity 2 \
  --max-capacity 10 \
  --vnet-name myVNet \
  --subnet appGatewaySubnet \
  --public-ip-address myPublicIP \
  --http-settings-cookie-based-affinity Enabled \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --priority 100
```

**2. Backend Pools - Multi-type Support**

Application Gateway peut cibler :
- **VMs** : Machines virtuelles Azure
- **VMSS** : VM Scale Sets
- **App Services** : Azure Web Apps
- **IP/FQDN** : Serveurs on-premises ou autres clouds
- **Private Link** : Services privés Azure

```bash
# Backend pool avec plusieurs types
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myBackendPool \
  --servers 10.0.1.4 10.0.1.5 webapp.azurewebsites.net
```

**3. HTTP Settings - Configuration Backend**

```bash
az network application-gateway http-settings create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSettings \
  --port 80 \
  --protocol Http \
  --cookie-based-affinity Enabled \
  --timeout 30 \
  --probe myHealthProbe \
  --host-name-from-backend-pool false \
  --host-name api.internal.com
```

**Paramètres clés :**
- **Cookie-based affinity** : Session persistence (cookie ApplicationGatewayAffinity)
- **Connection draining** : Termine proprement les connexions lors de retrait backend
- **Timeout** : 1-86400 secondes (défaut: 30)
- **Override backend hostname** : Réécrit le header Host

**4. Health Probes - Monitoring Avancé**

**Probe Custom HTTP**
```bash
az network application-gateway probe create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHealthProbe \
  --protocol Http \
  --host-name-from-http-settings true \
  --path /api/health \
  --interval 30 \
  --timeout 30 \
  --threshold 3 \
  --match-status-codes 200-399
```

**Pattern DevOps - Health Endpoint Avancé**
```javascript
// Node.js/Express example
app.get('/api/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    checks: {}
  };
  
  try {
    // Database check
    await db.query('SELECT 1');
    health.checks.database = 'ok';
    
    // Redis check
    await redis.ping();
    health.checks.cache = 'ok';
    
    // External API check
    const apiHealth = await fetch('https://api.external.com/health');
    health.checks.externalAPI = apiHealth.ok ? 'ok' : 'degraded';
    
    res.status(200).json(health);
  } catch (error) {
    health.status = 'unhealthy';
    health.error = error.message;
    res.status(503).json(health);
  }
});
```

**5. Listeners (HTTP/HTTPS)**

**HTTP Listener Basique**
```bash
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPListener \
  --frontend-port appGatewayFrontendPort \
  --frontend-ip appGatewayFrontendIP \
  --host-name www.contoso.com
```

**HTTPS Listener avec SSL Certificate**
```bash
# Upload SSL certificate
az network application-gateway ssl-cert create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myCert \
  --cert-file /path/to/cert.pfx \
  --cert-password "P@ssw0rd"

# Créer HTTPS listener
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSListener \
  --frontend-port 443 \
  --frontend-ip appGatewayFrontendIP \
  --ssl-cert myCert \
  --host-name www.contoso.com
```

**Multi-site Listener (Plusieurs domaines)**
```bash
# Listener pour site 1
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name site1Listener \
  --frontend-port 443 \
  --ssl-cert cert1 \
  --host-names www.site1.com site1.com

# Listener pour site 2
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name site2Listener \
  --frontend-port 443 \
  --ssl-cert cert2 \
  --host-names www.site2.com site2.com
```

**6. Routing Rules - Logique de Distribution**

**A. Basic Routing Rule**
```bash
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name rule1 \
  --http-listener myHTTPListener \
  --rule-type Basic \
  --address-pool myBackendPool \
  --http-settings myHTTPSettings \
  --priority 100
```

**B. Path-based Routing (URL Routing)**
```bash
# Créer URL path map
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myPathMap \
  --paths /api/* \
  --address-pool apiBackendPool \
  --http-settings apiHTTPSettings \
  --default-address-pool defaultBackendPool \
  --default-http-settings defaultHTTPSettings

# Path rule pour /images
az network application-gateway url-path-map rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --path-map-name myPathMap \
  --name imagesRule \
  --paths /images/* \
  --address-pool imagesBackendPool \
  --http-settings imagesHTTPSettings

# Associer à routing rule
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name pathRoutingRule \
  --http-listener myListener \
  --rule-type PathBasedRouting \
  --url-path-map myPathMap \
  --priority 200
```

**Architecture Path-based Routing :**
```
www.contoso.com
├── /api/*          → API Backend Pool (port 8080)
├── /images/*       → CDN/Storage Backend Pool
├── /admin/*        → Admin Backend Pool (port 9000)
└── /*              → Web Frontend Backend Pool (port 80)
```

**7. SSL/TLS Configuration - Patterns DevOps**

**A. SSL Termination (Déchiffrement au Gateway)**
```
Client (HTTPS) → App Gateway (déchiffre) → Backend (HTTP)
```
- **Avantage** : Offload SSL processing des backends
- **Usage** : Applications internes, microservices
- **Coût CPU** : Gateway gère le chiffrement

**B. End-to-End SSL (SSL de bout en bout)**
```
Client (HTTPS) → App Gateway (HTTPS) → Backend (HTTPS)
```
```bash
az network application-gateway http-settings update \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSettings \
  --protocol Https \
  --port 443 \
  --trusted-root-certificates backendCert
```
- **Avantage** : Sécurité maximale
- **Usage** : Conformité, données sensibles
- **Requis** : Certificats SSL sur backends

**C. SSL Policy Configuration**
```bash
# Politique SSL moderne (TLS 1.2+)
az network application-gateway ssl-policy set \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --min-protocol-version TLSv1_2 \
  --cipher-suites TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 \
                 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

**8. Web Application Firewall (WAF) - Sécurité Avancée**

**WAF Modes :**
- **Detection** : Log les attaques, ne bloque pas
- **Prevention** : Bloque les attaques détectées

```bash
# Activer WAF en mode Prevention
az network application-gateway waf-config set \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-type OWASP \
  --rule-set-version 3.2
```

**OWASP Top 10 Protection :**
1. **Injection** (SQL, NoSQL, LDAP)
2. **Broken Authentication**
3. **Sensitive Data Exposure**
4. **XML External Entities (XXE)**
5. **Broken Access Control**
6. **Security Misconfiguration**
7. **Cross-Site Scripting (XSS)**
8. **Insecure Deserialization**
9. **Using Components with Known Vulnerabilities**
10. **Insufficient Logging & Monitoring**

**Custom WAF Rules**
```bash
# Bloquer une IP spécifique
az network application-gateway waf-policy custom-rule create \
  --policy-name myWAFPolicy \
  --resource-group myRG \
  --name blockIP \
  --priority 100 \
  --rule-type MatchRule \
  --action Block \
  --match-conditions RemoteAddr IPMatch 192.168.1.100

# Rate limiting (DDoS protection)
az network application-gateway waf-policy custom-rule create \
  --policy-name myWAFPolicy \
  --resource-group myRG \
  --name rateLimitRule \
  --priority 200 \
  --rule-type RateLimitRule \
  --action Block \
  --rate-limit-duration OneMin \
  --rate-limit-threshold 100
```

**9. Rewrite Rules (v2 only) - Manipulation HTTP**

**A. Rewrite Headers**
```bash
# Rewrite Set
az network application-gateway rewrite-rule set create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myRewriteSet

# Ajouter X-Forwarded-For
az network application-gateway rewrite-rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --rule-set-name myRewriteSet \
  --name addXForwardedFor \
  --sequence 100 \
  --request-headers X-Forwarded-For="{var_client_ip}"

# Supprimer header sensible
az network application-gateway rewrite-rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --rule-set-name myRewriteSet \
  --name removeServerHeader \
  --sequence 200 \
  --response-headers Server=""
```

**B. URL Rewrite**
```bash
# Rediriger /old vers /new
az network application-gateway rewrite-rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --rule-set-name myRewriteSet \
  --name urlRewrite \
  --sequence 300 \
  --request-headers Location="/new" \
  --conditions http_req_url Equal "/old"
```

**Use Cases DevOps :**
- **Ajouter headers de sécurité** : HSTS, X-Frame-Options, CSP
- **Client IP forwarding** : X-Forwarded-For pour les backends
- **Masquer technologies** : Supprimer Server, X-Powered-By
- **URL canonicalization** : Normaliser les URLs

**10. Autoscaling (v2 only)**

```bash
az network application-gateway update \
  --name myAppGateway \
  --resource-group myRG \
  --min-capacity 2 \
  --max-capacity 10
```

**Calcul de capacité :**
- **1 Capacity Unit** = 2500 connexions/sec persistantes
- **Scaling** : Basé sur CPU, connexions, throughput
- **Temps** : ~6-7 minutes pour scale out/in
- **Coût** : Pay-per-capacity-unit-hour

**11. Monitoring et Diagnostics - DevOps Essentials**

**Métriques Clés à Monitorer :**
```bash
# CPU Utilization
az monitor metrics list \
  --resource myAppGateway \
  --resource-group myRG \
  --resource-type Microsoft.Network/applicationGateways \
  --metric "CpuUtilization"

# Response time
az monitor metrics list \
  --resource myAppGateway \
  --resource-group myRG \
  --metric "ApplicationGatewayTotalTime"

# Failed requests
az monitor metrics list \
  --resource myAppGateway \
  --resource-group myRG \
  --metric "FailedRequests"
```

**Logs Diagnostiques :**
```bash
# Activer logs
az monitor diagnostic-settings create \
  --name myDiagnostics \
  --resource myAppGatewayId \
  --logs '[
    {
      "category": "ApplicationGatewayAccessLog",
      "enabled": true
    },
    {
      "category": "ApplicationGatewayPerformanceLog",
      "enabled": true
    },
    {
      "category": "ApplicationGatewayFirewallLog",
      "enabled": true
    }
  ]' \
  --workspace myLogAnalyticsWorkspace
```

**KQL Queries pour Troubleshooting :**
```kql
// Top 10 erreurs 5xx
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where httpStatus_d >= 500
| summarize count() by httpStatus_d, requestUri_s
| top 10 by count_

// Requêtes lentes (>3 secondes)
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where timeTaken_d > 3000
| project TimeGenerated, requestUri_s, timeTaken_d, clientIP_s
| order by timeTaken_d desc

// WAF blocks
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where action_s == "Blocked"
| summarize count() by ruleId_s, Message
| order by count_ desc
```

**12. Architecture Patterns DevOps**

**Pattern A : Multi-Region avec Traffic Manager**
```
Traffic Manager (DNS)
├── Region 1: App Gateway → Backend Pool 1
└── Region 2: App Gateway → Backend Pool 2
```

**Pattern B : Microservices Routing**
```
App Gateway
├── /user-service/*     → User Microservice Pool
├── /order-service/*    → Order Microservice Pool
├── /payment-service/*  → Payment Microservice Pool
└── /static/*           → CDN/Storage
```

**Pattern C : Blue/Green Deployment**
```bash
# Backend pools
- bluePool (v1.0 - production actuelle)
- greenPool (v2.0 - nouvelle version)

# Étape 1: Déployer v2.0 dans greenPool
# Étape 2: Tester greenPool (health probes)
# Étape 3: Switcher routing rule: bluePool → greenPool
# Étape 4: Monitorer
# Étape 5: Rollback si problème (greenPool → bluePool)
```

**Pattern D : Canary Deployment (avec Path Rules)**
```bash
# 95% trafic → stable backend
# 5% trafic → canary backend

# Utiliser weighted routing ou custom headers
X-Canary-User: true → Canary Backend
```

**13. Infrastructure as Code - Terraform Examples**

**A. Load Balancer Standard avec Backend Pool**
```hcl
# Public IP
resource "azurerm_public_ip" "lb" {
  name                = "lb-public-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Load Balancer
resource "azurerm_lb" "main" {
  name                = "main-lb"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "Standard"

  frontend_ip_configuration {
    name                 = "PublicIPAddress"
    public_ip_address_id = azurerm_public_ip.lb.id
  }
}

# Backend Pool
resource "azurerm_lb_backend_address_pool" "main" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "backend-pool"
}

# Health Probe
resource "azurerm_lb_probe" "http" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "http-probe"
  protocol        = "Http"
  request_path    = "/health"
  port            = 80
  interval_in_seconds = 15
  number_of_probes    = 2
}

# Load Balancing Rule
resource "azurerm_lb_rule" "http" {
  loadbalancer_id                = azurerm_lb.main.id
  name                           = "http-rule"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 80
  frontend_ip_configuration_name = "PublicIPAddress"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.main.id]
  probe_id                       = azurerm_lb_probe.http.id
  load_distribution              = "SourceIP"
  idle_timeout_in_minutes        = 15
  enable_tcp_reset               = true
}

# NAT Rule pour SSH
resource "azurerm_lb_nat_rule" "ssh" {
  resource_group_name            = azurerm_resource_group.main.name
  loadbalancer_id                = azurerm_lb.main.id
  name                           = "ssh-vm1"
  protocol                       = "Tcp"
  frontend_port                  = 2221
  backend_port                   = 22
  frontend_ip_configuration_name = "PublicIPAddress"
}
```

**B. Application Gateway v2 avec WAF**
```hcl
# Public IP
resource "azurerm_public_ip" "appgw" {
  name                = "appgw-public-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Application Gateway
resource "azurerm_application_gateway" "main" {
  name                = "main-appgateway"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  sku {
    name     = "WAF_v2"
    tier     = "WAF_v2"
  }

  autoscale_configuration {
    min_capacity = 2
    max_capacity = 10
  }

  gateway_ip_configuration {
    name      = "gateway-ip-config"
    subnet_id = azurerm_subnet.appgw.id
  }

  frontend_port {
    name = "https-port"
    port = 443
  }

  frontend_port {
    name = "http-port"
    port = 80
  }

  frontend_ip_configuration {
    name                 = "frontend-ip"
    public_ip_address_id = azurerm_public_ip.appgw.id
  }

  backend_address_pool {
    name = "api-backend-pool"
  }

  backend_address_pool {
    name = "web-backend-pool"
  }

  backend_http_settings {
    name                  = "https-settings"
    cookie_based_affinity = "Enabled"
    port                  = 443
    protocol              = "Https"
    request_timeout       = 30
    probe_name            = "api-probe"
    
    connection_draining {
      enabled           = true
      drain_timeout_sec = 60
    }
  }

  http_listener {
    name                           = "https-listener"
    frontend_ip_configuration_name = "frontend-ip"
    frontend_port_name             = "https-port"
    protocol                       = "Https"
    ssl_certificate_name           = "ssl-cert"
    host_name                      = "api.contoso.com"
  }

  # URL Path Map pour microservices
  url_path_map {
    name                               = "api-path-map"
    default_backend_address_pool_name  = "web-backend-pool"
    default_backend_http_settings_name = "https-settings"

    path_rule {
      name                       = "api-rule"
      paths                      = ["/api/*"]
      backend_address_pool_name  = "api-backend-pool"
      backend_http_settings_name = "https-settings"
    }
  }

  request_routing_rule {
    name                       = "api-routing-rule"
    rule_type                  = "PathBasedRouting"
    http_listener_name         = "https-listener"
    url_path_map_name          = "api-path-map"
    priority                   = 100
  }

  ssl_certificate {
    name     = "ssl-cert"
    data     = filebase64("certificate.pfx")
    password = var.ssl_password
  }

  ssl_policy {
    policy_type = "Predefined"
    policy_name = "AppGwSslPolicy20220101"
  }

  # Health Probe
  probe {
    name                = "api-probe"
    protocol            = "Https"
    path                = "/api/health"
    interval            = 30
    timeout             = 30
    unhealthy_threshold = 3
    host                = "api.contoso.com"
    
    match {
      status_code = ["200-399"]
    }
  }

  # WAF Configuration
  waf_configuration {
    enabled          = true
    firewall_mode    = "Prevention"
    rule_set_type    = "OWASP"
    rule_set_version = "3.2"
  }

  # Rewrite rules pour headers de sécurité
  rewrite_rule_set {
    name = "security-headers"
    
    rewrite_rule {
      name          = "add-hsts"
      rule_sequence = 100
      
      response_header_configuration {
        header_name  = "Strict-Transport-Security"
        header_value = "max-age=31536000; includeSubDomains"
      }
    }

    rewrite_rule {
      name          = "remove-server-header"
      rule_sequence = 200
      
      response_header_configuration {
        header_name  = "Server"
        header_value = ""
      }
    }
  }

  tags = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}
```

**C. Monitoring avec Azure Monitor - Terraform**
```hcl
# Log Analytics Workspace
resource "azurerm_log_analytics_workspace" "main" {
  name                = "lb-logs"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "PerGB2018"
  retention_in_days   = 30
}

# Diagnostic Settings pour Load Balancer
resource "azurerm_monitor_diagnostic_setting" "lb" {
  name                       = "lb-diagnostics"
  target_resource_id         = azurerm_lb.main.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id

  enabled_log {
    category = "LoadBalancerAlertEvent"
  }

  enabled_log {
    category = "LoadBalancerProbeHealthStatus"
  }

  metric {
    category = "AllMetrics"
    enabled  = true
  }
}

# Diagnostic Settings pour Application Gateway
resource "azurerm_monitor_diagnostic_setting" "appgw" {
  name                       = "appgw-diagnostics"
  target_resource_id         = azurerm_application_gateway.main.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id

  enabled_log {
    category = "ApplicationGatewayAccessLog"
  }

  enabled_log {
    category = "ApplicationGatewayPerformanceLog"
  }

  enabled_log {
    category = "ApplicationGatewayFirewallLog"
  }

  metric {
    category = "AllMetrics"
    enabled  = true
  }
}

# Alert pour Load Balancer unhealthy
resource "azurerm_monitor_metric_alert" "lb_unhealthy" {
  name                = "lb-unhealthy-backends"
  resource_group_name = azurerm_resource_group.main.name
  scopes              = [azurerm_lb.main.id]
  description         = "Alert when backend health drops"
  severity            = 2
  frequency           = "PT1M"
  window_size         = "PT5M"

  criteria {
    metric_namespace = "Microsoft.Network/loadBalancers"
    metric_name      = "DipAvailability"
    aggregation      = "Average"
    operator         = "LessThan"
    threshold        = 50
  }

  action {
    action_group_id = azurerm_monitor_action_group.main.id
  }
}

# Alert pour Application Gateway failed requests
resource "azurerm_monitor_metric_alert" "appgw_failed" {
  name                = "appgw-failed-requests"
  resource_group_name = azurerm_resource_group.main.name
  scopes              = [azurerm_application_gateway.main.id]
  description         = "Alert when failed requests exceed threshold"
  severity            = 1
  frequency           = "PT1M"
  window_size         = "PT5M"

  criteria {
    metric_namespace = "Microsoft.Network/applicationGateways"
    metric_name      = "FailedRequests"
    aggregation      = "Total"
    operator         = "GreaterThan"
    threshold        = 100
  }

  action {
    action_group_id = azurerm_monitor_action_group.main.id
  }
}
```

**14. Best Practices DevOps - Checklist de Production**

**Load Balancer :**
- ✅ **Toujours utiliser Standard SKU** (Basic deprecated)
- ✅ **Configurer health probes avec endpoints dédiés** `/health` ou `/healthz`
- ✅ **Utiliser zones de disponibilité** pour HA multi-datacenter
- ✅ **Activer TCP Reset** pour connexions stale
- ✅ **Monitorer DipAvailability metric** (santé des backends)
- ✅ **Configurer NSG pour autoriser AzureLoadBalancer tag**
- ✅ **Utiliser NAT Gateway pour outbound** (pas outbound rules LB)
- ✅ **Session persistence selon besoin applicatif** (stateful vs stateless)
- ✅ **Logs diagnostiques vers Log Analytics**
- ✅ **Alerts sur health probe failures**

**Application Gateway :**
- ✅ **Toujours déployer v2** (v1 legacy)
- ✅ **Activer autoscaling** min=2, max=10+
- ✅ **WAF en mode Prevention** pour production
- ✅ **End-to-end SSL** pour données sensibles
- ✅ **TLS 1.2+ uniquement** (désactiver TLS 1.0/1.1)
- ✅ **Health probes custom** avec vérifications applicatives
- ✅ **Connection draining** activé (60s minimum)
- ✅ **Rewrite rules** pour headers de sécurité (HSTS, X-Frame-Options)
- ✅ **Dedicated subnet** /24 minimum pour App Gateway
- ✅ **Custom error pages** pour meilleure UX
- ✅ **Logs vers SIEM** (Sentinel, Splunk)
- ✅ **Monitor Capacity Units** pour coûts
- ✅ **Alerts sur 5xx errors** et latence

**Sécurité :**
- ✅ **NSG sur subnet Application Gateway**
- ✅ **WAF custom rules** pour votre application
- ✅ **Rate limiting** pour DDoS protection
- ✅ **IP whitelisting** si applicable
- ✅ **Certificates dans Key Vault** (pas dans code)
- ✅ **Rotation automatique certificats**
- ✅ **RBAC strict** sur ressources load balancing

**Monitoring et Alerting :**
- ✅ **Dashboard Azure Monitor** avec métriques clés
- ✅ **Alerts sur santé backends**
- ✅ **Alerts sur latence élevée**
- ✅ **Alerts sur erreurs 5xx**
- ✅ **Workbooks pour analyse trafic**
- ✅ **Integration avec outils DevOps** (PagerDuty, Slack)

**15. Troubleshooting Avancé - Playbook DevOps**

**Problème : Backends marqués unhealthy**
```bash
# 1. Vérifier status health probes
az network lb show --name myLB --resource-group myRG \
  --query "probes[].{Name:name,Protocol:protocol,Port:port,Path:requestPath}"

# 2. Tester manuellement l'endpoint
curl -v http://10.0.1.4/health

# 3. Vérifier NSG rules
az network nsg show --name myNSG --resource-group myRG \
  --query "securityRules[?destinationPortRange=='80']"

# 4. Vérifier depuis la VM que le service écoute
ssh user@vm1
netstat -tlnp | grep :80

# 5. Vérifier logs applicatifs
tail -f /var/log/app/error.log
```

**Problème : Application Gateway erreurs 502**
```bash
# 1. Vérifier backend health
az network application-gateway show-backend-health \
  --name myAppGateway \
  --resource-group myRG

# 2. Analyser logs
az monitor log-analytics query \
  --workspace myWorkspace \
  --analytics-query "
    AzureDiagnostics
    | where ResourceType == 'APPLICATIONGATEWAYS'
    | where httpStatus_d == 502
    | project TimeGenerated, requestUri_s, backendSettingName_s
    | take 50"

# 3. Vérifier timeout settings
az network application-gateway http-settings show \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSettings \
  --query "requestTimeout"

# 4. Tester backend directement
curl -H "Host: api.contoso.com" http://10.0.1.4/api/endpoint
```

**Problème : Performance dégradée**
```bash
# Load Balancer - Vérifier SNAT port exhaustion
az monitor metrics list \
  --resource myLB \
  --metric "UsedSNATPorts" \
  --resource-type Microsoft.Network/loadBalancers

# Application Gateway - Vérifier capacity units
az monitor metrics list \
  --resource myAppGateway \
  --metric "CurrentCapacityUnits" \
  --resource-type Microsoft.Network/applicationGateways

# Latency analysis
az monitor metrics list \
  --resource myAppGateway \
  --metric "ApplicationGatewayTotalTime,BackendResponseTime"
```

**16. Coûts et Optimisation**

**Load Balancer Standard - Calcul Coûts**
```
Coût mensuel = Base fee + Data processed

Base fee: ~$18/mois
Data processed: $0.005/GB

Exemple :
- 1TB trafic/mois = 1000 GB × $0.005 = $5
- Total: $18 + $5 = $23/mois
```

**Application Gateway v2 - Calcul Coûts**
```
Coût mensuel = Fixed capacity units + Variable capacity units + Data processed

Fixed (2 CU): ~$145/mois
Variable CU: $0.008/CU-hour
Data processed: $0.008/GB

Exemple avec autoscaling 2-5 CU:
- Base (2 CU): $145
- Variable (3 CU × 730h × $0.008): $17.52
- Data (500 GB × $0.008): $4
- Total: ~$167/mois
```

**Optimisation Coûts :**
- **Load Balancer** : Moins cher pour TCP/UDP simple
- **App Gateway v2** : Autoscaling évite surprovisionnement
- **Zones de disponibilité** : Pas de coût supplémentaire (sauf data transfer)
- **WAF** : +30-40% coût vs Standard (évaluer nécessité)

#### Comparaison Détaillée : Azure Load Balancer vs Application Gateway

**🎯 Différences Critiques pour l'Examen AZ-104**

**1. Couche OSI et Protocoles**

**Azure Load Balancer (Layer 4 - Transport)**
- **Protocoles supportés** : TCP, UDP uniquement
- **Fonctionnement** : Distribution basée sur IP source/destination + port
- **Visibilité** : Ne peut pas voir le contenu des paquets
- **Usage** : Load balancing basique, NAT, HA ports
- **Performance** : Très haute (pas d'inspection de contenu)

**Application Gateway (Layer 7 - Application)**
- **Protocoles supportés** : HTTP, HTTPS uniquement
- **Fonctionnement** : Inspection du contenu HTTP/HTTPS
- **Visibilité** : Peut analyser headers, URLs, cookies, body
- **Usage** : Routage intelligent, WAF, SSL termination
- **Performance** : Plus élevée latence (inspection de contenu)

**2. Fonctionnalités et Capacités**

**Azure Load Balancer - Fonctionnalités Clés**
- **Health Probes** : TCP, HTTP, HTTPS
- **Session Persistence** : Client IP, Client IP + Protocol, None
- **NAT Rules** : Port forwarding, inbound/outbound
- **HA Ports** : Load balancing sur tous les ports
- **Backend Pools** : VMs, VMSS, IP addresses
- **Distribution Methods** : 5-tuple hash, 3-tuple hash, Source IP

**Application Gateway - Fonctionnalités Clés**
- **URL-based Routing** : Routage basé sur le chemin URL
- **Host-based Routing** : Routage basé sur le header Host
- **Path-based Routing** : Routage basé sur le chemin de l'URL
- **Multi-site Hosting** : Plusieurs domaines sur même gateway
- **SSL Termination** : Décryptage SSL côté gateway
- **WAF Integration** : Protection contre OWASP Top 10
- **Cookie-based Affinity** : Session persistence basée sur cookies

**3. Cas d'Usage et Scénarios**

**Utiliser Azure Load Balancer quand :**
- **Applications non-HTTP** : Bases de données, services TCP/UDP
- **Performance maximale** : Latence minimale requise
- **Simplicité** : Load balancing basique sans inspection
- **Coût** : Solution la moins chère
- **Backend hétérogène** : Mélange de services différents

**Utiliser Application Gateway quand :**
- **Applications web** : Sites web, APIs REST
- **Routage intelligent** : Besoin de router selon URL/host
- **Sécurité web** : Protection contre attaques web
- **SSL centralisé** : Gestion centralisée des certificats
- **Multi-tenant** : Plusieurs sites sur même infrastructure

**4. Architecture et Déploiement**

**Azure Load Balancer**
- **Types** : Public, Internal
- **SKUs** : Basic, Standard
- **Backend** : VMs, VMSS, IP addresses
- **Frontend** : Public IP ou Private IP
- **Zones** : Standard SKU supporte Availability Zones

**Application Gateway**
- **Types** : v1, v2 (WAF v2)
- **SKUs** : Standard, WAF, Standard_v2, WAF_v2
- **Backend** : VMs, VMSS, App Services, IP addresses
- **Frontend** : Public IP uniquement
- **Zones** : v2 SKU supporte Availability Zones

**5. Configuration et Gestion**

**Azure Load Balancer - Configuration Type**
```json
{
  "loadBalancingRules": [
    {
      "name": "LBRule",
      "protocol": "Tcp",
      "frontendPort": 80,
      "backendPort": 80,
      "enableFloatingIP": false
    }
  ],
  "probes": [
    {
      "name": "HTTPProbe",
      "protocol": "Http",
      "port": 80,
      "path": "/health"
    }
  ]
}
```

**Application Gateway - Configuration Type**
```json
{
  "routingRules": [
    {
      "name": "Rule1",
      "ruleType": "Basic",
      "httpListener": "Listener1",
      "backendAddressPool": "Pool1",
      "backendHttpSettings": "Settings1"
    }
  ],
  "httpListeners": [
    {
      "name": "Listener1",
      "frontendPort": "Port1",
      "protocol": "Http"
    }
  ]
}
```

**6. Coûts et Facturation**

**Azure Load Balancer**
- **Basic** : Gratuit (limitations)
- **Standard** : ~$18/mois + trafic sortant
- **Facturation** : Par règle + trafic

**Application Gateway**
- **v1** : ~$18/mois + capacité
- **v2** : Pay-per-use + capacité
- **WAF** : Coût supplémentaire
- **Facturation** : Par heure + capacité + données

**7. Limitations et Contraintes**

**Azure Load Balancer**
- **Basic SKU** : Pas de HA ports, pas de zones
- **Backend** : Maximum 1000 instances
- **Rules** : Maximum 150 rules
- **Probes** : Maximum 5 probes

**Application Gateway**
- **Backend** : Maximum 100 instances
- **Rules** : Maximum 100 rules
- **Listeners** : Maximum 40 listeners
- **Certificates** : Maximum 20 certificates

**8. Matrice de Décision Rapide**

| Critère | Azure Load Balancer | Application Gateway |
|---------|-------------------|-------------------|
| **Protocole** | TCP/UDP | HTTP/HTTPS |
| **Couche** | Layer 4 | Layer 7 |
| **Performance** | Très haute | Haute |
| **Coût** | Faible | Élevé |
| **Sécurité** | Basique | Avancée (WAF) |
| **Routage** | Basique | Intelligent |
| **SSL** | Pas de gestion | Termination |
| **Monitoring** | Métriques de base | Métriques avancées |

**9. Scénarios d'Examen Courants**

**Scénario 1 : Application Web avec Routage**
- **Besoin** : Router `/api` vers backend API, `/app` vers frontend
- **Solution** : Application Gateway avec path-based routing
- **Raison** : Load Balancer ne peut pas router selon URL

**Scénario 2 : Base de Données avec HA**
- **Besoin** : Load balancing pour SQL Server
- **Solution** : Azure Load Balancer
- **Raison** : Application Gateway ne supporte que HTTP/HTTPS

**Scénario 3 : Sécurité Web**
- **Besoin** : Protection contre attaques OWASP
- **Solution** : Application Gateway avec WAF
- **Raison** : Load Balancer n'a pas de fonctionnalités de sécurité web

**Scénario 4 : Performance Maximale**
- **Besoin** : Latence minimale pour application critique
- **Solution** : Azure Load Balancer
- **Raison** : Pas d'inspection de contenu = latence minimale

#### Traffic Manager (DNS-based)
- **Global** : Répartition géographique
- **Methods** : Performance, Geographic, Weighted, Priority
- **Health monitoring** : Surveillance des endpoints

### 4.4 Network Watcher - Outils de Diagnostic Réseau

**⚠️ Erreur Courante QCM : Choisir le bon outil Network Watcher**

#### Vue d'ensemble et Outils
**Azure Network Watcher** est un service central de monitoring et diagnostic réseau qui fournit des outils pour :
- **Monitoring** : Surveillance continue des ressources réseau
- **Diagnostics** : Identification et résolution de problèmes de connectivité
- **Métriques** : Visualisation des performances réseau
- **Logging** : Activation/désactivation des logs pour ressources Azure VNet

**Disponibilité :**
- **Activation** : Automatique lors de la création d'un VNet
- **Scope** : Par région (un Network Watcher par région)
- **Gestion** : Network Watcher → Région → Outils disponibles

**Outils Network Watcher - Vue d'ensemble :**

| Outil | Use Case Principal | Sortie | Rapidité | Complexité |
|-------|-------------------|--------|----------|------------|
| **IP Flow Verify** | Vérifier si NSG bloque | Allow/Deny + Règle NSG | Immédiat | Faible |
| **Connection Troubleshoot** | Tester connectivité VM → VM | Reachable/Unreachable | Rapide | Faible |
| **Next Hop** | Vérifier routage | Type de hop suivant | Immédiat | Faible |
| **Topology** | Visualiser architecture | Diagramme réseau | Rapide | Faible |
| **Packet Capture** | Analyser trafic détaillé | Fichier .cap | Long | Élevée |
| **NSG Flow Logs** | Analyser historique trafic | Logs JSON | Délai analyse | Élevée |
| **Connection Monitor** | Surveiller latence continue | Métriques temps réel | Continu | Moyenne |

#### 1. IP Flow Verify - Diagnostic NSG (⭐ Outil Principal pour NSG)

**⚠️ Scénario d'Examen Typique :**
- **Question** : "Une VM ne peut pas se connecter à une autre VM. Quel outil utiliser pour identifier la règle NSG bloquante ?"
- **Réponse correcte** : **IP Flow Verify**

**Fonctionnalités principales :**
- **Spécification complète** : Source/destination IPv4, port, protocole (TCP/UDP), direction (inbound/outbound)
- **Identification précise** : Identifie le **NSG spécifique** et la **règle exacte** qui bloque
- **Résultat immédiat** : Allow ou Deny avec détails de la règle
- **Cas d'usage** : Troubleshooting rapide des problèmes de connectivité NSG

**Configuration via Azure CLI :**
```bash
# Vérifier si le trafic est autorisé
az network watcher test-ip-flow \
  --resource-group myRG \
  --vm myVM \
  --direction Outbound \
  --protocol TCP \
  --local 10.0.0.4:80 \
  --remote 10.1.0.4:3389

# Résultat exemple :
# Access: Deny
# Rule Name: UserRule_DenyRDP
# NSG: myNSG
```

**Configuration via PowerShell :**
```powershell
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $nw `
  -TargetVirtualMachineId $vm.Id `
  -Direction Outbound `
  -Protocol TCP `
  -LocalIPAddress "10.0.0.4" `
  -LocalPort "80" `
  -RemoteIPAddress "10.1.0.4" `
  -RemotePort "3389"
```

**Sortie Détaillée :**
```json
{
  "access": "Deny",
  "ruleName": "SecurityRule_DenyAll",
  "networkSecurityGroup": "/subscriptions/.../networkSecurityGroups/myNSG"
}
```

**Avantages :**
- ✅ **Identification directe** du NSG bloquant
- ✅ **Configuration minimale**, test immédiat
- ✅ **Résultat précis** : règle NSG exacte responsable du blocage
- ✅ **Usage** : Diagnostic rapide et précis

#### 2. Connection Troubleshoot - Test de Connectivité End-to-End

**Use Case :**
- Tester la connectivité entre deux ressources Azure
- Identifier si la communication est possible (reachable/unreachable)
- Détecter les problèmes de routage, NSG, firewall

**Sources supportées :**
- Virtual Machines
- VM Scale Sets
- Application Gateway
- Azure Bastion

**Destinations supportées :**
- VM (IP privée)
- URI (HTTP/HTTPS)
- FQDN (nom de domaine)
- IPv4 publique ou privée

**Configuration via Azure CLI :**
```bash
# Tester connectivité VM → VM
az network watcher test-connectivity \
  --resource-group myRG \
  --source-resource myVM1 \
  --dest-resource myVM2 \
  --protocol TCP \
  --dest-port 443

# Tester connectivité VM → URL
az network watcher test-connectivity \
  --resource-group myRG \
  --source-resource myVM1 \
  --dest-address www.microsoft.com \
  --protocol TCP \
  --dest-port 443
```

**Sortie Exemple :**
```json
{
  "connectionStatus": "Reachable",
  "avgLatencyInMs": 4,
  "minLatencyInMs": 2,
  "maxLatencyInMs": 8,
  "probesSent": 10,
  "probesFailed": 0
}
```

**Cas d'échec - Diagnostics :**
```json
{
  "connectionStatus": "Unreachable",
  "hops": [
    {
      "type": "Source",
      "id": "myVM1",
      "issues": []
    },
    {
      "type": "VirtualNetwork",
      "issues": []
    },
    {
      "type": "NetworkSecurityGroup",
      "id": "myNSG",
      "issues": [
        {
          "type": "NetworkSecurityRule",
          "context": ["Rule 'DenyHTTPS' blocked the connection"]
        }
      ]
    }
  ]
}
```

#### 3. Next Hop - Vérification du Routage

**Use Case :**
- Déterminer comment le trafic est routé depuis une VM
- Identifier le prochain saut (next hop) pour une destination
- Diagnostiquer problèmes de routage (UDR, routes système)

**Types de Next Hop :**
- **Internet** : Trafic vers Internet
- **VirtualAppliance** : Via une appliance réseau (NVA)
- **VirtualNetworkGateway** : Via VPN ou ExpressRoute Gateway
- **VnetLocal** : Destination dans le même VNet
- **VNetPeering** : Destination via VNet Peering
- **None** : Aucun routage (trafic bloqué)

**Configuration via Azure CLI :**
```bash
az network watcher show-next-hop \
  --resource-group myRG \
  --vm myVM \
  --source-ip 10.0.0.4 \
  --dest-ip 10.1.0.4

# Résultat :
# NextHopType: VNetPeering
# NextHopIpAddress: 10.1.0.4
# RouteTableId: System Route
```

**Scénario d'Examen :**
| Question | Next Hop Type | Raison |
|----------|---------------|--------|
| Trafic vers 8.8.8.8 | **Internet** | Destination publique |
| Trafic vers 10.1.0.4 (VNet peeré) | **VNetPeering** | Via peering |
| Trafic via NVA Firewall | **VirtualAppliance** | UDR configuré |
| Trafic vers on-premises via VPN | **VirtualNetworkGateway** | Via VPN Gateway |

#### 4. Topology - Visualisation du Réseau

**Use Case :**
- Visualiser l'architecture réseau d'un Resource Group
- Comprendre les interconnexions entre ressources
- Identifier rapidement la topologie globale

**Éléments affichés :**
- VNets et Subnets
- VMs et NICs
- Load Balancers
- Application Gateways
- VNet Peerings
- VPN Gateways
- NSGs associés

**Configuration via Azure CLI :**
```bash
az network watcher show-topology \
  --resource-group myRG \
  --output json > topology.json
```

**Utilisation Portal :**
```
Network Watcher → Topology → Sélectionner Resource Group
```

#### 5. Packet Capture - Analyse de Paquets Réseau

**Use Case :**
- Capture détaillée du trafic réseau pour analyse approfondie
- Debugging de problèmes applicatifs (HTTP, SQL, etc.)
- Analyse de sécurité (détection d'intrusions)

**Configuration :**
```bash
# Démarrer une capture
az network watcher packet-capture create \
  --resource-group myRG \
  --vm myVM \
  --name myCapture \
  --storage-account mystorageaccount \
  --time-limit 60

# Arrêter la capture
az network watcher packet-capture stop \
  --resource-group myRG \
  --name myCapture \
  --location eastus

# Télécharger et analyser avec Wireshark
```

**Limitations :**
- Nécessite l'extension VM Network Watcher
- Capture limitée en durée (max 5 heures)
- Impact potentiel sur les performances VM
- Fichiers .cap volumineux

**⚠️ Important :** Ne convient PAS pour identifier rapidement un NSG bloquant (utiliser IP Flow Verify)

#### 6. NSG Flow Logs - Analyse Historique du Trafic

**Use Case :**
- Analyser le trafic IP à travers les NSG sur une période prolongée
- Auditer et tracer les connexions réseau
- Détecter anomalies et patterns de trafic
- Conformité et forensic

**Configuration :**
```bash
# Activer NSG Flow Logs
az network watcher flow-log create \
  --resource-group myRG \
  --nsg myNSG \
  --storage-account mystorageaccount \
  --enabled true \
  --retention 90 \
  --format JSON \
  --log-version 2

# Activer Traffic Analytics (optionnel)
az network watcher flow-log configure \
  --nsg myNSG \
  --enabled true \
  --storage-account mystorageaccount \
  --workspace myLogAnalyticsWorkspace \
  --traffic-analytics true
```

**Format des Logs (Version 2) :**
```json
{
  "time": "2025-10-27T10:00:00Z",
  "systemId": "xxx",
  "macAddress": "00-0D-3A-...",
  "category": "NetworkSecurityGroupFlowEvent",
  "resourceId": "/subscriptions/.../networkSecurityGroups/myNSG",
  "operationName": "NetworkSecurityGroupFlowEvents",
  "properties": {
    "Version": 2,
    "flows": [
      {
        "rule": "Allow-HTTP",
        "flows": [
          {
            "mac": "00-0D-3A-...",
            "flowTuples": [
              "1698393600,10.0.0.4,10.1.0.4,80,3389,T,I,A,C,1024,512,10,5"
            ]
          }
        ]
      }
    ]
  }
}
```

**Tuple Format :**
```
timestamp,source_ip,dest_ip,source_port,dest_port,protocol,flow_direction,flow_state,encryption,bytes_sent,bytes_received,packets_sent,packets_received
```

**Traffic Analytics :**
- **Visualisation** : Dashboards dans Azure Monitor
- **Insights** : Top talkers, blocked traffic, geo-distribution
- **Alerting** : Alertes sur anomalies détectées

**Limitations :**
- ❌ Configuration complexe (Storage Account + Log Analytics requis)
- ❌ Analyse manuelle des logs JSON
- ❌ Délai d'ingestion (5-10 minutes)
- ❌ Ne pointe pas directement le NSG problématique en temps réel

#### 7. Connection Monitor - Surveillance Continue de Latence

**Use Case :**
- Mesurer la latence (RTT - Round Trip Time) entre VMs en continu
- Surveiller la disponibilité des endpoints
- Détecter dégradations de performance réseau
- Monitoring proactif

**Caractéristiques :**
- **Granularité** : Métriques par minute
- **Targets** : VM, FQDN, URI, IPv4 privée/publique
- **Protocols** : TCP, HTTP, HTTPS, ICMP
- **Monitoring** : 24/7 avec historique

**Configuration via Azure CLI :**
```bash
# Créer un Connection Monitor
az network watcher connection-monitor create \
  --name myConnectionMonitor \
  --location eastus \
  --endpoint-source-name VM1 \
  --endpoint-source-resource-id $vm1Id \
  --endpoint-dest-name VM2 \
  --endpoint-dest-resource-id $vm2Id \
  --test-config-name HTTPTest \
  --protocol HTTP \
  --http-port 80 \
  --test-frequency 60

# Voir les résultats
az network watcher connection-monitor show \
  --name myConnectionMonitor \
  --location eastus
```

**Métriques collectées :**
- **RTT (Round Trip Time)** : Latence moyenne, min, max
- **Probes Sent/Failed** : Taux de succès
- **Checks Failed Percent** : Pourcentage d'échec

**Alertes :**
```bash
# Créer une alerte si RTT > 100ms
az monitor metrics alert create \
  --name HighLatencyAlert \
  --resource-group myRG \
  --scopes $connectionMonitorId \
  --condition "avg RoundTripTimeMs > 100" \
  --description "Alert when RTT exceeds 100ms"
```

#### Matrice de Décision - Quel Outil Utiliser ?

| Symptôme / Question | Outil Recommandé | Raison |
|---------------------|-----------------|--------|
| **VM ne peut pas communiquer avec une autre VM** | IP Flow Verify | Identifie NSG bloquant + règle exacte |
| **Identifier quelle règle NSG bloque le trafic** | IP Flow Verify | Diagnostic NSG en temps réel |
| **Tester si une VM peut atteindre une URL** | Connection Troubleshoot | Test end-to-end avec diagnostics |
| **Comprendre le routage du trafic** | Next Hop | Montre le prochain saut |
| **Visualiser l'architecture réseau** | Topology | Diagramme de toutes les ressources |
| **Analyser trafic détaillé (debugging app)** | Packet Capture | Capture complète pour Wireshark |
| **Auditer historique de trafic NSG** | NSG Flow Logs | Logs détaillés sur période longue |
| **Surveiller latence en continu** | Connection Monitor | Monitoring proactif 24/7 |

#### Comparaison Détaillée - Scénarios d'Examen

**Scénario 1 : Communication VM bloquée - Diagnostic rapide**
- **Besoin** : Identifier pourquoi une VM ne peut pas communiquer
- **Solution** : **IP Flow Verify** ✅
- **Raison** : Identification immédiate du NSG et de la règle bloquante
- **Alternative (mauvaise)** : NSG Flow Logs ❌ (trop long, analyse manuelle)

**Scénario 2 : Vérifier connectivité à un service externe**
- **Besoin** : Tester si une VM peut atteindre https://api.example.com
- **Solution** : **Connection Troubleshoot** ✅
- **Raison** : Test direct avec indication reachable/unreachable

**Scénario 3 : Comprendre pourquoi le trafic passe par un NVA**
- **Besoin** : Déterminer le routage du trafic
- **Solution** : **Next Hop** ✅
- **Raison** : Montre le type de routage (VirtualAppliance) et l'IP du NVA

**Scénario 4 : Analyser un problème applicatif HTTP complexe**
- **Besoin** : Voir les requêtes/réponses HTTP détaillées
- **Solution** : **Packet Capture** ✅
- **Raison** : Capture complète pour analyse Wireshark

**Scénario 5 : Conformité - Audit des connexions sur 6 mois**
- **Besoin** : Tracer toutes les connexions pour audit
- **Solution** : **NSG Flow Logs** + Traffic Analytics ✅
- **Raison** : Historique complet avec rétention longue durée

#### Best Practices Network Watcher

✅ **À FAIRE :**
- Utiliser **IP Flow Verify** en premier pour problèmes NSG
- Activer **NSG Flow Logs** pour audit et conformité
- Configurer **Connection Monitor** pour services critiques
- Utiliser **Topology** pour comprendre l'architecture

❌ **À ÉVITER :**
- Utiliser Packet Capture pour identifier un NSG bloquant (trop complexe)
- Analyser manuellement NSG Flow Logs sans Traffic Analytics
- Oublier d'activer Network Watcher dans les nouvelles régions

** Stratégie de diagnostic identifiée :**
1. **Premier choix** : IP Flow Verify pour identifier rapidement le NSG
2. **Deuxième choix** : NSG Flow Logs pour analyse historique
3. **Dernier recours** : Packet Capture pour investigation approfondie

** Tips critiques identifiés :**

**1. Commandes de Diagnostic Spécialisées**
- **`netstat -an`** : Diagnostic des ports d'écoute (essentiel pour troubleshooting)
- **`Test-NetConnection`** : Tests de connectivité modernes (remplace ping)
- **`nbtstat -c`** : Diagnostic NetBIOS (legacy, moins fréquent)
- **`Get-AzVirtualNetworkUsageList`** : PowerShell Azure (pas de diagnostic réseau)

**2. Stratégie de Diagnostic par Couche**
- **Couche Application** : `netstat -an` pour ports d'écoute
- **Couche Transport** : `Test-NetConnection` pour tests TCP/UDP
- **Couche Réseau** : `ping` ou `Test-NetConnection` pour ICMP
- **Couche Application** : `nslookup` pour résolution DNS

**3. Outils Azure vs Outils Système**
- **Azure PowerShell** : `Get-Az*` pour gestion des ressources Azure
- **Outils Windows** : `netstat`, `Test-NetConnection` pour diagnostic réseau
- **Outils Legacy** : `nbtstat`, `ping` pour compatibilité
- **Règle** : Diagnostic réseau = outils système, pas PowerShell Azure

#### Traffic Analytics
** Ressources requises identifiées :**
1. **Log Analytics Workspace** : Analyse et stockage
2. **Storage Account** : Stockage NSG Flow Logs
3. **NSG Flow Logs** : Source de données activée

**Prérequis :** Même région pour tous les composants

#### Other Features
- **IP Flow Verify** : Tester règles NSG
- **Next Hop** : Routing troubleshooting
- **Packet Capture** : Capture de paquets sur VMs

### 4.5 VPN et ExpressRoute

#### Site-to-Site VPN
- **Virtual Network Gateway** : Passerelle Azure
- **Local Network Gateway** : Représentation on-premises
- **Connection** : Lien entre les gateways
- **Protocols** : IKEv1, IKEv2, SSTP

** Cas d'usage identifié :** Connexions chiffrées on-premises
- **Scénario** : Activer la connectivité VNet vers ressources on-premises avec connexion chiffrée
- **Solution** : Configurer une Virtual Network Gateway (VPN Gateway)
- **Mécanisme** : Envoie du trafic chiffré entre un réseau virtuel et un emplacement on-premises via connexion publique
- **Configuration** : Dépend de plusieurs ressources avec paramètres configurables

** Erreur fréquente identifiée :** Confusion entre Private Endpoints et VPN Gateways
- ** Erreur** : Utiliser des Private Endpoints pour la connectivité on-premises
- ** Correct** : Utiliser des Virtual Network Gateways pour les connexions chiffrées
- **Différenciation** : Private Endpoints = Accès privé aux services Azure
- **Usage** : VPN Gateways = Connexions chiffrées vers on-premises

#### Point-to-Site VPN
- **Client certificates** : Authentification par certificat
- **Azure AD authentication** : Authentification moderne
- **RADIUS** : Authentification externe

#### ExpressRoute
- **Private connectivity** : Connexion privée dédiée
- **No Internet** : Pas de transit par Internet
- **Higher bandwidth** : Jusqu'à 100 Gbps
- **Lower latency** : Latence prévisible

### 4.6 Azure Bastion

#### Vue d'ensemble
**Azure Bastion** est un service PaaS géré qui fournit une connectivité sécurisée aux VMs

**Caractéristiques principales :**
- **Connexion via navigateur** : RDP/SSH directement depuis le portail Azure
- **Sécurité renforcée** : Pas d'exposition des ports RDP (3389) et SSH (22)
- **Pas d'IP publique requise** : Les VMs restent complètement privées
- **Protection DDoS** : Intégré avec Azure DDoS Protection
- **Protocoles** : RDP et SSH via SSL (port 443)

#### Comparaison des Solutions d'Accès aux VMs

**Azure Bastion (Recommandé)**
- **Avantage** : Sécurité maximale, pas d'exposition de ports
- **Accès** : Via navigateur et portail Azure
- **Configuration** : Service géré, simple à déployer
- **Coût** : Service facturé par heure
- **Usage** : Production, environnements sécurisés

**Remote Desktop (RDP Direct)**
- **Type** : Fonctionnalité du système d'exploitation Windows
- **Exposition** : Port RDP (3389) exposé sur Internet
- **Risque** : Vulnérable aux attaques brute force
- **Configuration** : Nécessite IP publique sur la VM
- **Usage** : Environnements de développement uniquement

**Azure Monitor**
- **Fonction** : Monitoring et diagnostics
- **Limitation** : N'offre PAS de connectivité aux VMs
- **Usage** : Surveillance des performances

**Azure Network Watcher**
- **Fonction** : Diagnostic réseau
- **Limitation** : N'offre PAS de connectivité RDP/SSH
- **Usage** : Troubleshooting connectivité

** Point clé identifié :** Sécurité vs Accessibilité
- **Azure Bastion** : Meilleure pratique pour accès sécurisé sans exposition de ports
- **RDP Direct** : Solution simple mais dangereuse, exposée aux attaques
- **Règle** : Toujours préférer Azure Bastion en production

#### Références Microsoft
- Quickstart - Create an Azure private DNS zone using the Azure portal | Microsoft Learn
- Configure Azure DNS - Training | Microsoft Learn
- Monitor and maintain Azure resources
- Azure Network Watcher | Microsoft Learn

---
