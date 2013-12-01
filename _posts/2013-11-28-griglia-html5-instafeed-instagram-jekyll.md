---
layout: instafeed
title: Creare una griglia html5 con immagini prese da instagram
description: "Creare una griglia html5 con immagini prese da instagram grazie a instafeed.js e integrarla in jekyll per creare un header dinamico molto carino"
category: articles
tags: [js, instagram, jekyll]
comments: true

---

Qualche tempo fa, ho letto un [articolo](http://www.marchettidesign.net/2013/07/realizzare-una-griglia-in-html5-con-foto-e-info-richiamate-da-instagram/) su come creare una griglia html5 con immagini prese da instagram e ho deciso di seguire quella guida ed integrarla in devplayground.

Il mio obiettivo è quello di modificare l'immagine statica del tema 'Minimal Mistake' con delle immagini recuperate direttamente da instagram.

### Script 

Per prima cosa scarichiamo lo scritp [Instafeedjs](http://instafeedjs.com), salviamolo in `path/al/mio/sitojekyll` nella directory `assets/js/vendor/instafeed.min.js`

### Modifichiamo il sito in modo da poter utilizzare instafeedjs

Modifichiamo il file `_includes/head.html` in maniera da caricare lo script, inserendo la seguente riga di codice alla fine del file

{% highlight html %}
<script type="text/javascript" src="{{site.url}}/assets/js/vendor/instafeed.min.js"></script>
{% endhighlight %}

### Creaiamo l'applicazione instagram

Per recuperare le foto da instagram dobbiamo creare una applicazione, il processo è molto veloce, basta andare sul sito di instagram all'indirizzo [http://instagram.com/developer/](http://instagram.com/developer/) accedere con il proprio account e seguire il wizard per la creazione. Quando avrete registrato l'applicazione vi verrà dato il `cliendId` da utilizzare con instafeedjs.


### Inizializziamo lo script instafeedjs

Modifichiamo il file `_includes/scripts.html` aggiungendo alla fine il codice per inizializzare instafeed:

{% highlight js %}
<script type="text/javascript">
    var feed = new Instafeed({
    	get: 'tagged',
		tagName: 'picoftheday',
        clientId: 'il vostro client id',
        resolution: 'thumbnail',
        limit: 25
    });
    feed.run();
</script>
{% endhighlight %}

### Creaiamo l'html necessario per visualizzare le foto

Per visualizzare le foto creaiamo il div in cui instafeed metterà le foto, per fare questo modifichiamo il file `_layouts/post.html` in questo modo:

{% highlight html %}
...
{% raw %}{% include navigation.html %}{% endraw %}
<div class="wrapper-header">
    <div id="instafeed"></div>
    <div class="shadow"></div>
</div>

<div id="main" role="main" .... >
....
{% endhighlight %}

Il plugin instafeedjs andrà ad inserire le foto nel div `<div id="instafeed"></div>`

### Creaiamo il css per dare un tocco di "stile"

{% highlight css %}
<style type="text/css">
.wrapper-header {
	background: #2a2a2a;
	overflow: hidden;
	height: 307px;
	position: relative;
	border-bottom: 1px solid white;
	margin-bottom: 25px;
}

.wrapper-header #instafeed {
	width: 120%;
	height: 300px;
	position: absolute;
	z-index: 1;
}

.wrapper-header .shadow {
	position: absolute;
	z-index: 2;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	background: url('data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgi…gd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0idXJsKCNncmFkKSIgLz48L3N2Zz4g');
	background: -webkit-gradient(linear, 50% 0%, 50% 100%, color-stop(50%, rgba(0,0,0,0.4)), color-stop(100%, rgba(0,0,0,0.8)));
	background: -webkit-linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
	background: -moz-linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
	background: -o-linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
	background: linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
}
</style>
{% endhighlight %}

Mettiamo lo stile subito sopra il div `<div class="wrapper-header">` in questo modo:

{% highlight html %}
<style type="text/css">
.wrapper-header {
	background: #2a2a2a;
	overflow: hidden;
	height: 307px;
	position: relative;
	border-bottom: 1px solid white;
	margin-bottom: 25px;
}

.wrapper-header #instafeed {
	width: 120%;
	height: 300px;
	position: absolute;
	z-index: 1;
}

.wrapper-header .shadow {
	position: absolute;
	z-index: 2;
	top: 0;
	bottom: 0;
	left: 0;
	right: 0;
	background: url('data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgi…gd2lkdGg9IjEwMCUiIGhlaWdodD0iMTAwJSIgZmlsbD0idXJsKCNncmFkKSIgLz48L3N2Zz4g');
	background: -webkit-gradient(linear, 50% 0%, 50% 100%, color-stop(50%, rgba(0,0,0,0.4)), color-stop(100%, rgba(0,0,0,0.8)));
	background: -webkit-linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
	background: -moz-linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
	background: -o-linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
	background: linear-gradient(rgba(0,0,0,0.4) 50%,rgba(0,0,0,0.8) 100%);
}
</style>
<div class="wrapper-header">
    <div id="instafeed"></div>
    <div class="shadow"></div>
</div>
{% endhighlight %}

Ed ecco il risultato

<figure>
	<img src="/images/devplayground-instagram.png">
	<figcaption>devplayground instafeedjs</figcaption>
</figure>