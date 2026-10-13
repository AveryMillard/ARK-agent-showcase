# ARK Autonomous Agent

This is a project I've been messing around with to see how far I can get toward having an AI actually play ARK: Survival Ascended alongside me.

The idea has been in my head for years. Back when AI Dungeon and Character.AI were pretty new, me and a friend started talking about how cool it'd be if AI characters could eventually play games with you. ARK seemed like a particularly interesting one because of its story. I remember reading the explorer notes on the first map and imagining what it would be like if the other survivors actually behaved like people trying to survive, instead of players who already know they're in a game.

So that's basically what I'm working toward. I want to give an AI its own character, let it interact with the same world as me, and eventually be able to tell it something like "let's go get some basic tools" without having to explain every individual step.

I finally started building it in September 2026. Right now most of the work is getting the game side reliable enough that a higher-level AI will actually have something to work with.

## Latest demo: Collecting All Resources Course

I recently got a full resource gathering course working and decided to record myself doing the same course to see how the two compared.

The course has the agent gather from bushes, collect loose stones, equip a Stone Pick, chop down trees, mine regular rocks, and finally mine a metal-rich rock.

| Run | Time |
| --- | ---: |
| Human | 2:38 |
| Autonomous agent | 3:17.04 |

So the agent took about 1.25x as long as I did. I'm pretty happy with that, honestly. There were a couple weird pathfinding moments, but watching it didn't feel like I was waiting forever for it to do something. It's mostly a few extra seconds here and there that add up over the course.

[Video and release notes](https://github.com/AveryMillard/ARK-agent-showcase/releases/tag/demo-001)

[Full comparison video (MP4)](https://github.com/AveryMillard/ARK-agent-showcase/releases/download/demo-001/human-vs-agent-comparison.mp4)

One thing worth mentioning is that this is a scripted course. I decided the order of the tasks ahead of time. The agent handles moving around, interacting with resources, and checking the results, but there isn't a high-level AI independently planning the whole thing yet. Getting to that point is what I want to tackle next.

## How it works right now

I've got the project split between ARK's DevKit and Python.

On the ARK side, there's a separate character with its own inventory, equipment, movement, and harvesting. I wanted those interactions to use the game's actual systems rather than just simulate collecting items in Python. Movement mostly uses ARK's built-in navigation, with some local A* pathfinding to recover when it gets stuck.

Python talks to the game through a WebSocket connection. It can request actions, read information about nearby resources and inventory, and keep a task going based on what actually happened.

Eventually I want the LLM side to handle the larger decisions, while the faster, more reactive stuff stays closer to the game. I've also been thinking about how much of that higher-level logic could be reused outside ARK, since the planning side shouldn't need to know about the game's internals.

There's still a lot left to do. Better action feedback, a more general inventory system, crafting, and especially letting the AI make its own plans are all on the list.

I also wrote up a [project history](PROJECT_HISTORY.md) with the earlier experiments and some of the stuff that went wrong along the way. I'll probably keep using this repo for videos and updates as I get more working.

This is an unofficial personal project, not affiliated with the developers of ARK.
