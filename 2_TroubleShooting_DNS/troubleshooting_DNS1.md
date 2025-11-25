# Exercices de troubleshooting DHCP 2

- **Auteur(s)** : Charlier Thomas  
- **Date** : 25/11/2025
*GNS3 n'ayant pas acces a internet, je considérerais qu'une trace whireshark vers le router est un acces internet réussi*
---

## 1. Bug Report

Je travaille sur un réseau complet contenant un serveur DHCP utilisant dhcpd, un serveur DNS résolver, un NS server interne a l'entreprise, ainsi que deux clients DHCP : Direction et Atelier.  

Les utilisateurs indiquent que depuis des modifications, seul un client DHCP peut se connecter à Internet à la fois, et parfois le poste pouvant se connecter change. Avant cela, tous les clients DHCP pouvaient accéder à Internet simultanément.  

---

## 2. Collecte des symptômes

Depuis le PC Direction, un ping vers 1.1.1.1 échoue. La commande ip a montre que Direction n'a pas d'adresse IP.  

Depuis le PC Atelier, le ping fonctionne et l'adresse IP est 192.168.0.10. Après redémarrage d'Atelier, le ping fonctionne toujours et l'adresse IP reste 192.168.0.10.  

Après un certain temps, le PC Direction peut maintenant se connecter et obtient également l'adresse IP 192.168.0.10. Normalement, les deux PC devraient pouvoir se connecter simultanément et avoir des adresses IP différentes.  

Une capture Wireshark montre que les PC sans IP effectuent des DHCPDISCOVER sans obtenir de réponse. Parfois, les deux PC récupèrent l'IP .10 au même moment mais restent incapables de se connecter.  

### Liste des outils utilisés

Les outils utilisés pour cette analyse sont Wireshark, ping, ip a et dhcpd en mode debug.  

---

## 3. Identification et description du problème

Le problème observé est que seulement un client DHCP peut se connecter à Internet à la fois. Tous les clients fonctionnels reçoivent l'IP 192.168.0.10, les autres restent bloqués en DHCPDISCOVER.  

L'hypothèse est que la configuration DHCP a été modifiée et que seule l'IP .10 est disponible. Le premier client arrivé sur le réseau obtient cette IP jusqu'à la fin de son bail DHCP.  

Il semble que le fichier de configuration DHCPD ne permette que de récupérer l'IP .10.  

---

## 4. Proposition de solution

La solution consiste à modifier le fichier de configuration DHCPD situé dans /etc/dhcp/dhcpd.conf. Le range d'adresses était défini de 192.168.0.10 à 192.168.0.10, limitant ainsi le serveur à fournir une seule IP.  

Il faut élargir ce range pour permettre la distribution d'adresses supplémentaires, par exemple de 192.168.0.10 à 192.168.0.100. Cela permettra au serveur de fournir jusqu'à 90 IP différentes, permettant ainsi à 90 PC configurés en DHCP de se connecter sur le réseau interne.  

Après modification du fichier, il faut redémarrer le service DHCP pour appliquer la configuration.  

### Validation

Pour valider la solution, il faut renouveler l'adresse IP sur chaque client, vérifier avec ip a que chaque machine reçoit une IP unique dans le nouveau range et tester la connectivité Internet avec un ping vers 1.1.1.1.  

Après correction, le PC Direction obtient par exemple l'adresse IP 192.168.0.11 et le PC Atelier l'adresse IP 192.168.0.10. Les deux peuvent maintenant se connecter simultanément sans conflit d'IP, et aucun conflit n'est détecté sur le réseau avec Wireshark.  
