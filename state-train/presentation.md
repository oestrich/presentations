# All Aboard the Stateful Train

[.header: #FDDF26, line-height(.8), text-scale(2), Circular Std]
[.text: #FFFFFF, Circular Std]

![original](title-slide.png)

Eric Oestrich
SmartLogic

---
[.header: #37A792, alignment(left), Circular Std]

![original](plain-slide.png)
![left](eric.png)


# Who am I?

Eric Oestrich
Wizard at SmartLogic

---
[.header: #37A792, alignment(left), Circular Std]

![original](plain-slide.png)

![left](eric.png)

# Who is SmartLogic?

We build custom web and mobile apps.

Based in Baltimore, we've built over 150 apps since 2005.

---

![original](plain-slide.png)

# Agenda

- What does a standard application look like?
- Why would you go stateful?
- What does a stateful application look like?
- Is stateful better?
- Let's go stateful!

---

![original](section-div.png)

# Standard Application

---

![original](plain-slide.png)

# Standard Application

- Stateless HTTP
- Controllers go straight to the database
- Does not leverage OTP
- MVC, Rails-like

^ TODO How does data flow through this?

---

![original](plain-slide.png)

# Standard Application Examples

- Wordpress
- Hubspot
- Wikipedia
- Shopify

---

![original](plain-slide.png)

# Stateful Application

- Stateless HTTP
- Websockets/TCP connections
- Heavy use of OTP
- "Background" processes

^ TODO How does data flow through this? see later in the examples, just show a quick snippet up here first

---

![original](plain-slide.png)

# OTP and Processes

- Used to be Open Telecom Platform
- OTP is the standard library of Erlang
- GenServers, supervision trees, :ets, all the fun stuff
- Processes are the base building block of Erlang concurrency

---

![original](plain-slide.png)

# Supervision Tree

![inline](supervision-tree.png)

---

![original](section-div.png)

# Why Stateful?

---

![original](plain-slide.png)

# Why Stateful?

- We're on a platform that can handle it
- Local caches
- Processes working on state

---

![original](plain-slide.png)

## Did I mention Erlang is great at this?

![inline](servers.jpg)

[.footer: https://unsplash.com/photos/M5tzZtFCOfs]

---

![original](section-div.png)

# Let's review some stateful applications

^ Max and mid stateful

---

[.text: alignment(center)]

![original](plain-slide.png)

![inline](exventure.png)

^ Stateless |-------------------*| Stateful

^ TODO graphic of stateless to stateful, exventure on the edge

---

[.text: alignment(center)]

![original](plain-slide.png)

# ExVenture

A MUD Server
[exventure.org](https://exventure.org)

![inline](exventure-state.jpg)

---

![original](plain-slide.png)

# What is a MUD?

- Multi-User [Dungeon](https://en.wikipedia.org/wiki/Dungeon_\(video_game\))
- Online text adventure
- Connects with telnet

^ 1976 for Dungeon

---

![original](plain-slide.png)

# What is a text adventure?

```
This is an open field west of a white house, with a boarded front door.
There is a small mailbox here.
A rubber mat saying 'Welcome to Zork!' lies by the door.

>
```

---

![original](plain-slide.png)

# MidMUD

The flagship ExVenture instance

[midmud.com](https://midmud.com)

![inline](midmud.png)

---

![original](plain-slide.png)

# Review of ExVenture

- A "living" world
- Everything is always on
- The database is there for startup
- Lots of inter-process communication

---

![original](plain-slide.png)

# Demo

^ TODO diagram of how actors work together - we'll see this later

---

![original](plain-slide.png)

# Could this be stateless?

---

![original](plain-slide.png)

# Not Really

- Poll based?
- Background worker that updated the world every X seconds?
- Gross

---

![original](plain-slide.png)

[.text: alignment(center)]

![inline](grapevine.png)

^ Stateless |---------*----------| Stateful

^ TODO graphic of stateless to stateful, grapevine in the middle

---

![original](plain-slide.png)

# Grapevine

[.text: alignment(center)]

MUD community site
[grapevine.haus](https://grapevine.haus)

![inline](grapevine-state.jpg)

---

![original](plain-slide.png)

# Grapevine

- Steam for MUDs
- Cross game chat
- Web client

---

![original](plain-slide.png)

# Review of Grapevine

- Close to a standard application
- State involved is websocket and telnet
- Caches

---

![original](plain-slide.png)

# Demo

- Game listings, stats
- Cross game chat
- Web client

---

![original](plain-slide.png)

# Could this be stateless?

---

![original](plain-slide.png)

# Possibly

- Chat applications _can_ poll based
- Live caches could hit the database
- Still gross

^ Hitting the db: data scale is very low

---

![original](section-div.png)

# Is stateful better?

^ Than a standard "rails" application

---

![original](section-div.png)

# Do I want this?

^ Maybe
^ When should I go stateful?

---

![original](plain-slide.png)

# Depends

---

![original](section-div.png)

# Pros

---

![original](plain-slide.png)

# Soft realtime capabilities

## Erlang is built for soft realtime

---

![original](plain-slide.png)

# Website feels "alive"

## Websockets can update your website without a refresh

^ Great example: live view

---

![original](plain-slide.png)

# Reduced Database Queries

## Less round trips to the database for faster response times

---

![original](section-div.png)

# Cons

---

![original](plain-slide.png)

# Requires heavy usage of OTP

## You must architect out how your application is laid out in memory

^ Not using the one given to you from Phoenix

---

![original](plain-slide.png)

# Caches are Hard™

## There are 2 hard problems in computer science: cache invalidation, naming things, and off-by-1 errors.

---

![original](plain-slide.png)

# Harder to Deploy

- Warm caches on boot
- Long lived connections will drop
  - How will you deal with this?
- Clustered applications need to handle a rolling restart

---

![original](plain-slide.png)

# Rolling Restart

![inline](rolling-restart.jpg)

^ old and new code live together

---

![original](plain-slide.png)

# Here be Dragons

- Mailbox size
- Message passing *copies*
- Clustering woes

![left](dragon.jpg)

[.footer: https://unsplash.com/photos/vVKbHNhu2ZU]

^ this is a challenge to do and do well

---

![original](section-div.png)

# Let's Go Stateful

---

![original](plain-slide.png)

# GenServers

- The basic building block of OTP
- Handle messages out of the HTTP process
- Background tasks
- Any kind of state you want to hang on to

---
[.text: #666666, alignment(center), Circular Std]
![original](plain-slide.png)

# Supervision Trees

Watch over your GenServers

![inline](supervision.jpg)

[.footer: https://unsplash.com/photos/bt41lw9i6Kc]

---

![original](plain-slide.png)

# :ets

- Erlang Term Storage
- Very fast reads/writes
- Concurrent
- Downside: copies memory a lot

[.footer: http://erlang.org/doc/man/ets.html]

---

![original](section-div.png)

# Put It All Together

^ TODO Lots of diagrams

---

![original](plain-slide.png)

# Diagram of ExVenture

![inline](exventure-diagram.jpg)

---

![original](plain-slide.png)

# Connecting to the World

![inline](exventure-connecting.jpg)

---

![original](plain-slide.png)

# Players Acting on the World

![left](player-acting.jpg)

^ Notice no DB

---

![original](plain-slide.png)

# Updating a Skill from the Admin

![inline](exventure-updating-skills.jpg)

---

![original](plain-slide.png)

# Using a Skill

![inline](exventure-use-skill.jpg)

^ Notice no DB

---

![original](plain-slide.png)

# Grapevine Chat

![inline](grapevine-flow.jpg)

---

![original](plain-slide.png)

# Grapevine Web Telnet Client

![inline](grapevine-browser-reload.jpg)

---

![original](section-div.png)

# Wrap Up

^ TODO roll into intro

^ elixir gives you the tools to build a very stateful application without round tripping to the db each request, performance improvements, easier to diagram, easier to architect/reason about

---

![original](plain-slide.png)

# Elixir gives you the tools to build a stateful application with ease.

^ Easier to architect and reason about

---

![original](plain-slide.png)

# Ram and CPUs are cheap, use 'em

---

![original](plain-slide.png)

# LiveView is an easy way to start stateful

^ And you can continue more stateful by sending messages to that process

---

![original](plain-slide.png)

# Finally, you might not need it

---

![original](plain-slide.png)

# GitHub

- [oestrich on GitHub](github.com/oestrich)
- [oestrich/ex_venture](https://github.com/oestrich/ex_venture)
- [oestrich/grapevine](https://github.com/oestrich/grapevine)

---

![original](plain-slide.png)

[.text: alignment(center)]

# Titans of Text

[titansoftext.com](https://www.titansoftext.com)

![inline](titans.jpg)

---

![original](plain-slide.png)

[.text: alignment(center)]

# Season 3 of Smart Software

[podcast.smartlogic.io](https://podcast.smartlogic.io)

![inline](smart-software.png)

---

![original](plain-slide.png)

[.text: alignment(center)]

# Season 3 of ~~Smart Software~~
# Elixir Wizards

[podcast.smartlogic.io](https://podcast.smartlogic.io)

---

[.header: #FDDF26, text-scale(1.5), Circular Std]

![original](closer-slide.png)

# Thanks!
