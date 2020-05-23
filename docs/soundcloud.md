---
id: soundcloud
title: Build a SoundCloud Clone in baahu
sidebar_label: SoundCloud Clone
---

## Why?

This tutorial will give you a high-level understanding of how to design applications in baahu. SoundCloud in particular is a good example because it has lots of state, often shared between components and tabs.

#### NOTE:

This tutorial is under construction. Read the [implementation](https://github.com/tjkandala/baahuPlayer) and see the [live website](https://sleepy-varahamihira-e0fc1a.netlify.app/) to see the current state of the project. The list of functional requirements/scope will grow over time.

## Functional Requirements

- Audio "miniplayer" fixed to the bottom of the screen, persists state between page transitions and reloads

- Fixed navbar at the top of the screen with search input

- The playing state should be synchronized between the miniplayer and the song + search pages

- Authorized and Unauthorized users use the same app, but only Authorized users can comment on songs

## Design + Finding a Machine Hierarchy

For the purpose of simplicity, we will implement three pages: Search ("/"), Song("/:songid"), and Login ("/login").

It can help to visualize the application you want to build before deciding what your Machine Components will be.

### Search Page

![search page](/img/search_page.svg)

Let's ignore the nav + player bars, which appear on every page, for now.

We see a play button on each search result, so we know that search results need to maintain internal state. Let's design the **"search result"** machine.

Each search result represents a song. The component will render the songs artwork, title, and playing state.

Users should be able to:

1. play and pause songs that are already the active song (the song played by the player bar)

2. change the selected song by pressing play on any song _not_ currently selected

From these requirements, we can model the search result as a machine with five states:

1. playing: this song is the active song, and it is currently playing.

2. paused: this song is the active song, and it is currently paused.

3. inactive: this song is not the active song.

4. loading: this song is loading, and will start playing when it is ready

5. error: this song failed to load

SoundCloud (web) itself doesn't have a visual representation of loading states, which can be jarring for users on slow connections. Modeling state is easier to handle with explicit state machines.

### Song Page

![song page](/img/song_page.svg)

There are two distinct features on this page

1. play and pause active song / loading inactive song

2. comment input machine

We will create two machines for this page: "big player" machine and "comment input" machine.

For the sake of the tutorial, the "big player" machine will be almost identical to the "search result" machine. In a true SoundCloud clone, you would implement a waveform timeline; its associated challenges are out of scope for a Baahu tutorial.

Let's model the comment input machine:

The primary

**States**

1.

**Context**

The comment machine contains state that cannot be modeled by a finite state machine; the timestamp. "Extended state," known as context in baahu,

### Login Page

![login page](/img/login_page.svg)

### Nav + Players bars

## Implementation

Github repo:
