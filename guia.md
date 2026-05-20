# Informe Tècnic Avançat: Configuració de Proxy Web i Filtrat d'URL amb IPFire

## 1. Introducció i Context
Aquest document detalla de forma exhaustiva el procediment tècnic per a la posada en marxa i configuració del servei de Proxy Web (Squid) i el mòdul de Filtre de Contingut (URL Filter) en un entorn de tallafocs IPFire. 

L'objectiu principal és aplicar polítiques de restricció d'accés a la xarxa local (Green) per complir amb diferents requeriments: bloquejos temàtics (categories), llistes negres i blanques de dominis, control de rutes específiques, bloqueig per expressions regulars (paraules clau) i restriccions basades en horaris i subxarxes.

---

## 2. Fase Prèvia: Preparació de l'Entorn

### 2.1. Activació del servei Proxy al Servidor IPFire
Per defecte, l'IPFire no intercepta el trànsit web. S'ha d'activar el proxy per a la xarxa interna.
1. A la interfície web d'administració, cal dirigir-se al menú **Xarxa > Web Proxy**.
2. A l'apartat *Configuracions comunes*, es marca la casella **Activat en Green** per habilitar el servei a la xarxa local.
3. S'observa i es manté el **Port del proxy** establert per defecte al valor `800`.
4. A la part inferior d'aquesta mateixa secció, al requadre dedicat al **URL filter**, és obligatori marcar la casella **Activat**. Si no es fa aquest pas, el proxy funcionarà, però no s'aplicarà cap de les regles de filtratge posteriors.
5. Es desen els canvis. L'estat a la pàgina principal passarà de "Proxy Apagado" a estar actiu [1].

