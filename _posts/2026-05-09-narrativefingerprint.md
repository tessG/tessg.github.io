---
layout: post
title: "Narrative Fingerprint"
author: "Tess Gaston"
categories: AI assisted data analysis
tags: [data viz, graph database, AI]
image: narrativefingerprint/narrativefingerprint.png
---

Narrative Fingerprint Explorer — a proof of concept [try it out](http://159.223.26.34:3000/)
I all started when I came across Ioannis Siglidis research  [Ioannis Siglidis's Hidden Objects project](https://hidden-objects.github.io/), I found myself asking a question it wasn't asking.
The project is about placement of objects in scenes — ie. teaching a model where a pizza belongs in a room.
What if the scene was a narrative theory framework, the pizza a story, and the placement was the narrative framing of it?
The way we as humans frame the same event differently resulting in wildly different stories that are all true at the same time, makes narrative framing hard to formalize.

This is a proof of concept for using a knowledge graph to make narrative structure visible and queryable. 
An explicit ontology is made available for a language model (Claude) to support it's annotation of soMe posts about a given event 
The graph is then enriched with the annotations, and queried from a web client. 
The theoretical knowledge lives in a knowledge graph, not in model weights, so it remains inspectable, versionable, and contestable.

_This was built with Claude's help from ideation to implementation. I am not well versed in narrative theory — Claude suggested a list of
concepts from the theoretical frameworks of Entman and Greimas , and I am not able to 
confidently validate them. I used Claude code to help build the application in approximately 12 hours_

⏺ Infrastructure
  - GraphDB 10 on DigitalOcean (Docker, Ubuntu 24, 2GB RAM + swap) - SPARQL endpoint at http://159.223.26.34:7200/repository
  - Next.js 16 App Router, TypeScript, Tailwind, running locally via npm run dev

  Ontology (http://159.223.26.34:7200/resource?uri=http:%2F%2Fwww.openrdf.org%2Fschema%2Fsesame%23nil&role=context)
  - 4 abstract classes: NarrativeRole, FramingFunction, AffectiveRegister, CausalStructure
  - 18 theory-level tags (Hero, Villain, Victim, Outrage, SystemicCause, etc.)
  - nar:Post — content, date, platform, language, region, event
  - nar:Annotation — links post → tag (one annotation per tag per post)
  - nar:Event — groups posts into named events (BLM Denmark 2020, Gaza 2024, Ukraine 2022)

  Pipeline - work in progress (pipeline/)
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