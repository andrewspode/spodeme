---
title: Virgil
description: A journalling companion built on Claude Code, designed for long-running conversations that accumulate context over months and years.
date: 2026-04-09
permalink: /software/virgil/
image: /software/virgil/virgil.png
tags: [AI, Journalling, Claude Code, Python, Open Source]
---

![Virgil](virgil.png)

Journalling is an oft-overlooked therapy tool - there's a lot to be said for getting your thoughts and feelings down on paper. The process alone is often enough to help you work through something. But sometimes that little bit of feedback is what's needed.

I started talking to Claude in the evenings to work through how my day had gone - what I describe as Journal++. After a few weeks it began tying things together in ways I found genuinely useful: noticing patterns, connecting threads, pushing back when I was being too hard on myself or not hard enough.

The problem was context. Claude's web chat gets progressively slower as a conversation grows, and it's just not built for something that runs for months. It didn't take long before it had ground to a halt, even after compacting itself twice - and losing information in the process.

So I built [Virgil](https://github.com/andrewspode/virgil). Named after the guide in Dante's Inferno - someone who knows you well enough to be honest with you.

Virgil runs as a [Claude Code](https://claude.ai/code) project in a git repository. It keeps full conversation logs and builds a hierarchy of summaries - daily, weekly, monthly, quarterly, yearly - so context accumulates rather than disappearing. When you start a new session, Virgil reads back through those summaries and picks up exactly where you left off, without having to process months of raw history - though it's all there when it needs more detail.

A hook injects the current date and time into every message, so it always knows when it is - morning, evening, how long since you last spoke - without you having to say so.

This recursive summarisation has real benefits beyond journalling, so feel free to fork it and bend it to your needs.

[![View on GitHub](https://img.shields.io/badge/View_on-GitHub-black?logo=github)](https://github.com/andrewspode/virgil)
