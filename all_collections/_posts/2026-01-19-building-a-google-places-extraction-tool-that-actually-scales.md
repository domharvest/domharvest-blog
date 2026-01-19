---
layout: post
title: "Building a Google Places Extraction Tool That Actually Scales"
date: 2026-01-19
categories: [data, scraping, google-places]
---

Extracting data from Google Places looks easy until you try to do it at scale.

A few hundred results work fine. A few thousand start to hurt. Country-wide extraction exposes all the cracks: pagination limits, duplicates, rate limits, unclear costs, long runs that fail halfway.

This post explains the **pattern** I ended up with while building a local Google Places extraction tool.
The example comes from a real UK project, but the approach is **country-agnostic**.

No scraping. No SaaS. Just controlled, repeatable extraction.

---

## The real problem is not access

Google Places data is accessible through an official API. That is not the hard part.

The hard part is everything around it:

- Pagination caps that force you to fan out queries
- The same business appearing in multiple nearby searches
- API rate limits and temporary errors
- Long extractions that fail after hours of progress
- No visibility into cost until the job is done

Most tools ignore these constraints or hide them behind subscriptions.

---

## The use case that forced structure

The original job was simple on paper:

- Extract barbers across the UK
- Include phone numbers
- Export to CSV
- Avoid duplicates
- Keep API costs predictable

In practice, this meant covering **200+ cities** across England, Scotland, Wales, and Northern Ireland, while keeping the run resumable and auditable.

This forced a more disciplined architecture.

---

## Core design principles

### 1. Country-agnostic search strategy

The UK case used a predefined city list.
The same approach works anywhere.

The key idea is not the country, but the **search grid**:

- Cities, regions, or custom locations
- One query per location
- Controlled pagination depth per area

Change the input list and the tool works globally.

---

### 2. Explicit pagination control

Google Places pagination is slow and capped.

The tool:

- Limits pages per location
- Inserts delays between page requests
- Stops early when results become redundant

This trades raw speed for predictability. At scale, predictability wins.

---

### 3. Deduplication as a first-class concern

Duplicates are guaranteed.

Deduplication happens on:

- Place ID
- Phone number

This removes overlaps across nearby cities and repeated queries. Deduplication is not a cleanup step. It is part of the extraction loop.

---

### 4. Filtering before export, not after

Filtering thousands of rows in Excel is a failure mode.

The tool filters during extraction:

- Include keywords
- Exclude keywords
- Optional phone requirement

Bad data is never written to disk.

---

### 5. Rate limiting and checkpoints

Two things matter in long runs:

- Not getting blocked
- Not losing progress

The tool includes:

- Fixed delays between requests
- Periodic checkpoints saved to disk

If the process stops, it resumes from the last checkpoint. No reruns. No wasted quota.

---

## Example configuration pattern

A simplified configuration looks like this conceptually:

- Search query and optional place type
- Target result count
- Include and exclude keywords
- Pagination depth
- Request delays
- Output format and checkpoint interval

Logic stays stable. Variability moves into configuration.

---

## Why this is not scraping

This approach uses the official Google Places API.

That brings tradeoffs:

- You respect rate limits
- You pay per request
- You accept the API’s data model

In return you get:

- Stability
- Legal clarity
- Predictable failures

For many business datasets, this is a better trade than scraping.

---

## What this pattern does not solve

Being explicit matters.

This does not:

- Bypass Google limits
- Extract hidden fields
- Guarantee completeness beyond API constraints

It optimizes **control**, not omniscience.

---

## From script to product, without changing the idea

The first version was a Node.js script built for a client job.

Later, the same pattern evolved into a local desktop app:

- UI instead of config files
- Live cost estimation
- Pause and resume controls
- Field-level export selection

Same principles. Better ergonomics.

The important part is not the app. It is the pattern.

---

## Takeaway

If you need Google Places data at scale, the winning move is not clever scraping.

It is:

- Breaking geography into controlled units
- Treating pagination and deduplication as core logic
- Making cost and failure visible early
- Running locally, with your own API key

Everything else is implementation detail.

I later turned this into a local tool: t.co/nQErTYs5tY
