---
layout: post
title: About Exfiltradaz
date: 2026-03-25 12:00 -0500
categories: leaks
tags: leaks 
author: ZoqueLabs
description: What is Exfiltradaz - Monitoring leaks and exposure of LATAM data
lang: en
---

# Exfiltradaz - Monitoring leaks and data exposure in LATAM

[Exfiltradaz](/exfiltradaz) is an initiative by ZoqueLabs to track the exposure and circulation of data in Latin America.

It starts from a simple idea: many leaks do not appear in news or formal reports, but they do leave their mark in forums, marketplaces, channels and spaces where this data is shared, sold or discussed.

Exfiltradaz observes these spaces and organizes that information to make visible what is circulating, where and how frequently.

---

## What does it do

Exfiltradaz collects posts associated with data breaches and transforms them into structured data.

This allows:

- identify activity by country  
- observe affected sectors  
- track recurring actors and sources  
- build periodic snapshots of the ecosystem  

---

## How it works

The system is based on a pipeline that:

- collect messages from open sources  
- extract relevant information from embedded content  
- filters records associated with LATAM countries  
- normalizes fields (country, sector, source, etc.)  
- generate snapshots in JSON  
- build reports in Markdown from that data  

The data and reports generated are published openly:

- Dataset: https://github.com/ZoqueLabs/leaks-data  
- Pipelines: https://github.com/ZoqueLabs/leak-observatory  

All code and data is open. Exfiltradaz can be reviewed, reused or replicated as a starting point for further research.

---

## What it is (and what it is not)

Exfiltradaz is neither a repository of leaked databases nor a space for direct exposure of sensitive information.

It is also not an incident verification system.

It is a way of observing and structuring public signals about leaks, understanding that:

- not everything published is verifiable  
- not everything relevant is visible  
- and much of the ecosystem operates in gray areas  

---

## Limitations

- depends on accessible public sources  
- there is bias towards what is visible  
- the classification by sectors responds to interpretive criteria and can be ambiguous in some cases   
- the identification of victims is partial  

The data must be read considering these conditions.

---

## Why it exists

Because in the region there are not many ways to see this type of activity in a continuous and structured way.

Exfiltradaz seeks to provide a basic layer of organized **visibility**.

From there, other analyzes can be built.
