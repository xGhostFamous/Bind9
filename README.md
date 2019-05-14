#Installation & Configuration d'un serveur DNS - Bind9

Ici, vous verrez comment mettre en place et configurer un serveur DNS. 

`DNS` est l’acronyme de Domain Name Service, ou « Système de Nom de Domaine » en Français. Il s’agit d’un service informatique réseau permettant de faire correspondre un nom de domaine ou URL à une adresse IP.

Un nom de domaine, pour faire simple, est la partie d’une URL (adresse internet ou email), qui va vous renvoyer vers un ordinateur en particulier, dans ce cas on appelle cet ordinateur, un serveur.

###Pré-recquis

###Preconf du serveur
Vous devez tout d'abord definir un nom de machine en modifiant le fichier hostname

**hostnamectl set-hostname `urhostname`**

Vous devez par la suite modifier votre fichier hosts pour permettre une résolution locale des noms à partir des adresses IP. 
Cela se fait dans le fichier : `/etc/hosts`

```
cat <<-EOF > /etc/hosts
127.0.0.1	localhost
127.0.1.1	`urhostname`
X.X.X.X`(ur_ip)`	`urdomain`	`urhostname.urdomain`

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
EOF
```

Il peut être nécéssaire de modifier le fichier `/etc/resolv.conf` afin que notre machine soit directement intégrée dans la zone DNS.

```
cat <<-EOF >> /etc/resolv.conf
search `urdomain`
domain `urdomain`
nameserver `@ip_dnsServer`
EOF
```
