---
title: "Toward Building a Lab-Internal Agent Skill"
date: 2026-02-16
tags: ["LLM agents", "skills", "research tooling"]
excerpt: "How I turned common lab workflows into a Claude skill so future students don't have to learn the hard way."
draft: false
---

As a senior-year PhD student, I've started thinking about what I can leave behind for the lab when I graduate.

I still miss the lab friends who graduated before me. They held my hand during my junior years, walking me through the common pitfalls and mistakes they had experienced firsthand. But once they graduated, their lessons became scattered scripts on GitHub — useful, but missing the context of sitting together and working through problems side by side.

When I started working with coding agents, it felt like that smart colleague had come back. At first, they weren't quite capable enough to generate the publication-ready figures I needed. But that changed once [skills](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) came out.

## What Are Skills?

Skills provide a standardized workflow for agents. Instead of prompting from scratch every time, you can encode domain expertise — the kind of knowledge that normally takes months of mentorship to transfer — into a structured format that an agent can follow reliably.

## Building a Skill for the Lab

I wrapped up some of our lab's most common tools and workflows into a skill called [statgen_skills](https://github.com/CamelliaRui/statgen_skills) for internal use. Each skill includes a description of the workflow and the reasoning behind each step, following the [best practices for authoring skills](https://resources.anthropic.com/hubfs/The-Complete-Guide-to-Building-Skill-for-Claude.pdf?hsLang=en).

The goal is straightforward: help incoming students onboard to our field's popular methods and common workflows without needing someone to sit next to them every step of the way. The skill captures not just *what* to do, but *why* — the kind of tacit knowledge that usually lives in a senior student's head and disappears when they defend their thesis.

## What's Next

This is still early. The skill covers the basics, but there's a lot more institutional knowledge worth encoding — from debugging common pipeline failures to interpreting edge cases in simulation results. I plan to keep expanding it as I wrap up my own projects.

If you're in a similar position — a senior student thinking about knowledge transfer — I'd encourage you to try building a skill for your lab. The format is surprisingly natural for capturing the "here's what I wish someone had told me" kind of advice.
