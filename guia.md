# Informe Tècnic Avançat: Configuració de Proxy Web i Filtrat d'URL amb IPFire

## 1. Introducció i Context
Aquest document detalla de forma exhaustiva el procediment tècnic per a la posada en marxa i configuració del servei de Proxy Web (Squid) i el mòdul de Filtre de Contingut (URL Filter) en un entorn de tallafocs IPFire. 

L'objectiu principal és aplicar polítiques de restricció d'accés a la xarxa local (Green) per complir amb diferents requeriments: bloquejos temàtics (categories), llistes negres i blanques de dominis, control de rutes específiques, bloqueig per expressions regulars (paraules clau) i restriccions basades en horaris i subxarxes.

---

## 2. Fase Prèvia: Preparació de l'Entorn

### 2.1. Activació del servei Proxy al Servidor IPFire
Per defecte, l'IPFire no intercepta el trànsit web. S'ha d'activar el proxy per a la xarxa interna abans d'aplicar qualsevol filtre.
1. A la interfície web d'administració, cal dirigir-se al menú **Red > Web Proxy**.
2. A l'apartat *Configuraciones comunes*, es marca la casella **Activado en Green** per habilitar el servei a la xarxa local.
3. S'observa i es manté el **Puerto del proxy** establert per defecte al valor `800`.
4. A la part inferior d'aquesta mateixa secció, al requadre dedicat al **URL filter**, és obligatori marcar la casella **Activado**. Si no es fa aquest pas, el proxy funcionarà, però no s'aplicarà cap de les regles de filtratge posteriors.
5. Es desen els canvis. L'estat a la pàgina principal passarà de "Proxy Apagado" a estar actiu per defecte.

