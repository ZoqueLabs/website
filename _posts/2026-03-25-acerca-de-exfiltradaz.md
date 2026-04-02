---
layout: post
title: Acerca de Exfiltradaz
date:   2026-03-25 12:00 -0500
categories: filtraciones. leaks
tags: filtraciones, leaks 
author: ZoqueLabs
description: Qué es Exfiltradaz - Monitoreo de filtraciones y exposición de datos LATAM
lang: es
---

# Exfiltradaz - Monitoreo de filtraciones y exposición de datos en LATAM

[Exfiltradaz](/exfiltradaz) es una iniciativa de ZoqueLabs para seguirle el rastro a la exposición y circulación de datos en América Latina.

Parte de una idea simple: muchas filtraciones no aparecen en noticias ni reportes formales, pero sí dejan huella en foros, marketplaces, canales y espacios donde estos datos se comparten, venden o discuten.

Exfiltradaz observa esos espacios y organiza esa información para hacer visible qué está circulando, dónde y con qué frecuencia.

---

## Qué hace

Exfiltradaz recolecta publicaciones asociadas a filtraciones de datos y las transforma en datos estructurados.

Esto permite:

- identificar actividad por país  
- observar sectores afectados  
- rastrear actores y fuentes recurrentes  
- construir snapshots periódicos del ecosistema  

---

## Cómo funciona

El sistema se basa en un pipeline que:

- recolecta mensajes desde fuentes abiertas  
- extrae información relevante desde contenido embebido  
- filtra registros asociados a países de LATAM  
- normaliza campos (país, sector, fuente, etc.)  
- genera snapshots en JSON  
- construye reportes en Markdown a partir de esos datos  

Los datos y reportes generados se publican de forma abierta:

- Dataset: https://github.com/ZoqueLabs/leaks-data  
- Pipeline: https://github.com/ZoqueLabs/leak-observatory  

Todo el código y los datos son abiertos. Exfiltradaz puede ser revisado, reutilizado o replicado como punto de partida para otras investigaciones.

---

## Qué es (y qué no es)

Exfiltradaz no es un repositorio de bases de datos filtradas ni un espacio de exposición directa de información sensible.

Tampoco es un sistema de verificación de incidentes.

Es una forma de observar y estructurar señales públicas sobre filtraciones, entendiendo que:

- no todo lo publicado es verificable  
- no todo lo relevante es visible  
- y gran parte del ecosistema opera en zonas grises  

---

## Limitaciones

- depende de fuentes públicas accesibles  
- existe sesgo hacia lo que es visible  
- la clasificación por sectores responde a criterios interpretativos y puede ser ambigua en algunos casos   
- la identificación de víctimas es parcial  

Los datos deben leerse considerando estas condiciones.

---

## Por qué existe

Porque en la región no hay muchas formas de ver este tipo de actividad de manera continua y estructurada.

Exfiltradaz busca aportar una capa básica de **visibilidad organizada**.

A partir de ahí, otros análisis pueden construirse.
