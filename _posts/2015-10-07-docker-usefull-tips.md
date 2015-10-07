---
layout: post
title: Docker tips and tricks
description: "Some usefull docker tips and tricks"
category: articles
tags: [docker, tips, devops]
comments: true
---

Da qualche tempo ormai ho iniziato a giocare con docker, inizialmente solo per curiosità ma adesso direi che non posso farne a meno. Docker è ... beh ci sono un sacco di articoli che parlano di docker inutile dilungarmi.
Docker ha sostituito Vagrant nel mio ciclo di sviluppo del software, lo trovo molto più prestante e molto più semplice da configurare.

### Stoppare|Rimuovere|Killare tutti i container

Per stopparli

{% highlight bash %}
docker stop $(docker ps -q)
{% endhighlight %}

Per rimuoverli

{% highlight bash %}
docker rm $(docker ps -q)
{% endhighlight %}

### Rimuovere le immagini non taggate

{% highlight bash %}
docker rmi $(docker images -q -f dangling=true)
{% endhighlight %}


### Rimuovere tutte le immagini

{% highlight bash %}
docker rmi $(docker images -q)
{% endhighlight %}