![Estat inicial del servidor abans de l'activació](pics/Captura%20de%20pantalla%202026-05-13%20211244.png)

### 2.2. Configuració del Proxy a l'Equip Client
S'ha optat per una implementació de **proxy no transparent**. Aquesta solució requereix acció obligatòria sobre els clients, ja que el trànsit no es redirigeix automàticament.
1. A l'equip client s'accedeix a les preferències de xarxa del sistema o del navegador.
2. Es canvia la configuració del proxy a mode **Manual**.
3. S'introdueix l'adreça IP de la interfície Green de l'IPFire: `192.169.11.254` i el port `800` tant per als protocols HTTP com HTTPS. Això obliga el navegador a enviar totes les peticions web a través del tallafocs.

---

## 3. Implementació de les Polítiques de Filtratge
Totes les polítiques següents es configuren exclusivament des del menú **Red > Filtro de contenido**.

### Activitat 1: Instal·lació de les llistes negres (Blacklists)
Perquè el filtre de categories funcioni, el proxy necessita descarregar un diccionari que classifica pàgines web segons la seva temàtica.
1. Es navega a la secció *Mantenimiento de URL Filter*.
2. Al bloc d'*Actualización automática de lista negra*, es marca la casella **Activar actualización automática**.
3. Es defineix la freqüència al menú desplegable com a **mensualmente**.
4. Es selecciona l'origen de la base de dades: **Univ. Toulouse**.
5. Es clica a *Guardar configuraciones de actualización* i es prem el botó **Actualizar ahora** per forçar la descàrrega.

### Activitat 2: Bloqueig per categories (Bancs i Ràdio)
Un cop descarregada la llista, es procedeix a bloquejar temàtiques completes.
1. A la secció *Configuraciones de URL filter*, es localitza l'àrea de **Categorías bloqueadas**.
2. Es busquen i es marquen les caselles de les categories **bank** i **radio**.
3. Es desa i es reinicia el servei al final de la pàgina.

![Configuració de les categories bank i radio](pics/Captura%20de%20pantalla%202026-05-14%20193102.png)

*   **Comprovació i Justificació Tècnica:** Al client, s'intenta accedir a `ing.es`. El navegador mostra un error `ERR_TUNNEL_CONNECTION_FAILED` en lloc de la típica pàgina vermella d'accés denegat de l'IPFire. Això succeeix perquè el trànsit **TLS (HTTPS) està xifrat**. El proxy utilitza el protocol SNI per llegir únicament el domini, detecta que és un banc i talla la connexió (el túnel) bruscament, sent impossible injectar codi HTML d'avís dins d'una connexió segura.

![El tall del túnel TLS a ing.es](pics/Captura%20de%20pantalla%202026-05-14%20194659.png)

### Activitat 3: Bloqueig de dominis específics
Per denegar l'accés a pàgines concretes al marge de la seva categoria:
1. Es localitza l'apartat **Lista Negra personalizada**.
2. A la caixa de text esquerra, *Dominios bloqueados (uno por línea)*, s'afegeixen:
   `elnacional.cat`
   `tecnocampus.cat`
3. Es marca la casella **Activar Lista Negra personalizada:** situada a sota de la caixa per habilitar la directiva.

![Configuració de la llista negra de dominis](pics/Captura%20de%20pantalla%202026-05-20%20181532.png)

### Activitat 4: Bloqueig d'una URL concreta respectant la resta del domini
El bloqueig de rutes o directoris específics dins d'un domini requereix pàgines HTTP, ja que l'encriptació HTTPS oculta la ruta completa al proxy.
1. S'utilitza un domini HTTP. A la mateixa secció de *Lista Negra personalizada*, a la caixa dreta anomenada **URLs bloqueadas**, s'introdueix la ruta exacta: `www.textfiles.com/jason/`.
2. Es desa la configuració.
3. **Comprovació:** S'intenta accedir primer a la portada del domini (`textfiles.com`), la qual carrega amb èxit. A continuació, en intentar entrar a la ruta bloquejada (`textfiles.com/jason/`), com el trànsit és en text pla HTTP, el proxy llegeix la ruta exacta i intercepta la connexió mostrant la pàgina **ACCESS DENIED**.

![Avís de bloqueig d'IPFire per HTTP a la ruta específica](pics/Captura%20de%20pantalla%202026-05-20%20181525.png)

### Activitat 5: Bloqueig per paraula clau (anime) amb excepció per Llista Blanca
Aquesta directiva demostra l'ordre de prioritat de les normes: la Llista Blanca sempre preval sobre els bloquejos generals.
1. **La restricció:** A la secció *Lista de frases personalizadas* (expressions regulars a la URL), s'introdueix el terme `anime` i es marca la casella per activar-ho.
2. **L'excepció:** A la secció *Lista Blanca personalizada* i a *Dominios permitidos* s'afegeix `animenewsnetwork.com`. S'activa la casella.

![Configuració de la llista de frases i l'excepció a la llista blanca](pics/Captura%20de%20pantalla%202026-05-20%20183154.png)

3. **Comprovació pràctica:**
   * S'accedeix a `www4.animeflv.net`. El proxy detecta la paraula prohibida al SNI del domini i rebutja la connexió tallant el túnel TLS.

![Tall del túnel en un domini bloquejat per expressió](pics/Captura%20de%20pantalla%202026-05-20%20182516.png)

   * S'accedeix a `animenewsnetwork.com`. Malgrat contenir la paraula prohibida, com que està registrat a la Llista Blanca, el domini carrega completament i de forma funcional.

![Càrrega satisfactòria del domini amb excepció](pics/Captura%20de%20pantalla%202026-05-20%20183200.png)

### Activitat 6: Restricció d'accés per temps i orígens
Es talla l'accés complet a Internet per a una subxarxa en un horari definit.
1. A l'àrea *Añadir nueva regla de restricción de tiempo*, es marquen tots els dies de la setmana (Lun-Dom).
2. Es defineix la franja horària des de les **00:00 fins a les 24:00**.
3. **Paràmetres clau:** 
   * **Host o red(es) de origen:** S'especifica la subxarxa Green al complet: `192.169.11.0/24`.
   * **Destino:** Es selecciona `cualquier`.
   * **Acceso:** Es canvia el desplegable a `bloquear` i es clica **Agregar**.

![Configuració pas a pas de la restricció temporal](pics/Captura%20de%20pantalla%202026-05-20%20184211.png)

![Regla temporal agregada correctament al sistema](pics/Captura%20de%20pantalla%202026-05-20%20184239.png)

4. **Comprovació:** Al tractar de navegar des de la màquina client a una pàgina de tests HTTP com `httpforever.com`, l'IPFire mostra directament la pantalla d'**ACCESS DENIED**, demostrant que l'equip pertany a la xarxa castigada dins de la franja temporal.

![Pantalla de bloqueig horari executada amb èxit](pics/Captura%20de%20pantalla%202026-05-20%20184331.png)

***
[Tornar enrere](README.md)

