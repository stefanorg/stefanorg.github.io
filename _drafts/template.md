---
layout: post
title: TEMPLATE Macports Abilitare Apache SSL su Mac OSX Lion
description: "TEMPLATE Macports Apache SSL su Mac OSX Lion"
category: articles
tags: [https, ssl, apache, osx, macports]
comments: true
---

Dopo aver installato il server web apache, php54, mysql tramite macport (si trovano parecchie guide online basta googlare un pochino).

Spostiamoci nella directory di installazione di apache

{% highlight bash %}
(n4z4@n4z4-mb) - (16:54) - (~)
-=>>cd /opt/local/apache2
{% endhighlight %}

### Generiamo le chiavi rsa per creare i certificati da utilizzare con apache

Per prima cosa creiamo la directory dove genere le chiavi

{% highlight bash %}
(n4z4@n4z4-mb) - (16:54) - (/opt/local/apache2)
-=>>sudo mkdir ssl
Password:
{% endhighlight %}

Per generare le chiavi usiamo il seguente comando, senza inserire nessuna passphrase in modo tale che all'avvio di apache il servizio non si blocchi.

{% highlight bash %}
sudo ssh-keygen -f ssl.key
Password:
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in ssl.key.
Your public key has been saved in ssl.key.pub.
The key fingerprint is:
19:c2:5b:ff:80:4b:30:77:96:72:c0:4c:39:ce:0c:f6 root@n4z4-mb.local
The key's randomart image is:
+--[ RSA 2048]----+
|       +o.       |
|     .o =. .     |
|     .=*=.=      |
|       BEX       |
|      . S o      |
|       . . o     |
|        .   .    |
|                 |
|                 |
+-----------------+
{% endhighlight %}


Sono state genere 2 chiavi, chiave pubblica `ssl.key.pub` e chiave privata `ssl.key` :
{% highlight bash %}
(n4z4@n4z4-mb) - (17:04) - (/opt/local/apache2/ssl)
-=>>ls
ssl.key     ssl.key.pub
{% endhighlight %}

### Generiamo il file per il certificato di richiesta request.csr

Dobbiamo generare il certificato di richiesta `request.csr` che ci servirà per creare il certificato SSL. 
Per generare il certificato utilizziamo la chiave privata creata precedentemente. Dopo aver lanciato il comando ci verranno chieste alcune informazioni per la creazione del certificato di richiesta, ad esempio come CN mettete il vostro `ServerName` nel mio caso `localhost` 

{% highlight bash %}
(n4z4@n4z4-mb) - (17:16) - (/opt/local/apache2/ssl)
-=>>sudo openssl req -new -key ssl.key -out request.csr
{% endhighlight %}

Dopo aver creato il certificato di richiesta possiamo generare il nostro certificato SSL 

{% highlight bash %}
(n4z4@n4z4-mb) - (17:28) - (/opt/local/apache2/ssl)
-=>>sudo openssl x509 -req -days 365 -in request.csr -signkey ssl.key -out server.crt
Signature ok
subject=........
Getting Private key
{% endhighlight %}
Adesso che abbiamo generato il nostro certificato Self Signed `server.crt`, non ci resta che configurare apache per servire le richieste `https`.

### Configuriamo Apache

Spostiamoci nella directory `/opt/local/apache2/conf/extra` modifichiamo il file di configurazione del modulo ssl di apache `httpd-ssl.conf` cerchiamo le linee che iniziano con SSLCertificateFile, SSLCertificateKeyFile e modifichiamole come segue:

{% highlight bash %}
SSLCertificateFile "/opt/local/apache2/ssl/server.crt" #il nostro certificato ssl
SSLCertificateKeyFile "/opt/local/apache2/ssl/ssl.key" #la nostra chiave privata
{% endhighlight %}

Controllate che nello stesso file siano commentate (inizino con #) le linee che iniziano con SSLCACertificatePath, SSLCARevocationPath

Verifichiamo che nel file di configurazione di apache `/opt/local/apache2/httpd.conf` sia abilitata la sezione ssl e in caso abilitiamola rimuovendo il commento # come segue:
{% highlight bash %}
# Secure (SSL/TLS) connections
Include conf/extra/httpd-ssl.conf
{% endhighlight %}

### Configuriamo il vhost
Verifichiamo che nel file di configurazione di apache `/opt/local/apache2/httpd.conf` sia abilitata la sezione Virtual Host e in caso abilitiamola rimuovendo il commento # come segue:

{% highlight bash %}
# Virtual hosts
Include conf/extra/httpd-vhosts.conf
{% endhighlight %}

Adesso modifichiamo il virtual host '/opt/local/apache2/extra/httpd-vhosts.conf' aggiungiamo sotto NameVirtualHost *:80 la riga NameVirtualHost *:443 in questo modo

{% highlight bash %}
NameVirtualHost *:80
NameVirtualHost *:443
{% endhighlight %}

Ora possiamo configurare un virtual host ssl come segue:

{% highlight bash %}
<VirtualHost *:443>
    SSLEngine on
    SSLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
    SSLCertificateFile /private/etc/apache2/ssl/server.crt
    SSLCertificateKeyFile /private/etc/apache2/ssl/ssl.key
    ServerName localhost #Questo e' lo stesso del CN utilizzato durante la generazione del certificato di richiesta request.csr
    DocumentRoot "/path/al/mio/sito/web"
</VirtualHost>
{% endhighlight %}

Adesso possiamo riavviare apache e collegarci al nostro nuovo sito in HTTPS.

{% highlight bash %}
(n4z4@n4z4-mb) - (17:32) - (~/apache2-server/conf)
-=>>sudo /opt/local/etc/LaunchDaemons/org.macports.apache2/apache2.wrapper restart
Password:
{% endhighlight %}