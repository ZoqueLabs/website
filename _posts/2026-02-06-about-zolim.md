---
layout: post
title: "About ZOLIM"
date: 2026-02-05 00:00:45 -0500 
categories: zolim
tags: inteligencia de amenazas, spyware, c2
author: ZoqueLabs
description: ¿Què es ZOLIM? (Zoque-Observatorio Latinoamericano de Infraestructura Maliciosa)
lang: en
mermaid: true
---

# ZOLIM - Zoque Latin American Observatory of Malicious Infrastructure

<br>
[[Go to ZOLIM](/zolim)]
<br>

**ZOLIM (Zoque Observatory of Malicious Infrastructure)** is a research initiative by ZoqueLabs that documents, analyzes, and publishes **periodic snapshots of malicious network infrastructure in Latin America**, with a particular focus on command-and-control (C2) systems and related threat frameworks.

ZOLIM is designed as an **observational and longitudinal system**. It does not monitor networks in real time, nor does it perform active scanning. Instead, it relies on structured snapshots that allow infrastructure to be compared across time.

## How ZOLIM works

ZOLIM periodically:
1. Queries third-party internet intelligence platforms (currently **Censys** and **Shodan**),
2. Filters results by country using a curated regional scope,
3. Matches exposed services against **signature sets** of known malicious tooling,
4. Normalizes and merges results into a single **timestamped snapshot**,
5. Generates technical outputs (reports and machine-readable datasets),
6. Publishes every snapshot **publicly** for transparency and reuse.

All processing is reproducible, and each snapshot is treated as a self-contained artifact.

## What ZOLIM is useful for

ZOLIM is intended to support:
- longitudinal analysis of malicious infrastructure in the region,
- identification of infrastructure reuse, churn, and drift over time,
- research into ASN, ISP, and country-level patterns,
- civil-society reporting, technical publications, and capacity building,
- downstream ingestion into threat-intelligence platforms.

ZOLIM focuses on **infrastructure behavior**, not attribution.

## Outputs and transparency

Every snapshot produced by ZOLIM is published in a **public repository**:

👉 **Public snapshots repository:**  
[https://github.com/ZoqueLabs/olim_datasets](https://github.com/ZoqueLabs/olim_datasets)

Each snapshot typically includes:
- a human-readable technical report,
- a normalized dataset (CSV),
- STIX 2.1 and MISP exports for interoperability.

## Software used

ZOLIM is powered by an open pipeline called **`zoque-infra-mapper`**, which handles:
- data collection from external sources,
- signature matching,
- dataset merging,
- report and export generation.

👉 **Software repository:**  
[https://github.com/ZoqueLabs/zoque-infra-mapper](https://github.com/ZoqueLabs/zoque-infra-mapper)

For implementation details, configuration, and limitations, see the README in the repository.

## Things to note and be careful about

- ZOLIM is **observational**, not authoritative.
- Matches are **signature-based**, not attribution.
- Results may include false positives or stale data inherited from upstream sources.
- Infrastructure presence does not imply control, intent, or operator identity.

All outputs should be interpreted with caution and contextualized appropriately.

## System overview

```mermaid
---
config:
  theme: dark
---
flowchart TB
    A[Censys] --> B[zoque-infra-mapper]
    C[Shodan] --> B
    D[Country Scope] --> B
    E[Threat Signatures] --> B

    B --> F[Merged Snapshot]
    F --> G[Technical Report]
    F --> H[CSV Dataset]
    F --> I[STIX 2.1 Export]
    F --> J[MISP Event]

    G --> K[Public Snapshots Repository]
    H --> K
    I --> K
    J --> K
````
<br>
[[Ir a ZOLIM](/zolim)]
