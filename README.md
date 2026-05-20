# Práctica: UD9.AA3 Servicios de proxy (IPFire)

**Módulo:** 0227 Servicios de red (RA8. Acceso a redes públicas)  
**Ciclo:** CFGM SMX  
**Servicio principal:** Proxy Web (Squid) y Filtro de URLs  

## 🎯 Objetivo de la práctica
El objetivo de esta actividad es aprender a administrar el control de acceso a internet en una red local utilizando el servicio de Proxy Web y el Filtro de Contenido (URL Filter) del cortafuegos IPFire. Se pondrán en práctica configuraciones de listas negras, listas blancas, expresiones regulares y control horario.

## 📋 Enunciado de las actividades
Configura el proxy de IPFire para que realice las siguientes acciones de filtrado:

1. **Instalar las listas negras:** Configura la descarga o actualización de las bases de datos de filtrado en el sistema.
2. **Bloqueo por categorías:** Bloquea las categorías relacionadas con bancos y radio. Comprueba que el bloqueo funciona intentando acceder a las páginas de prueba (ing.es y ah.fm).
3. **Bloqueo de dominios:** Bloquea explícitamente los dominios de elnacional y tecnocampus.
4. **Bloqueo de rutas específicas:** Prueba a bloquear una ruta concreta de un dominio, permitiendo que la portada del dominio siga siendo accesible.
5. **Bloqueo por palabras y excepciones:** Bloquea toda página web que contenga el término prohibido "anime", con la excepción de permitir un dominio específico mediante la lista blanca.
6. **Restricción temporal:** Prueba el funcionamiento del bloqueo de navegación por horas definiendo una franja horaria restringida.

## ⚙️ Requisitos previos
* Disponer de un servidor IPFire con las interfaces de red interna (LAN) y externa (Internet) operativas.
* Un equipo cliente conectado a la red local.
* Configurar el proxy de forma manual en el navegador del equipo cliente para enrutar el tráfico por el puerto correspondiente.

---
[Guia](guia.md)
