*data: 2014-02-10*

Giocando con Selenium
=====================

![https://commons.wikimedia.org/wiki/File:Elektronskal_34.png](images/selenio.png)

E’ parecchio che conosco [Selenium](https://www.selenium.dev), credo dal 2009 quando l’ho visto usare in un workshop
di [Francesco Trucchia](https://www.francescotrucchia.it) al [PHPDay](http://2013.phpday.it) in quel di Verona.
Francesco ci mostrava come si poteva rifattorizzare un’applicazione dopo averne ingabbiato
il comportamento con test funzionali.

*Traduzione: prima di modificare il codice di un’applicazione che non conosci,
magari scritta male, generalmente da un’altro, è buona norma scrivere un programma
che simuli l’intervento dell’utente e svolga tutte le operazioni
più importanti sull’applicazione stessa.*

Con Selenium è possibile pilotare il browser e quindi muoversi tra le pagine di un sito,
compilare form, cliccare link, draggare elementi.
Quando facciamo questo sulla nostra applicazione stiamo eseguendo un test automatico
per verificare che si comporti come ci aspettiamo.
Quando invece lo facciamo su un altro sito per raggiungere delle informazioni
ed estrarle periodicamente, stiamo facendo web scraping.

La leggenda narra che con il Selenium IDE sia possibile registrare delle macro,
mentre un utente reale esegue le operazioni sull’applicazione,
per poi rieseguirle in un secondo momento e verificare che questa continui
a comportarsi nello stesso modo, nonostante il programmatore
abbia cambiato la sua struttura interna.

Ad oggi questo mito è stato ormai sfatato poiché è evidente che il modo più
pratico per utilizzare Selenium è quello di programmare manualmente le operazioni
da eseguire attraverso una delle varie librerie a disposizione per ciascuno
linguaggio di programmazione.

Per funzionare Selenium deve poter pilotare un browser, ad esempio Firefox,
ed un browser ha bisogno di un ambiente grafico per funzionare.
Su un server web però non abbiamo un ambiente grafico.
Sul proprio computer invece è piuttosto difficile ed irritante fare qualsiasi
altra cosa mentre Firefox continua ad aprirsi, *sbilinare* e chiudersi da solo.

Possiamo però simulare l’ambiente grafico e dire a Selenium di usarlo.
La parolina magica, suggeritami da [Giorgio Sironi](https://github.com/giorgiosironi)
all’[AgileDay2013](https://www.agileday.it/), è **xvfb**.

Installiamo [Xvfb](https://www.x.org/releases/current/doc/man/man1/Xvfb.1.xhtml)
e facciamo in modo che venga lanciato all’avvio della macchina:

```
sudo apt-get install xvfb
sudo vim /etc/init.d/xvfb
```

```
# /etc/init.d/xvfb

#!/bin/bash

if [ -z "$1" ]; then
echo "`basename $0` {start|stop}"
exit
fi

case "$1" in
start)
/usr/bin/Xvfb :99 -ac -screen 0 1024x768x8 > /dev/null &
;;

stop)
killall Xvfb
;;
esac
```

```
sudo chmod 755 /etc/init.d/xvfb
vim ~/.bash_aliases
```

```
# ~/.bash_aliases

...
alias xvfb='etc/init.d/xvfb'
```

```
sudo /etc/init.d/update-rc.d xvfb default 10
```

Ora è possibile avviare xvfb tramite il comando `xvfb start` e fermarlo tramite `xvfb stop`

Ora installiamo Selenium.

Scarichiamo da [https://www.selenium.dev/downloads](https://www.selenium.dev/downloads)
l’ultima versione di **Selenium Server (formerly the Selenium RC Server)**:

Lo mettiamo tra le nostre librerie e poi facciamo la stessa cosa che abbiamo fatto per xvfb,
ne semplifichiamo l’esecuzione.

```
wget http://selenium.googlecode.com/files/selenium-server-standalone-2.39.0.jar
sudo mkdir /var/lib/selenium
sudo mv selenium-server-standalone-2.39.0.jar /var/lib/selenium
cd /var/lib/selenium
sudo ln -s selenium-server-standalone-2.39.0.jar selenium-server.jar
sudo vim /etc/init.d/selenium
```

```
# /etc/init.d/selenium

#!/bin/bash

if [ -z "$1" ]; then
echo "`basename $0` {start|stop}"
exit
fi

export DISPLAY=":99"
case "$2" in
show)
export DISPLAY=":0"
;;
esac

case "$1" in
start)
java -jar /var/lib/selenium/selenium-server.jar -browserSessionReuse > /dev/null &
;;

stop)
curl http://localhost:4444/selenium-server/driver/?cmd=shutDownSeleniumServer
;;
esac
```

```
sudo 755 selenium
vim ~/.bash_aliases
```

```
# ~/.bash_aliases
...
alias selenium='etc/init.d/selenium'
```

Ora è possibile avviare selenium nell’ambiente grafico generato da xvfb con
il comando selenium start oppure in quello reale (se c’è) con `selenium start show`,
e fermarlo con `selenium stop`

In alcuni casi potrebbe essere interessante lanciarlo all’avvio del server:
lo si può fare con

```
sudo /etc/init.d/update-rc.d selenium default 20
```

Se non l’avete fatto non rimane che installare Firefox

```
sudo apt-get install firefox
```

Ora possiamo creare test funzionali con
[Selenium2 PHPUnit Extension](http://phpunit.de/manual/3.7/en/selenium.html) oppure
far web scraping con [Mink](http://mink.behat.org).

Questo articolo non è altro che un’assemblaggio ed una personalizzazione
delle pratiche lette nei seguenti:

- [http://www.installationpage.com/selenium/how-to-run-selenium-headless-firefox-in-ubuntu/](http://www.installationpage.com/selenium/how-to-run-selenium-headless-firefox-in-ubuntu/)

- [http://www.labelmedia.co.uk/blog/setting-up-selenium-server-on-a-headless-jenkins-ci-build-machine.html](http://www.labelmedia.co.uk/blog/setting-up-selenium-server-on-a-headless-jenkins-ci-build-machine.html)

- [http://testerinyou.blogspot.it/2011/05/run-selenium-rc-using-command-prompt.html](http://testerinyou.blogspot.it/2011/05/run-selenium-rc-using-command-prompt.html)

Direi che è tutto. Ciao!

