---
layout: page
title: Alcuni consigli per scrivere meglio i moduli per Zend Framework 2
description: "Come scrivere dei moduli per zend framework 2 seguendo le best practice"
category: articles
tags: [zf2, module]
comments: true
---

>Questo articolo è una traduzione dell'ottimo articolo di Michaël Gallego: [Some tips to write better Zend Framework 2 modules](http://www.michaelgallego.fr/blog/2013/01/21/some-tips-to-write-better-zend-framework-2-modules)

# C'è già un modulo!!
Prima di scrivere il tuo modulo, controlla se qualcuno ha già scritto un modulo simile! Invece di creare un nuovo modulo, aiuta a migliorare quello esistente inviando delle pull request. Puoi fare una ricerca su [ZF2.modules](http://modules.zendframework.com/) per parola chiave.

Per approfondire leggi anche l'[articolo di Martin Shwalbe sui moduli](http://hounddog.github.com/blog/there-is-a-module-for-that/) .

# Segui le convenzioni
Zend Framework 2, come ogni framework moderno, ha delle convenzioni rigorose riguardo il codice. Ovviamente il framework rispetta le convenzioni PSR-0/1/2, e anche tu dovresti. Inoltre ci sono altre convenzioni standard su dove mettere alcuni file.

Per esempio, molti moduli sono configurabili. Dato che ci sono molte opzioni, è buona idea centralizzare queste opzioni in una classe php (che estende Zend\Stdlib\AbstractOptions). E' pratica comune salvare tutti queste classi in una directory “Options” cosi che ogni option class ha il proprio namespace. Certamente potete mettere tutte queste classi dove volete, ma facendo cosi è piu semplice per gli sviluppatori orientarsi nel vostro codice.

Inoltre, molti sviluppatori mettono tanti file nella stessa directory solo per evitare di creare una nuova directory. `Seriamente, create nuove directory`. Non fa male a nessuno e aiuta le persone a trovare i file che cercano. Ad esempio, se create un Paginator adapter, semplicemente seguite la struttura di ZF 2 e salvatelo in `MioModulo\Paginator\Adapter\MioAdapter`, non in `MioModulo\Form\Adapter\MioAdapter`, solo perchè siete troppo pigri per creare una nuova directory !!

Come esempio, guardate le opzioni del modulo [DoctrineModule](https://github.com/doctrine/DoctrineModule/tree/master/src/DoctrineModule/Options)

# Dai la possibilità di configurare tutto ciò che è necessario sia configurabile
I moduli sono fatti per essere condivisi e riutilizzati da altri. Le persone non sempre hanno esattamente le stesse necessità che hai tu. Ogni volta che una cosa "potrebbe" essere modificata da altri, aggiungi opzioni per consentirne la modifica.

Per esempio, se il tuo moduo genera qualche pagina HTML con dello stile copiato da un file CSS rilasciato nel tuo modulo ... semplicemente dai la possibilità agli altri di poter impostare il path al proprio file CSS (in questo modo non dovranno modificare manualmente il TUO file contenuto nel modulo ma useranno il loro).

Quando rilasci il tuo modulo, dovresti rilasciare due file nella directory config. Uno chiamato “module.config.php” che fornisce configurazioni “standard” e che non dovrebbero essere modificate dall'utente e un ALTRO file chiamato “mio_modulo.global.php.dist” che contiene tutte le opzioni che possono essere modificate dall'utente (questo file tipicamente viene copiato nella directory config/autoload della tua applicazione).

Un grande esempio è il [file di configurazione del modulo ZfcUser](https://github.com/ZF-Commons/ZfcUser/blob/master/config/zfcuser.global.php.dist). Come puoi vedere, quasi tutto può essere modificato, per questo il modulo è molto flessibile.

# Imposta le dipendenze nel costruttore, evita initializers e setter/getter
Il service locator di Zend Framework 2 ci permette davvero di gestire le dipendenze in maniera elegante, il mocking... è uno strumento potente che non è difficile da usare una volta capito. Tuttavia, ZF 2 è stato rilasciato con qualcosa chiamato `initializers`. Sono sicuro che li hai già incontrati. Per esempio, dove una classe implementa le interfacce ServiceLocatorAwareInterface o EventManagerAwareInterface ... il service manager automaticamente setta quelle dipendenze (tramite setter).

Questo può essere interessante, però ha un problema che può portare a un sacco di guai e ad un cattivo uso da parte degli utilizzatori del tuo modulo. Per esempio, nel modulo ZfrRest, ci sono classi che vengono usate per fare il parsing HTTP degli oggetti request/response. Tutti questi parser estendono una classe astratta chiamata AbstractParser.

Come esempio qui c'è il costruttore :
{% highlight php %}
public function __construct(DecoderPluginManager $pluginManager)
{
    $this->decoderPluginManager = $pluginManager;
}
{% endhighlight %}

Invece del costruttore, avremmo potuto scrivere un setter/getter per settare il DecoderPluginManager e impostare la dipendenza attraverso il service manager or utilizzare l'initializer. MA QUESTO assume che l'utente che utilizza il modulo UTILIZZI il service manager per istanziare gli oggetti.

Finchè un giorno qualcuno crea il parser in questo modo:

{% highlight php %}
$parser = new BodyParser();
$parser->parse($request);
{% endhighlight %}

E boom. Non ha chiamato il setDecoderPluginManager ne ha creato l'istanza tramite service manager. La dipendenza non è impostata correttamente e questo codice lancerà delle eccezioni.

QUESTO PERCHè il DecoderPluginManager è una dipendeza ESSENZIALE del parser (il parser non può funzionare senza il plugin), semplicemente impostate le dipendeze direttamente nel costruttore. Cosi le persone non possono utilizzare la vostra classe in malo modo. Anches se volessero.

# Riduci l'utilizzo delle Closure in favore delle Factory
E' uso comune in ZF 2 gestire le dipendenze tramite l'uso di closure come factories nel metodo getServiceConfig della classe Module.php. Invece, sarebbe meglio usare esplicitamente delle "Factory" class e impostarle nel file module.config.php (nella chiave service_manager). Questo è leggermente più efficiente (perchè le closure vengono istanziate, è più facile creare una stringa che una closure), ti permette di cachare i file di configurazioni (dato che le closure non sono cachabili), e rimuovo tutte le closure dal file Module.php che risulta più leggibile.

{% highlight php %}
return array(
    'service_manager' => array(
        'factories' => array(
            'MyModule\My\Service' => 'MyModule\Service\MyServiceFactory'
        )
    )
);
{% endhighlight %}

# Utilizza il Service Manager più che puoi per impostare le tue dipendenze
Questo consiglio è strettamente correlato al "Dai la possibilità di configurare tutto ciò che è necessario sia configurabile". Certe volte le persone chiedono delle feature davvero strane, tanto da chiederti "perchè vuole quella dannata feature?!?". Certamente tu non vuoi aggiungere questa feature al core del tuo modulo ma almeno puoi rendergli la vita semplice permettendogli di fare l'override di quello che vuole.

E con ZF 2, questo è davvero semplice: crea la maggior parte degli oggetti attraverso il service manager. Ad esempio, in ZfrRest, abbiamo aggiunto listener che forniscono feature utili in un contesto REST.
Il primo modo per fare questo è :
{% highlight php %}
public function onBootstrap(EventInterface $e)
{
    $application     = $e->getTarget();
    $serviceManager  = $application->getServiceManager();
    $eventManager    = $application->getEventManager();

    $eventManager->attach(new ZfrRest\Mvc\HttpExceptionListener());
}
{% endhighlight %}

Tuttavia, pensa a quello strano tipo che vuole inviare una email a sua nonna ogni volta che viene sollevata un'eccezione. Come possiamo fare? Beh, lui potrebbe modificare il codice del tuo modulo, ma questo non è proprio il modo migliore perchè ogni volta che aggiorna il modulo deve rifare le modifiche nuovamente.

Fortunatamente, usare il Service Manager risolve il problema:
{% highlight php %}
public function onBootstrap(EventInterface $e)
{
    $application     = $e->getTarget();
    $serviceManager  = $application->getServiceManager();
    $eventManager    = $application->getEventManager();

    $eventManager->attach($serviceManager->get('ZfrRest\Mvc\HttpExceptionListener'));
}
{% endhighlight %}

Adesso, il listener viene recuperato dal service manager. Certamente, ZfrRest fornisce un'implementazione di default di questo listener aggiunta nella configurazione del service manager:

{% highlight php %}
// in the module.config.php
return array(
    'service_manager' => array(
        'invokables' => array(
           'ZfrRest\Mvc\HttpExceptionListener' => 'ZfrRest\Mvc\HttpExceptionListener'
        )
    )
);
{% endhighlight %}

Adesso se qualcuno vuole fare l'ovveride del listener, deve solo modificare la chiave invokables nel suo file di configurazione:

{% highlight php %}
// in the module.config.php
return array(
    'service_manager' => array(
        'invokables' => array(
           'ZfrRest\Mvc\HttpExceptionListener' => 'MyModule\Mvc\SendBananaListener'
        )
    )
);
{% endhighlight %}

# Unit testing
Scrivi i test per il tuo modulo (si, è noioso). Ho già usato moduli non testati ed è davvero un casino. Ogni volta che aggiorno il modulo, tutta l'applicazione fallisce a causa dell'aggiornamento al modulo che ha rotto qualcosa. Scrivere delle test suite è soprattutto importante per le persone che vogliono usare il tuo modulo in maniera professionale.

Ancora una volta, scrivi i test per il tuo modulo. Punto.

# Pubblica il tuo modulo su Packagist (per l'utilizzo con Composer) e su zf2.modules
Molte persone utilizzano Composer come gestore delle dipendenze tra i moduli. Mentre tu potresti odiare composer, ricordati che molte persone lo adorano. Quindi pubblica il tuo modulo su Packagist se pensi sia utile.

Inoltre, puoi aggiungere il tuo modulo sulla pagina dedicata ai [moduli per ZF2](http://modules.zendframework.com/)

# Imposta delle dipendenze solo su quello che utilizzi nel modulo
Zend Framework 2 è un framework modulare. Questo significa che le persono non hanno bisogno di scaricare tutto il framework per usarlo. Per esempio, alcuni potrebbero essere interessati solo ai moduli filter e validator (usando i componenti di Symfony 2 Mvc). Tutto grazie a Composer.

Certamente le persone si infastidiscono se usare il tuo modulo significa scaricare tutto il framework ZF2. Ecco perchè invece di impostare l'intero framework come dipendenza, inserisci solo i moduli necessari. Ad esempio, diciamo che il tuo modulo usa il componente InputFilter di ZF2, niente altro. Modifica il file composer.json del tuo modulo :

{% highlight json %}

"require": {
    "php": ">=5.3.3",
    "zendframework/zendframework": "2.*"
}

{% endhighlight %}

con :
{% highlight json %}
"require": {
    "php": ">=5.3.3",
    "zendframework/zend-inputfilter": "2.*"
}
{% endhighlight %}

Adesso, verrà scaricato SOLO Zend Input Filter (per la precisione verranno scaricate anche le dipendenze del modulo InputFilter, ma non è un problema tuo ed è tutto gestito da composer, perchè ogni componente di ZF2 ha il suo file composer.json !).

>Per sistemare le dipendenze automaticamente è possibile usare questa libreria utilissima [zf2-components-list-generator](https://github.com/robertboloc/zf2-components-list-generator)