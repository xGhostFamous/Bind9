# Installation & Configuration d'un serveur DNS - Bind9

Ici, vous verrez comment mettre en place et configurer un serveur DNS. 

`DNS` est l’acronyme de Domain Name Service, ou « Système de Nom de Domaine » en Français. Il s’agit d’un service informatique réseau permettant de faire correspondre un nom de domaine ou URL à une adresse IP.

Un nom de domaine, pour faire simple, est la partie d’une URL (adresse internet ou email), qui va vous renvoyer vers un ordinateur en particulier, dans ce cas on appelle cet ordinateur, un serveur.

#### Pourquoi installer un serveur DNS

Pour au moins trois raisons :
- Éviter de tenir à jour la table hosts de chaque poste client d’un réseau.
- Avoir un cache DNS qui accélère la recherche des noms.
- Sur un réseau locale, un serveur DNS permet d’accélérer le trafic sur le réseau car de nombreux services ont besoins d’un serveur DNS bien configuré pour fonctionner correctement (WEB, POP, SMTP,..) 

## Pré-requis

On considère que :  
**MyHostname** = ghost  
**Mydomain** = mondomaine.com  
**My@ip** = 192.168.1.1  
**Un serveur mail** = 192.168.1.2  
**Un serveur web** = 192.168.1.3  
**Une machine cliente** = 192.168.1.4

### Preconf du serveur
Vous devez tout d'abord definir un nom de machine en modifiant le fichier hostname

**hostnamectl set-hostname `ghost`**

Vous devez par la suite modifier votre fichier hosts pour permettre une résolution locale des noms à partir des adresses IP.  Cela se fait dans le fichier : `/etc/hosts`

```
cat <<-EOF > /etc/hosts
127.0.0.1	localhost
127.0.1.1	ghost
192.168.1.1	mondomaine.com	ghost.mondomaine.com

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
EOF
```

Il peut être nécéssaire de modifier le fichier `/etc/resolv.conf` afin que notre machine soit directement intégrée dans la zone DNS.

```
cat <<-EOF >> /etc/resolv.conf
search mondomaine.com
domain mondomaine.com
nameserver 192.168.1.1
EOF
```

### Installation de Bind9

L'installation du service est classique, le nom du paquet à installer est : `bind9` 

**apt-get install -y bind9**

### Configuration de Bind9

Les fichiers de configuration se trouvent sous : `/etc/bind/`  
Le fichier principal de configuration de `bind9` est le fichier `named.conf.local`.  
Dans ce dernier, vous pourrez configurer vos zones;  
02 zones sont à déclarer :  
- la zone domaine : `mondomaine.com`  
- la zone inverse associée : `1.168.192.in-addr.arpa` Cette dernière permet de vérifier qu'un nom est valide (Trouver l’adresse IP à partir du nom). `Vous noterez que l'adresse ip est indiquée à l'envers, on appelle ça la résolution inverse`.

```
cat <<-EOF > etc/bind/named.conf.local
zone "mondomaine.com" {
    type master;
    file "/etc/bind/zones/db.mondomaine.com";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.1.168.192"; 
};
EOF
```
*type master : Cette ligne indique que le serveur est le serveur principal de ce domaine.


Il vous faudra alors créer ces 02 fichiers de zones et les éditer comme suit :  
- le fichier de zone `db.mondomaine.com`
```
cat <<-EOF > /etc/bind/db.mondomaine.com
$TTL    604800

@       IN      SOA     ghost.mondomaine.com. root.mondomaine.com. (
                              6         ; Serial
                         604820         ; Refresh
                          86600         ; Retry
                        2419600         ; Expire
                         604600 )       ; Negative Cache TTL

; les lignes suivantes permettent au serveur de se retrouver lui même
@       IN      NS      ghost.mondomaine.com.
ghost      A      192.168.1.1

; la ligne suivante permet d'identifier le serveur mail -- on lui donne une priorité de 10
@       IN      MX      10      mail.mondomaine.com. 

; les lignes suivantes définissent la table entre les noms et les IP
mail       A      192.168.1.2
www        A      192.168.1.3
cli        A      192.168.1.4

; les lignes suivantes sont des alias entre des noms et des autres noms
mail            CNAME   mail1
cli             CNAME   client
www             CNAME   web
ghost           CNAME   dns
EOF
```
- le fichier de zone `db.1.168.192`
```
cat <<-EOF > /etc/bind/db.1.168.192
$TTL    604800

@       IN      SOA     ghost.mondomaine.com. root.mondomaine.com. (
                             21         ; Serial
                         604820         ; Refresh
                          864500        ; Retry
                        2419270         ; Expire
                         604880 )       ; Negative Cache TTL

;
@       IN      NS      ghost.mondomaine.com.
@       IN      MX      10      mail.mondomaine.com. ;10 is the priority
;
$ORIGIN 1.168.192.in-addr.arpa.
1       IN      PTR     ghost.mondomaine.com.
2       IN      PTR     mail.mondomaine.com.
3       IN      PTR     www.mondomaine.com.
4       IN      PTR     cli.mondomaine.com.
EOF
```

Avant de redémarrer le serveur, nous allons tester le fichier de domaine créé pour vérifier s’il est correcte afin d’éviter des erreurs au redémarrage de bind.  
La commande named-checkzone, inclue dans le package de bind9, va vérifier la syntaxe du fichier passé en paramètre.  
**named-checkzone mondomaine.com /etc/bind/db.mondomaine.com**  

Il faut maintenant redémarrer le service pour prendre en compte les modifications.  
**systemctl restart bind9**

#### Quelques explications 

Il existe différents types d’enregistrements représentant chacun un type information différent. Voici une liste des plus courants :
- Enregistrement A : C’est l’enregistrement le plus courant. Il fait correspondre une adresse IPv4 à un nom d’hôte.
- Enregistrement AAAA : Variante de l’enregistrement A, il fait correspondre une adresse IPv6 à un nom d’hôte.
- Enregistrement CNAME (Canonical Name) : Il permet de créer un alias pointant vers un autre enregistrement du domaine courant ou d’un domaine externe.
- Enregistrement MX (Mail Exchange) : Il donne le serveur d’envoi d’emails. Cet enregistrement doit pointer obligatoirement vers un enregistrement de type A.
- Enregistrement NS (Name Server) : Il définit les serveurs DNS du domaine. Cet enregistrement doit pointer obligatoirement vers un enregistrement de type A.
- Enregistrement TXT : Il permet de définir un enregistrement contenant un texte libre. Cet enregistrement est notamment utilisé pour confirmer le détenteur du domaine pour pouvoir utiliser certains services externes tel que Google Webmaster tools ou encore un service d’envoi de mails (Mandrill, Mailgun, …).
