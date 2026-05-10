# new_test — Topologie VPN IPSec bi-sites avec pfSense

Projet GNS3 (v2.2.58.1) simulant une architecture réseau d'entreprise répartie sur deux sites géographiques, reliés par un tunnel IPSec. Chaque site dispose de ses propres équipements utilisateurs, d'un pare-feu pfSense, et d'une DMZ de services (Site A uniquement).

---

## Architecture générale

```
[Internet / NAT]
      |
    [FAI] — Cisco 7200 (routeur WAN)
    /   \
   /     \
pfSense-1  pfSense-2
(Site A)   (Site B)
   |           |
Switch1     Switch2
   |           |
employés     Workers
Directeur
Admin-AIC
   |
Switch3 (DMZ)
   |
Serveur-WEB / Serveur-FTP / SRV-DNS
```

Le routeur FAI interconnecte les deux sites via des liens point à point. Un tunnel **IPSec** est établi entre pfSense-1 et pfSense-2 pour chiffrer le trafic inter-sites.

---

## Plan d'adressage

| Équipement / Réseau | Adresse IP              | Remarque              |
|---------------------|-------------------------|-----------------------|
| LAN Site A          | 192.168.1.0/24          | GW : 192.168.1.1      |
| LAN Site B          | 192.168.20.0/24         | GW : 192.168.20.2     |
| DMZ (Site A)        | 192.168.2.0/24          | –                     |
| WAN pfSense-1       | 84.0.0.1/30             | vers FAI f1/0         |
| WAN pfSense-2       | 85.0.0.1/30             | vers FAI f2/0         |
| employé (VPCS)      | DHCP                    | Switch1               |
| employé1/2/3        | DHCP                    | Switch1               |
| Directeur           | DHCP                    | Switch1               |
| Admin-AIC           | 192.168.1.101/24        | GW : 192.168.1.2      |
| Worker1-6           | 192.168.20.x            | Switch2               |
| Serveur-WEB         | 192.168.2.3/24          | Switch3               |
| SRV-DNS             | 192.168.2.53            | Switch3               |

---

## Inventaire des équipements

### Site A

| Nom        | Type         | Rôle                          | Console |
|------------|--------------|-------------------------------|---------|
| pfSense-1  | QEMU (x86_64) | Pare-feu / VPN endpoint      | VNC     |
| Switch1    | Ethernet switch | Commutateur LAN             | –       |
| Switch3    | Ethernet switch | DMZ                          | –       |
| employé    | VPCS         | Poste utilisateur             | Telnet  |
| employé1   | VPCS         | Poste utilisateur             | Telnet  |
| employé2   | VPCS         | Poste utilisateur             | Telnet  |
| employé3   | VPCS         | Poste utilisateur             | Telnet  |
| Directeur  | VPCS         | Poste direction               | Telnet  |
| Admin-AIC  | VMware       | Poste administrateur          | –       |
| Serveur-WEB | VMware      | Serveur HTTP (DMZ)            | –       |
| Serveur-FTP | VMware      | Serveur FTP (DMZ)             | –       |
| SRV-DNS    | VMware       | Serveur DNS (DMZ)             | –       |

### Site B

| Nom           | Type          | Rôle                   | Console |
|---------------|---------------|------------------------|---------|
| pfSense-2     | QEMU (x86_64) | Pare-feu / VPN endpoint | VNC    |
| Switch2       | Ethernet switch | Commutateur LAN       | –       |
| Worker1-siteB | VPCS          | Poste utilisateur      | Telnet  |
| Worker2       | VPCS          | Poste utilisateur      | Telnet  |
| Worker3       | VPCS          | Poste utilisateur      | Telnet  |
| worker4       | VPCS          | Poste utilisateur      | Telnet  |
| Worker5       | VPCS          | Poste utilisateur      | Telnet  |
| Worker6       | VPCS          | Poste utilisateur      | Telnet  |
| Remote-PC     | VMware        | PC distant             | –       |

### Infrastructure commune

| Nom   | Type      | Rôle                          | Console |
|-------|-----------|-------------------------------|---------|
| FAI   | Dynamips (Cisco 7200) | Routeur WAN / FAI  | Telnet  |
| NAT1  | NAT       | Accès Internet                | –       |

---

## Connectivité (liens)

```
employé      (e0) <-> Switch1     (e2)
employé1     (e0) <-> Switch1     (e4)
employé2     (e0) <-> Switch1     (e5)
employé3     (e0) <-> Switch1     (e6)
Directeur    (e0) <-> Switch1     (e3)
Admin-AIC    (e0) <-> Switch1     (e7)
Switch1      (e0) <-> pfSense-1   (em1)   [LAN]
pfSense-1    (em0)<-> FAI         (f1/0)  [WAN]
pfSense-1    (em2)<-> Switch3     (e0)    [DMZ]
Switch3      (e1) <-> Serveur-WEB (e0)
Switch3      (e3) <-> SRV-DNS     (e0)
Switch3      (e4) <-> Serveur-FTP (e0)
FAI          (f0/0)<-> NAT1       (nat0)  [Internet]
FAI          (f2/0)<-> pfSense-2  (em0)  [WAN]
pfSense-2    (em1)<-> Switch2     (e0)    [LAN]
Switch2      (e1) <-> Worker1-siteB (e0)
Switch2      (e2) <-> Worker2     (e0)
Switch2      (e3) <-> Worker3     (e0)
Switch2      (e4) <-> worker4     (e0)
Switch2      (e5) <-> Worker5     (e0)
Switch2      (e6) <-> Worker6     (e0)
Switch2      (e7) <-> Remote-PC   (e0)
```

---

## Prérequis

### Images requises

| Équipement   | Image                                  |
|--------------|----------------------------------------|
| pfSense-1/2  | `pfSense-CE-2.7.2-RELEASE-amd64.iso`  |
| pfSense disk | `empty100G.qcow2`                      |
| FAI          | `cisco-7200.gns3a` (Dynamips IOS)      |
| Serveurs (VMware) | Image VMware selon configuration  |

### Ressources recommandées

| Ressource | Valeur minimale |
|-----------|----------------|
| RAM pfSense | 2048 Mo (× 2) |
| RAM FAI (c7200) | 512 Mo |
| GNS3 | ≥ 2.2.58 |
| GNS3 VM | Recommandée pour QEMU/VMware |

---

## Démarrage du projet

1. Ouvrir GNS3 et importer `new_test.gns3`.
2. Vérifier que les images pfSense et Cisco 7200 sont bien référencées dans les préférences.
3. Démarrer les nœuds dans l'ordre suivant :
   - FAI → pfSense-1 → pfSense-2 → Switches → VPCS/VMs
4. Accéder aux consoles pfSense en VNC pour configurer le tunnel IPSec (Phase 1 et Phase 2).
5. Tester la connectivité inter-sites avec `ping` depuis un VPCS.

---

## Points de vérification

- Tunnel IPSec Site A ↔ Site B monté (`Status > IPSec` dans pfSense).
- Résolution DNS depuis les postes clients (`nslookup` vers `192.168.2.53`).
- Accès HTTP/FTP aux serveurs de la DMZ depuis les deux sites.
- Isolation du trafic inter-sites chiffré (capture Wireshark sur les liens WAN).