![Estat inicial del servidor abans de l'activació](pics/Captura%20de%20pantalla%202026-05-13%20211244.png)

### 2.2. Configuració del Proxy a l'Equip Client
S'ha optat per una implementació de **proxy no transparent**. Com indica la documentació tècnica, aquesta és la solució més senzilla, però requereix acció obligatòria sobre els clients, ja que el trànsit no es redirigeix automàticament [2].
1. A l'equip client (màquina Zorin/Ubuntu de proves), s'accedeix a les preferències de xarxa del sistema o del navegador.
2. Es canvia la configuració del proxy a mode **Manual**.
3. S'introdueix l'adreça IP de la interfície Green de l'IPFire: `192.169.11.254` i el port `800` tant per als protocols HTTP com HTTPS [1, 2]. Això obliga el navegador a enviar totes les peticions web a través del tallafocs.

---

## 3. Implementació de les Polítiques de Filtratge
Totes les polítiques següents es configuren exclusivament des del menú **Xarxa > Filtre de contingut**.

### Activitat 1: Instal·lació de les llistes negres (Blacklists)
Perquè el filtre de categories funcioni, el proxy necessita descarregar un diccionari que classifica milions de pàgines web segons la seva temàtica.
1. Es navega a la secció *Manteniment de URL Filter*.
2. Al bloc d'*Actualització automàtica de llista negra*, es marca la casella **Activar actualització automàtica**.
3. Es defineix la freqüència al menú desplegable com a **mensualment**.
4. Es selecciona l'origen de la base de dades: **Univ. Toulouse**.
5. Es clica a *Guardar configuracions d'actualització* i, molt important, es prem el botó **Actualitzar ara** per forçar la descàrrega immediata de la llista i poder aplicar els filtres en aquest precís instant.

### Activitat 2: Bloqueig per categories (Bancs i Ràdio)
Un cop descarregada la llista, es procedeix a bloquejar temàtiques completes.
1. A la secció *Configuracions de URL filter*, es localitza l'àrea de **Categorías bloqueadas**.
2. Es busquen i es marquen les caselles de les categories **bank** i **radio** [3].
3. Es desa i es reinicia el servei al final de la pàgina.
4. **Comprovació i Justificació Tècnica:** Al client, s'intenta accedir a `ing.es` [4]. El navegador mostra un error `ERR_TUNNEL_CONNECTION_FAILED` en lloc de la típica pàgina vermella d'accés denegat de l'IPFire. Com s'explica a la teoria teòrica del proxy, això succeeix perquè el trànsit **TLS (HTTPS) està xifrat**. El proxy utilitza el protocol SNI per llegir únicament el domini al qual es vol anar, detecta que és un banc i talla la connexió (el túnel) bruscament, sent impossible injectar codi HTML d'avís dins d'una connexió segura sense fer tècniques avançades de MitM (Man in the Middle) [5].

![Configuració de les categories bank i radio](pics/Captura%20de%20pantalla%202026-05-14%20193102.png)
![El tall del túnel TLS a ing.es](pics/Captura%20de%20pantalla%202026-05-14%20194659.png)

### Activitat 3: Bloqueig de dominis específics
Si es desitja denegar l'accés a pàgines concretes al marge de la seva categoria:
1. Es localitza l'apartat **Lista Negra personalizada**.
2. A la caixa de text esquerra, *Dominios bloqueados (uno por línea)*, s'afegeixen exactament:
   `elnacional.cat`
   `tecnocampus.cat`
3. Es marca la casella **Activar Lista Negra personalizada:** situada a sota de la caixa per habilitar la directiva [6].

![Configuració de la llista negra de dominis](pics/Captura%20de%20pantalla%202026-05-20%20181532.png)

### Activitat 4: Bloqueig d'una URL concreta respectant la resta del domini
El bloqueig de rutes o directoris específics dins d'un domini té una limitació tècnica important: **només funciona correctament de forma nativa en pàgines HTTP** (sense xifratge), ja que l'encriptació HTTPS oculta la ruta completa al proxy [5].
1. S'utilitza un domini HTTP de proves. A la mateixa secció de *Lista Negra personalizada*, però a la caixa dreta anomenada **URLs bloqueadas**, s'introdueix la ruta: `www.textfiles.com/jason/` [7].
2. Es comprova que la llista negra segueix activada.
3. **Comprovació:** S'intenta accedir primer a la portada del domini (`textfiles.com`), la qual carrega amb èxit [8]. A continuació, en intentar entrar a la ruta exacta bloquejada (`textfiles.com/jason/`), com el trànsit no està xifrat, el proxy pot llegir l'adreça completa i retorna la seva pàgina HTML oficial d'**ACCESS DENIED** [9].

![Bloqueig d'URL específica a textfiles.com](pics/Captura%20de%20pantalla%202026-05-20%20181137.png)
![Avís de bloqueig d'IPFire per HTTP](pics/Captura%20de%20pantalla%202026-05-20%20181525.png)

### Activitat 5: Bloqueig per paraula clau (anime) amb excepció per Llista Blanca
Aquesta directiva demostra l'ordre de prioritat de les normes: la Llista Blanca sempre preval sobre els bloquejos generals [10].
1. **La restricció:** A la secció *Lista de frases personalizadas* (que actua llegint expressions regulars a la URL), s'introdueix el terme `anime` i es marca la casella per activar aquesta llista [11, 12].
2. **L'excepció:** Es puja a la secció *Lista Blanca personalizada* i a *Dominios permitidos* s'afegeix `animenewsnetwork.com`. S'activa la casella corresponent [12].
3. **Comprovació pràctica:**
   * S'accedeix a `www4.animeflv.net`. El proxy llegeix la paraula "anime" al SNI del domini i rebutja la connexió tallant el túnel TLS [13].
   * S'accedeix a `animenewsnetwork.com`. Malgrat contenir la paraula prohibida, com que està registrat a la Llista Blanca, el domini carrega completament i de forma funcional, demostrant l'excepció [14].

![Configuració de la llista de frases i l'excepció a la llista blanca](pics/Captura%20de%20pantalla%202026-05-20%20183154.png)
![Tall del túnel en un domini bloquejat per expressió](pics/Captura%20de%20pantalla%202026-05-20%20182516.png)
![Càrrega satisfactòria del domini amb excepció](pics/Captura%20de%20pantalla%202026-05-20%20183200.png)

### Activitat 6: Restricció d'accés per temps i orígens
Es pot tallar l'accés complet a Internet per a una subxarxa sencera depenent del dia i l'hora.
1. A la part final de la configuració, a l'àrea *Añadir nueva regla de restricción de tiempo*, es marquen les caselles corresponents a tots els dies de la setmana (Lun-Dom) [15].
2. Es defineix la franja horària de prohibició des de les **00:00 fins a les 24:00**.
3. **Paràmetres clau:** 
   * **Host o xarxa d'origen:** S'especifica la subxarxa Green al complet afegint l'adreça amb la seva màscara: `192.169.11.0/24` [16].
   * **Destinació:** Es selecciona `cualquier` [16].
   * **Accés:** Es canvia el desplegable a `bloquear` [16].
4. Es clica a **Agregar** i posteriorment, a l'extrem inferior de la pantalla, es clica a **Guardar y Reiniciar**.
5. **Comprovació:** Al tractar de navegar des de la màquina client a una pàgina de tests HTTP com `httpforever.com`, l'IPFire mostra directament la pantalla d'**ACCESS DENIED**, demostrant que l'equip pertany a la xarxa d'origen castigada dins de la franja temporal especificada [17].

![Configuració pas a pas de la restricció temporal](pics/Captura%20de%20pantalla%202026-05-20%20184211.png)
![Regla temporal agregada al sistema](pics/Captura%20de%20pantalla%202026-05-20%20184239.png)
![Pantalla de bloqueig horari executada](pics/Captura%20de%20pantalla%202026-05-20%20184331.png)

***
*Fi de l'informe tècnic.*
