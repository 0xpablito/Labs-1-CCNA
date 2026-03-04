# Infrastructure L2/L3 - Segmentation & Redondance

## 🎯 Objectifs et Démarche
Après avoir suivi le parcours théorique du CCNA et assimilé les fondamentaux du réseau, j'ai choisi de dépasser le cadre des exercices guidés pour concevoir une infrastructure de zéro. L'objectif était de tester mes automatismes et de consolider mes acquis avant l'examen **CCNA 200-301**.

Ma méthode d'apprentissage repose sur une approche concrète : **identifier et résoudre mes propres erreurs de configuration.** Ce lab a été réalisé avec un minimum d'aide extérieure pour me forcer à diagnostiquer chaque dysfonctionnement manuellement, car c'est de cette manière que j'assimile le mieux les concepts.

### 🛠️ Compétences techniques validées :
* **L2 (Commutation)** : VLANs, Trunks (802.1Q), Etherchannel (LACP), Spanning-Tree (PortFast, BPDU Guard) et Port-Security.
* **L3 (Routage)** : Routage Inter-VLAN (SVI & Router-on-a-Stick) et routage statique.
* **Services & Sécurité** : DHCP Server (avec gestion des exclusions), NAT/PAT (Overload), ACLs (Standard & Étendues) et sécurisation des accès (SSHv2).

> 💡 Cette approche m'a permis de transformer des erreurs de logique en opportunités d'apprentissage. Pour illustrer ma démarche de diagnostic, je consacre une section **Troubleshooting** en bas de page aux principaux défis techniques résolus durant ce lab.


## 1. Vue d'ensemble
Déploiement d'une architecture réseau hiérarchique. L'objectif est de valider la mise en place d'un cœur de réseau redondant et d'une segmentation stricte du trafic.
<img width="1565" height="709" alt="image" src="https://github.com/user-attachments/assets/fdd24f6d-166c-48aa-aea9-7b2858eb0cff" />





## 2. Implémentation technique

### Phase 1 : Configuration de base et Sécurité
Déploiement d'une configuration de base uniformisée et verrouillage des accès pour garantir l'intégrité et la gestion sécurisée du parc.

* Cette étape définit les bases de sécurité indispensables avant le déploiement des services réseau.
* Accès Distant : Migration vers SSHv2 (chiffrement RSA 1024 bits) et désactivation du protocole Telnet. 
* Identité & Accès : Création d'un compte admin local et protection du mode privilégié par hachage MD5. 
* Management : Configuration d'une interface SVI dédiée pour l'administration de l'équipement. 
* Confort CLI : Activation du logging synchronous pour éviter les interruptions lors de la saisie des commandes. 

🔗 [Consulter le script de base](./configs/01_base_setup.md)
🧪 [Consulter les tests de validation](./tests/01_base_setup.md)

### Phase 2 : Segmentation VLAN & Routage Inter-VLAN
Mise en place d'une isolation logique des services et centralisation du routage sur le cœur de réseau via une architecture hybride.

 Côté Siège (Switch L3)
* Segmentation : VLANs 10 (Admin), 20 (Prod), 30 (Sales), 40 (Guest).
* Routage SVI : Interfaces virtuelles sur le Switch L3 pour un routage inter-VLAN à vitesse filaire.
* Optimisation : Activation du PortFast sur les ports d'accès pour une connectivité instantanée des postes de travail.

 🔗 [Consulter le script de base](./configs/02.1_vlan_config.md)
 🧪 [Consulter les tests de validation](./tests/02.1_VLAN.md)

 Côté Opérations (Router-on-a-Stick)
* Segmentation : VLANs 70 (Partners) et 80 (Logistics).
* Routage Sub-interfaces : Utilisation du routeur central pour segmenter les flux opérationnels via le protocole 802.1Q.
* Lien Trunk : Configuration d'un lien d'agrégation entre le switch d'accès et le routeur pour transporter plusieurs VLANs sur un seul câble.

 🔗 [Consulter le script de base](./configs/02.2_vlan_config.md)
 🧪 [Consulter les tests de validation](./tests/02.2_VLAN.md)

 Sécurité Réseau Globale
* VLAN Natif (VLAN 99) : Migration du trafic non tagué vers un VLAN dédié sur tous les Trunks (Switchs et Routeur) pour contrer le VLAN Hopping.
* VLAN "BlackHole" (VLAN 999) : Redirection de tous les ports inutilisés vers un VLAN isolé avec extinction administrative (shutdown).

### Phase 3 : Haute Disponibilité & Performance Réseau
Optimisation des liaisons physiques et sécurisation de la topologie pour garantir une infrastructure résiliente face aux pannes.

* Agrégation de Liens (LACP) : Création de liens agrégés (Port-Channels) entre le Switch L3 et les switchs d'accès pour augmenter la bande passante et offrir une redondance matérielle.
* Interopérabilité : Utilisation du protocole LACP (IEEE 802.3ad). Ce choix garantit l'interopérabilité avec des équipements multi-constructeurs.
* Négociation Dynamique : Configuration en mode Active pour permettre une détection automatique des erreurs et une agrégation sécurisée des liens physiques.
* Sécurisation Spanning-Tree : Déploiement du BPDU Guard et du Root Guard pour éviter les boucles ou les switchs malveillants.
* Optimisation STP : Activation du PortFast sur tous les ports utilisateurs. Cela permet aux PC d'accéder au réseau immédiatement (en sautant les 30 secondes d'attente du Spanning-Tree) dès qu'ils sont branchés.

