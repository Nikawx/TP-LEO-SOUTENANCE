# B3 CS - Infras Réseau Sécurisées

## Auteur
LEO - B3 CS

---

## Topologie

- **Routeur** : r1.tp1.efrei (Cisco c7200)
- **Core switch** : core1.tp1.efrei (IOU)
- **Access switches** : access1, access2, access3, access4 (IOU)
- **Clients VPCS** : mario, luigi, peach, daisy, lakitu
- **VMs Rocky Linux** : bowser.tp1.efrei, dhcp.tp1.efrei
- **Machine attaquante** : Kali Linux

---

## Réseaux et VLANs

| Réseau  | Adresse       | VLAN | Description              |
|---------|---------------|------|--------------------------|
| admins  | 10.1.10.0/24  | 10   | Laptops des admins       |
| clients | 10.1.20.0/24  | 20   | Laptops des autres gens  |
| servers | 10.1.30.0/24  | 30   | Serveurs de l'infra      |
| guests  | 10.1.40.0/24  | 40   | Accès réseau invités     |

---

## TP1 - Base setup

### Part 1 - Network Setup
- Configuration des sous-interfaces du routeur R1 (router-on-a-stick)
- Configuration des VLANs et trunks sur tous les switches
- Configuration des IPs statiques sur les VPCS et VMs Rocky
- Vérification : ping intra-VLAN et inter-VLAN

### Part 2 - Internet
- Configuration DHCP sur FastEthernet1/0 de R1 vers NAT1
- Configuration NAT/PAT sur R1
- Configuration DNS 1.1.1.1 sur les clients
- Vérification : ping efrei.fr depuis les clients

### Part 3 - DHCP
- Installation et configuration de dnsmasq sur dhcp.tp1.efrei
- Configuration DHCP relay sur R1 (ip helper-address)
- Distribution automatique des IPs, gateway et DNS aux clients
- Vérification : tous les clients récupèrent une IP via DHCP

---

## TP2 - Offensive Sec

### Part 1 - DHCP attacks
- **DHCP Spoofing** : rogue DHCP server avec dnsmasq sur la Kali
- **dhcp_race.py** : script Scapy simulant un serveur DHCP malveillant
- **dhcp_starvation.py** : script Scapy épuisant la plage d'adresses DHCP

### Part 2 - ARP poisoning
- **arp_poisoning.py** : script Scapy empoisonnant la table ARP d'une victime
- **arp_mitm.py** : script Scapy réalisant un MITM bidirectionnel entre bowser et R1

### Part 3 - DNS spoof
- MITM via DHCP spoof pour indiquer la Kali comme serveur DNS
- dnsmasq configuré pour répondre avec l'IP de la Kali pour efrei.fr
- Résultat : bowser ping 10.1.10.43 au lieu de 51.210.229.203

---

## TP3 - Defensive Sec

### Part 1 - DHCP Snooping
- Activé sur tous les switches pour les VLANs 10, 20, 30, 40
- Ports trusted : uniquement les ports vers équipements réseau
- Option 82 désactivée (incompatible avec dnsmasq)
- Résultat : le rogue DHCP de la Kali est bloqué

### Part 2 - DAI (Dynamic ARP Inspection)
- Activé sur tous les switches pour les VLANs 10, 20, 30, 40
- Repose sur la base DHCP snooping pour valider les associations IP/MAC
- Résultat : le script arp_poisoning.py est bloqué par le switch

### Part 3 - IP Source Guard
- Activé sur les ports clients avec `ip verify source port-security`
- Bloque tout paquet avec une IP source non autorisée
- Résultat : les paquets avec fausse IP source sont droppés

### Part 4 - Protections protocoles automatiques
- **STP** : PortFast + BPDU Guard activés sur les ports clients
- **CDP** : désactivé sur les ports clients

---

## Fichiers de configuration

| Fichier | Description |
|---------|-------------|
| r1-running-config.txt | Config complète du routeur R1 |
| core1-running-config.txt | Config du switch core |
| access1-running-config.txt | Config access switch 1 |
| access2-running-config.txt | Config access switch 2 |
| access3-running-config.txt | Config access switch 3 |
| access4-running-config.txt | Config access switch 4 |

---

## Scripts Scapy

| Script | Description |
|--------|-------------|
| dhcp_race.py | Serveur DHCP rogue en Scapy |
| dhcp_starvation.py | DHCP starvation |
| arp_poisoning.py | ARP poisoning d'une victime |
| arp_mitm.py | ARP MITM bidirectionnel |
| stp_poisoning.py | Tentative de prise de contrôle STP |

---

## Captures Wireshark

| Fichier | Description |
|---------|-------------|
| p1_routed_ping.pcap | Ping inter-VLAN entre clients et guests |
| p2_no_nat.pcap | Ping vers 1.1.1.1 sans NAT |
| p2_nat.pcap | Ping vers 1.1.1.1 avec NAT |
| p2_routed_ping.pcap | Ping efrei.fr avec résolution DNS |
| p3_dhcp.pcap | Échange DHCP complet |
| p1_dhcp_race.pcap | Course DHCP entre serveur légitime et rogue |
| p1_dhcp_starvation.pcap | DHCP starvation |
| p1_dhcp_dos.pcap | Client DOS après starvation |
| p2_poisoning.pcap | Trames ARP malveillantes |
| p2_mitm.pcap | MITM ARP bidirectionnel |
| p4_dns_spoof.pcap | DNS spoof + ping vers machine attaquante |
