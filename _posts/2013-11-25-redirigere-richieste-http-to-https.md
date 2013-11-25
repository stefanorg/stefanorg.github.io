---
layout: post
title: Redirigere tutte le richieste http su https tramite Virtual Host
description: "Come redirigere tutte le richieste da http ad https configurando opportunamente il nostro Virtual Host"
category: articles
tags: [https, ssl, apache, zf1, linux, centos]
comments: true
---

Su una applicazione sviluppata con Zend Framework 1, ho avuto la necessità di redirigere tutte le richieste http ad https. 
Nel mio caso, avendo accesso alla configurazione del server, posso modificare il Virtual Host.

> Per abilitare il protocollo http su apache potete dare un'occhiata a questo mio altro post: [Abilitare Apache SSL su OSX Lion]({% post_url 2013-11-23-abilitare-apache-ssl-osx-lion %})

L'applicazione sviluppata con ZendFramework 1 è ospitata su un server CentOS 6.3 con webserver apache2.

Modifico il file di configurazione del Virtual Host che si trova in `/etc/httpd/conf.d/ilmiosito.conf` in questo modo

{% highlight bash %}
#configurazione vhost porta 80
<VirtualHost ilmiosito.local:80>
        RewriteEngine on
        ReWriteCond %{SERVER_PORT} !^443$
        #tutte le richieste http le redirigo su https
        RewriteRule ^/(.*) https://%{HTTP_HOST}/$1 [NC,R,L]
</VirtualHost>

#configurazione vhost porta 443 https
<VirtualHost ilmiosito.local:443>
   DocumentRoot "/var/www/ilmiosito/public"
   ServerName ilmiosito.local

   RewriteEngine on
   RewriteCond %{HTTP_HOST} ^ilmisito$ [NC]
   RewriteRule ^(.*)$ https://ilmiosito.local$1 [R=301,L]

   # This should be omitted in the production environment
   SetEnv APPLICATION_ENV production

   <Directory "/var/www/ilmiosito/public">
       Options Indexes MultiViews FollowSymLinks
       AllowOverride All
       Order allow,deny
       Allow from all
   </Directory>

  ErrorLog logs/ssl/ilmiosito-error.log
  CustomLog logs/ssl/ilmiosito-access.log combined
  SSLEngine on
  SSLCertificateFile /etc/httpd/ssl/crt/ilmiosito.crt
  SSLCertificateKeyFile /etc/httpd/ssl/key/ilmisito.key
</VirtualHost>
{% endhighlight %}