🔗 [Consulter le script de base](/configs/03_Etherchannel.md) 
🧪 [Consulter les tests de validation](/tests/03_Etherchannel.md)

### Phase 4 : Services IP & Connectivité WAN
* Finalisation de la couche de services pour l'autonomie des utilisateurs et l'ouverture sécurisée du réseau vers l'extérieur.
* Adressage Dynamique (DHCP) : Centralisation du service sur le Switch L3 (Siège) et le Routeur (Dépôt) pour automatiser l'attribution des adresses IP.
* Gestion des Exclusions : Réservation des adresses .1 à .5 dans chaque pool pour sécuriser les passerelles et les ressources statiques (comme l'imprimante réseau).
* Sécurité & Filtrage (ACL) :
  * ACL Étendue (101) : Isolation de l'imprimante de production (192.168.20.2) pour interdire tout flux provenant du VLAN 40 (Guest).
  * ACL de Management (10) : Restriction des accès VTY pour autoriser uniquement les postes du VLAN Admin à configurer les équipements.
* Translation d'Adresses (NAT/PAT) : Mise en œuvre du NAT Overload sur l'interface Serial du routeur pour permettre à tous les VLANs internes de partager une IP publique unique.
* Routage Statique : Configuration d'une route par défaut vers l'ISP et de routes récapitulatives pour assurer la communication bidirectionnelle entre le Siège et le Dépôt.

🔗 [Consulter le script de base](/configs/04_IP&WAN.md) 
🧪 [Consulter les tests de validation](/tests/04_IP&WAN.md)

## 🛠️ Troubleshooting : Défis Techniques & Résolutions

### 1. Configuration du Trunk sur EtherChannel
* **Symptôme** : Malgré la création du Port-Channel entre le Switch L3 et les switchs d'accès, le trafic des VLANs ne passait pas. Les interfaces restaient en mode "access" par défaut.
* **Diagnostic** : La commande `switchport mode trunk` avait été appliquée sur l'intervalle d'interfaces physiques (`interface range f0/21-22`) mais pas sur l'interface logique du groupe (`interface port-channel 1`). 
* **Résolution** : Pour corriger l'instabilité et l'incohérence de configuration, j'ai dû réinitialiser les interfaces physiques, supprimer le Port-Channel, puis recréer l'agrégation en appliquant les commandes `switchport mode trunk` directement sur l'interface Port-Channel.

### 2. Dysfonctionnements du NAT
* **Symptôme** : Le trafic provenant du Siège (192.168.x.x) sortait correctement sur Internet, mais les hôtes du Dépôt (172.16.x.x) ne généraient aucune entrée dans la table NAT (`show ip nat translations`).
* **Diagnostic** : Deux erreurs distinctes bloquaient le processus :
  * 1. **Erreur d'interface** : La commande `ip nat inside` était appliquée uniquement sur l'interface physique `g0/0/1`. Or, avec le Router-on-a-Stick, le trafic passe par les sous-interfaces.
    2. **Oubli de mise à jour de l'ACL** : Après avoir passé le Dépôt en 172.16.x.x, l'ACL 1 autorisait toujours uniquement l'ancien réseau (192.168.x.x). Le trafic arrivait bien au routeur, mais le NAT l'ignorait car il n'était pas dans la liste des réseaux autorisés à sortir.
* **Résolution** :
  * Ajout du `ip nat inside` sur les sous-interfaces `g0/0/1.70` et `g0/0/1.80`.
  * Ajout de la ligne `access-list 1 permit 172.16.0.0 0.0.255.255`.
 
 ### 3. Conflit d'IP et gestion des exclusions DHCP
* **Symptôme** : Impossible d'attribuer l'adresse statique prévue (`192.168.20.2`) à l'imprimante réseau.
* **Diagnostic** : Lors de mes tests, j'ai activé le DHCP sur les PC pour vérifier que le serveur distribuait bien les adresses. Comme je n'avais pas encore configuré les `ip dhcp excluded-address`, le serveur a pioché la première adresse disponible (.2) pour l'attribuer à un PC, créant un conflit avec l'IP statique réservée à l'imprimante.
* **Résolution** :
  * Configuration des exclusions sur le switch L3 : `ip dhcp excluded-address 192.168.20.1 192.168.20.5` pour protéger les passerelles et les ressources statiques.
  * Réinitialisation du processus DHCP sur l'ensemble des PC (`ipconfig /renew`) : cette commande force les PC à contacter à nouveau le serveur pour obtenir une nouvelle IP qui sera maintenant située dans le pool prévu (au-delà de la .5).

##  Configuration & Fichiers

###  Accès au Lab
Vous pouvez télécharger le fichier complet pour l'ouvrir dans Cisco Packet Tracer :
* [📥 Télécharger le lab (.pkt)](LAB-1.pkt)

### 📄 Fichiers de Configuration (Show Run)
Pour une lecture rapide, les configurations sont disponibles ici :
* [Routeur de Bordure (R-EDGE)](configs/R-EDGE.txt)
* [Switch de Cœur (CORE)](configs/CORE.txt)
* [Switch Accès Admin (ACC-01)](configs/ACC-01.txt)
* [Switch Accès Prod (ACC-02)](configs/ACC-02.txt)
* [Switch Accès Dépôt (DEPOT)](configs/DEPOT.txt)
