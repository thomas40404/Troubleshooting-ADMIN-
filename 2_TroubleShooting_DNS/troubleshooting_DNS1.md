# Exercices de troubleshooting DHCP 2

- **Auteur(s)** : Charlier Thomas  
- **Date** : 25/11/2025  
*GNS3 n’ayant pas accès à internet, je considérerai qu’une trace Wireshark vers le routeur est un accès internet réussi.*
---

## 1. Bug Report

Je travaille sur un réseau complet contenant un serveur DHCP utilisant dhcpd, un serveur DNS résolveur, un NS interne à l’entreprise, ainsi que deux clients DHCP : Direction et Atelier.

Le client me dit avoir des problèmes de connexion lorsqu’il est connecté dans l’entreprise.

---

## 2. Collecte des symptômes

Depuis le PC Direction, un ping vers 1.1.1.1 réussit car une trace vers le routeur est reçue (rappel : la VM n’a pas d’accès à internet, un ping ne peut donc pas réellement réussir).

Depuis le PC Direction, je ping google.com : ma trace vers le résolveur est bien reçue mais le résolveur répond “Refused”. ![capture](/2_TroubleShooting_DNS/capture_refused.heic)

Je regarde l’IP du PC Direction et vois : 192.168.0.10.

### Liste des outils utilisés

Les outils utilisés pour cette analyse sont :
Wireshark,  
netstat,  
ping,  
ip a,  
named -g.

---

## 3. Identification et description du problème

Le problème observé est que le résolveur refuse les demandes des clients. Il semblerait que le résolveur n’accepte pas les requêtes provenant du range IP de la machine.

---

## 4. Proposition de solution

Modifier le range pour que le résolveur accepte les requêtes des clients DNS. Il faut donc aller dans /etc/bind/named.conf pour vérifier cela.  
[config avant résolution](/2_TroubleShooting_DNS/config1.heic)

Nous pouvons voir :

```
allow-recursion {
    192.168.1.0/24;  --> tous les 192.168.1.
    127.0.0.1/32;    --> pour lui-même
};
```

Mais notre client DNS a comme IP 192.168.0.10, il n’est donc effectivement pas dans le bon range d’IP.  
Nous pouvons soit aller dans le serveur DHCP pour vérifier les IP qu’il attribue et modifier le serveur DNS pour que cela corresponde exactement, soit mettre 192.168.0.0/18 pour que tout potentiel sous-réseau soit pris en compte.

Je choisis la solution la plus propre et modifie donc le résolveur comme suit :

```
allow-recursion {
    192.168.0.0/24;  --> tous les 192.168.0.
    127.0.0.1/32;    --> pour lui-même
};
```
Cela est plus propre car cela n’accepte que les 254 adresses du réseau réel, et non des milliers de machines inexistantes qui pourraient potentiellement être utilisées par des personnes malveillantes.
L’IP 192.168.0.10 est donc maintenant prise en compte dans la plage d’IP.

### Validation

Je réessaye après le changement et constate que le serveur envoie une requête vers le routeur, ce qui prouve qu’il a accepté la demande (le routeur répond mal car il n’a pas de connexion internet et ne peut donc pas répondre).  
[trace après résolution](/2_TroubleShooting_DNS/traceFIn.heic)

Pour m’en assurer, je ping **www.woodytoys.lab**, et le ping fonctionne bel et bien, ce qui prouve le bon fonctionnement du résolveur et du SOA.  
![capture](/2_TroubleShooting_DNS/ping%20.HEIC)
