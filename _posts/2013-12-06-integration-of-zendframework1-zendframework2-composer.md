---
layout: post
title: Integrare composer.phar e zendframework2 in un progetto zendframework1 
description: "In un applicazione sviluppata con zf1 ho voluto integrare composer.phar come gestore delle dipendenze, inoltre ho voluto integrare zf2 in modo da utilizzare in parallelo zf1 e zf2"
category: articles
tags: [php, zf1, zf2, composer]
comments: true
---

In un applicazione sviluppata con zf1 ho voluto integrare composer.phar come gestore delle dipendenze, inoltre ho voluto integrare zf2 in modo da utilizzare in parallelo zf1 e zf2 ecco come:

## Integrazione di composer come gestore delle dipendenze in zf1
Diciamo che la nostra applicazione zf1 sia situata nella directory in `/home/devplayground/project/zf1app`

### Download composer
Per prima cosa è necessario scaricare la libreria composer, direttamente dalla pagina ufficiale scegliete il metodo che preferite o [cliccate qui](http://getcomposer.org/composer.phar) per scaricarlo direttamente.
Una volta scaricato copiatelo dentro la directory del progetto `/home/devplayground/project/zf1app`

### Integrazione di composer in una applicazione ZF1
Adesso bisogna inizializzare composer e rispondere ad una serie di domande, l'inizializzazione serve a creare il file di configurazione di composer `composer.json` che somiglierà più o meno a questo:

{% highlight bash %}
{
    "name": "devplayground/zf1app",
    "require": {
        "zendframework/zendframework1": "1.*"
    },
    "authors": [
        {
            "name": "Stefano Corallo",
            "email": ""
        }
    ],
    "minimum-stability": "dev"
}
{% endhighlight %}

Per inizializzare composer eseguiamo il comando init e rispondiamo alle domande:
{% highlight bash %}
 [devplayground@localhost]$ cd /home/devplayground/project/zf1app
 [devplayground@localhost]$ php composer.phar init

{% endhighlight %}

finche non ci viene richiesto se vogliamo specificare delle dipendenze, quindi inseriamo zf1 nel seguente modo

{% highlight bash %}
...
Define your dependencies.

Would you like to define your dependencies (require) interactively [yes]? 
Search for a package []: zendframework/zendframework1

Found 15 packages matching zendframework/zendframework1

   [0] zendframework/zendframework1
   [1] bombayworks/zendframework1
   [2] zendframework/zendframework
   [3] emagister/zendframework1
   [4] guilhermeblanco/zendframework1-doctrine2
   [5] zendframework/zftool
   [6] zendframework/zendpdf
   [7] zendframework/zend-stdlib
   [8] zendframework/zend-eventmanager
   [9] zendframework/zend-servicemanager
  [10] zendframework/zend-validator
  [11] zendframework/zend-loader
  [12] zendframework/zend-code
  [13] zendframework/zend-math
  [14] zendframework/zend-developer-tools

Enter package # to add, or the complete package name if it is not listed []: 0
Enter the version constraint to require []: 1.*
Search for a package []:
Would you like to define your dev dependencies (require-dev) interactively [yes]? no
{% endhighlight %}

A questo punto il file `composer.json` è stato creato adesso scarichiamo le nostre librerie tramite il comdando `install`

{% highlight bash %}
[devplayground@localhost]$ php composer.phar install
{% endhighlight %}

Quando composer avrà finito di scaricare la libreria zendframework1 la troveremo nella directory `vendor` all'interno del nostro progetto, la struttura sarà più o meno cosi:

{% highlight bash %}
[devplayground@localhost]$ tree . -L 1
|-- application
|-- bin
|-- Changelog
|-- composer.json
|-- composer.lock
|-- composer.phar
|-- library
|-- public
`-- vendor
    |-- autoload.php
    |-- bin
    |-- composer
    `-- zendframework

{% endhighlight %}


### Verifica Funzionamento

Adesso che abbiamo scaricato tramite composer la nostra libreria zendframework1, dobbiamo modificare l'applicazione zf1app in maniera tale che sia in grado di caricare la libreria, per farlo modifichiamo il file `public/index.php` in questo modo

Prima:
{% highlight php %}
...
...
set_include_path(implode(PATH_SEPARATOR, array(
    realpath(APPLICATION_PATH . '/../library'),
    get_include_path(),
)));
...
...
{% endhighlight %}

Dopo:
{% highlight php %}
...
...
set_include_path(implode(PATH_SEPARATOR, array(
    realpath(APPLICATION_PATH . '/../library'),
    get_include_path(),
)));
//Composer Class loading 
define('VENDOR_PATH', APPLICATION_PATH . '/../vendor');
require_once VENDOR_PATH . '/autoload.php';
...
...
{% endhighlight %}

A questo punto non è più necessaria la libreria di Zend installata in `library/Zend` perchè viene caricata quella installata tramite composer quindi possiamo eliminarla

{% highlight bash %}
[devplayground@localhost]$ cd /home/devplayground/project/zf1app/library
[devplayground@localhost]$ rm -rf Zend
{% endhighlight %}


## Integrazione di ZF2 in una applicazione ZF1

Adesso per scaricare zf2 ed integrarlo nel nostro progetto zf1 è semplicissimo, utilizziamo composer

{% highlight bash %}
[devplayground@localhost]$ cd /home/devplayground/project/zf1app
[devplayground@localhost]$ composer require zendframework/zendframework:2.2.*
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
  - Installing zendframework/zendframework (dev-master 55d43c9)
    Cloning 55d43c9c5cc7dcb698467eae42c1309fcbe66312

Writing lock file
Generating autoload files
{% endhighlight %}

A questo punto abbiamo installato la libreria zf2, precisamente si trova in `vendor/zendframework/zendframework`

> La libreria viene già caricata in quanto abbiamo già modificato il file `public/index.php` precedentemente inserendo il classloading di composer.

A questo punto se volessimo utilizzare la Dependency Injection, detto fatto, ad esempio modifichiamo il file `public/index.php` subito dopo il class loading di composer:

{% highlight bash %}
//Composer Class loading 
define('VENDOR_PATH', APPLICATION_PATH . '/../vendor');
require_once VENDOR_PATH . '/autoload.php';

$di = new \Zend\Di\Di(); 
var_dump($di);
die();
{% endhighlight %}