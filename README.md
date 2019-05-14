# Installation & Configuration d'un serveur DNS - Bind9

Ici, vous verrez comment mettre en place et configurer un serveur DNS. 

`DNS` est l’acronyme de Domain Name Service, ou « Système de Nom de Domaine » en Français. Il s’agit d’un service informatique réseau permettant de faire correspondre un nom de domaine ou URL à une adresse IP.

Un nom de domaine, pour faire simple, est la partie d’une URL (adresse internet ou email), qui va vous renvoyer vers un ordinateur en particulier, dans ce cas on appelle cet ordinateur, un serveur.

## Pré-requis

On considère que :  
**MyHostname** = ghost  
**Mydomain** = famous.com  
**My@ip** = 192.168.1.1  
**Un serveur mail** = 192.168.1.2  
**Un serveur web** = 192.168.1.3

### Preconf du serveur
Vous devez tout d'abord definir un nom de machine en modifiant le fichier hostname

**hostnamectl set-hostname `ghost`**

Vous devez par la suite modifier votre fichier hosts pour permettre une résolution locale des noms à partir des adresses IP.  Cela se fait dans le fichier : `/etc/hosts`

```
cat <<-EOF > /etc/hosts
127.0.0.1	localhost
127.0.1.1	ghost
192.168.1.1	famous.com	ghost.famous.com

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
EOF
```

Il peut être nécéssaire de modifier le fichier `/etc/resolv.conf` afin que notre machine soit directement intégrée dans la zone DNS.

```
cat <<-EOF >> /etc/resolv.conf
search famous.com
domain famous.com
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
	- la zone domaine : `famous.com`  
	- la zone inverse associée : `1.168.192.in-addr.arpa` Cette dernière permet de traduire vos adresses Ip en noms de domaines.

```
cat <<-EOF > etc/bind/named.conf.local
zone "famous.com" {
    type master;
    file "/etc/bind/zones/db.famous.com";
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.1.168.192"; 
};
EOF
```

Il vous faudra alors créer ces 02 fichiers de zones et les éditer comme suit :

`/etc/bind/db.famous.com`
```
$TTL    604800

@       IN      SOA     ghost.famous.com. root.famous.com. (
                              6         ; Serial
                         604820         ; Refresh
                          86600         ; Retry
                        2419600         ; Expire
                         604600 )       ; Negative Cache TTL
;
@       IN      NS      ghost.famous.com.

ghost      A      192.168.10.2
```


