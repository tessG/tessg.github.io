---
layout: post
title: "Narrative Fingerprint"
author: "Tess Gaston"
categories: AI assisted data analysis
tags: [data viz, AI, vibe codinh]
image: narrativefingerprint/narrativefingerprint.png
---

Narrative Fingerprint Explorer — a proof of concept
Stumbling across Ioannis Siglidis's Hidden Objects project, I found myself asking a question it wasn't asking. 
The project places objects in scenes — teaching a model where a pizza belongs in a room.
What if the scene was an event, and the placement was narrative framing? 
The cultural frames through which we perceive events are no less structured than a living room. 
They are just harder to see, because we are inside them.

Siglidis's own framing of his research made this feel worth pursuing (paraphrasing): _that by operationalising perceptual 
and ontological questions, machine learning turns modelling into ontology-in-action, and that the humanities 
— as statistical sciences of perception — are ideal generators of machine learning problems_.

This is a proof of concept for making narrative structure visible and queryable. 
It uses an explicit ontology as the lens through which a language model annotates posts about a given event 
— producing what I am calling a narrative fingerprint. 
The theoretical knowledge lives in a knowledge graph, not in model weights, so it remains inspectable, versionable, and contestable.
I am not well versed in narrative theory — Claude suggested the frameworks by Entman and Greimas, and I am not able to 
validate them independently. 

_This was built with Claude's help from idea to implementation, driven partly by my curiosity about knowledge representation and graph technology, 
and partly by an itch to vibe with Claude Code_.


⏺ Infrastructure
  - GraphDB 10 on DigitalOcean (Docker, Ubuntu 24, 2GB RAM + swap) — SPARQL endpoint at GRAPHDB_URL/repositories/GRAPHDB_REPO
  - Next.js 16 App Router, TypeScript, Tailwind, running locally via npm run dev

  Ontology (nar: prefix http://narratives.poc/ontology#)
  - 4 abstract classes: NarrativeRole, FramingFunction, AffectiveRegister, CausalStructure
  - 18 theory-level tags (Hero, Villain, Victim, Outrage, SystemicCause, etc.)
  - nar:Post — content, date, platform, language, region, event
  - nar:Annotation — links post → tag (one annotation per tag per post)
  - nar:Event — groups posts into named events (BLM Denmark 2020, Gaza 2024, Ukraine 2022)

  Pipeline (pipeline/)
  - scrape.ts — Reddit OAuth2, fetches top posts for an event's subreddit → pipeline/data/posts.json
  - annotate.ts — Claude Haiku classifies each post against the 18-tag ontology → pipeline/data/annotated.json (resumable)
  - ingest.ts — batch SPARQL UPDATE into GraphDB, writes event node + post + annotation triples

  Shared config (lib/)
  - events.ts — event registry (URI, label, subreddit, language, region) imported by both pipeline and API routes
  - sparql.ts — fetch wrapper for GraphDB SPARQL queries

  API routes (app/api/) — all accept ?event=<uri> query param
  - tag-frequency — tag label × annotation count
  - cooccurrence — tag pair × post count
  - temporal-arc — date × causal tag × count (SystemicCause vs IndividualCause over time)
  - tension-posts — posts carrying contradictory tag combinations (Hero+Disgust, Villain+Solidarity, etc.)
  - narrative-graph — theory nodes weighted by annotation count + co-occurrence edges

  Dashboard (app/)
  - Event selector — drives all data fetches; reads from lib/events.ts
  - NarrativeGraphView — p5.js force-directed graph, abstract nodes at center, theory nodes in periphery weighted by annotation count
  - TagFrequencyView — bar chart of tag usage
  - FrameShiftView — temporal arc of causal framing
  - CooccurrenceView — tag co-occurrence
  - TensionPostsView — contradictory posts feed