# ARK Autonomous Agent

A personal game AI and systems-integration project exploring how an autonomous character could play ARK: Survival Ascended alongside a human. Built with Python, ARK's DevKit, and a WebSocket bridge, it currently demonstrates scripted resource gathering with game-state feedback and an initial crafting interface.

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

## Crafting: implementation and validation status

As of the October 18, 2026 Python update, crafting is implemented at the controller level for **Stone Pick and Stone Hatchet**. That is a narrower milestone than an agent independently deciding which tools it needs and gathering the ingredients.

| Capability | Current evidence |
| --- | --- |
| Stone Pick crafting in ARK | Successful live crafting was recorded in earlier project documentation. |
| Python support for Stone Pick and Stone Hatchet | Both recipes are configured, with multiple accepted command aliases for each. Requests craft one item at a time. |
| Python verification behavior | Eight crafting tests focus on Stone Pick, using simulated game responses and inventory updates. These are controller tests, not live DevKit integration tests. |
| Stone Hatchet end-to-end integration | Python configuration is present; successful live crafting through the ARK DevKit bridge still needs independent validation. |
| Autonomous tool planning | Future work. The current controller does not independently plan a gather-and-craft workflow. |

### How a craft is confirmed

Python sends a crafting request over the WebSocket bridge; the ARK Blueprint side is responsible for the actual inventory transaction. Python does not subtract ingredients or award a pretend output. The game's recipe allowlist remains authoritative, so adding a Python recipe entry alone does not establish working game integration.

The controller checks the outcome in two stages:

1. **Match the response to the request.** Each request has a unique ID. The controller waits for a response with that ID and checks that the item and quantity match what it requested.
2. **Verify the inventory change.** A “created” response alone is insufficient. The controller requires an inventory snapshot received after the creation response, showing exactly one more of the requested item than the count captured before sending the request. Only then does it report verified success.

This separates “the command was sent,” “the game reported creation,” and “the output was observed in inventory.” Rejections are reported as such. Missing responses or inventory confirmation leave the outcome unconfirmed; the controller warns against automatic retries because the game may already have completed the transaction. It also blocks overlapping crafting requests.

The existing tests cover confirmation after an inventory increase, delayed inventory updates, missing-material rejection, duplicate requests, missing inventory telemetry, unsupported recipes, missing responses, and malformed responses. They support the Python verification design; they do not prove every configured recipe works in-game.

## Next steps

The immediate crafting milestone is live Stone Hatchet testing through the DevKit bridge, with recipe-specific test coverage to accompany it. Broader inventory handling, better action feedback, and higher-level planning remain ongoing work. The longer-term goal is still to ask for basic tools and have the agent choose and execute the necessary steps.

The implementation source and DevKit project are maintained separately; this repository shares demonstrations, architecture, and development notes.

I also wrote up a [project history](PROJECT_HISTORY.md) with the earlier experiments and some of the stuff that went wrong along the way. I'll probably keep using this repo for videos and updates as I get more working.

This is an unofficial personal project, not affiliated with the developers of ARK.
