---
layout: post
title: Redirigere le richieste http ad https con JHttps
description: "JHttps è un modulo per zendframework2 che permette facilmente di configurare le rotte che vogliamo redirigire su https senza aver bisogno di modificare il virtual-host di apache o il file .htaccess"
category: articles
tags: [https, ssl, zf2, library, module]
comments: true
---

[JHttps](https://github.com/stefanorg/JHttps) è un modulo per zendframework2 che permette facilmente di configurare le rotte che vogliamo redirigire su https senza aver bisogno di modificare il virtual-host di apache o il file .htaccess

### Installazione

Scarichiamo la libreria tramite composer `php composer.phar require stefanorg/jhttps:dev-master`

### Configurazione

Una volta scaricata la libreria abilitiamola modificando il file `config/application.config.php`

{% highlight php %}
	...
	'modules' => array(
        'Application',
        'JHttps'
    ),
    ...
{% endhighlight %}

Copiamo il file `vendor/stefanorg/jhttps/config/jhttps.config.global.php.dist` in `config/autoload/jhttps.config.global.php`

Adesso possiamo abilitare le rotte inserendole in 'routes', ad esempio abilito https per la pagina di login dell'utente:

{% highlight php %}
<?php
$config = array(
    'jhttps' => array(
    	/**
         * If you want reset to http scheme for non https route
         */
        'force_http_for_non_https_route' => true,
        'routes' => array(
            //enable https for user login page
            'zfcuser/login'
        )
    )
);
return $config;
{% endhighlight %}
