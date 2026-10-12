# ARK Autonomous Agent

*An experimental AI companion project for ARK: Survival Ascended.*

I've been working on an agent that can have its **own character inside ARK**. Not just a script pressing buttons for my player, but a separate character that can navigate the world, find resources, use real items, and eventually make its own decisions about how to complete a task.

The long-term idea is pretty simple to describe: I want to be able to say something like **"let's go get some basic tools"** and have the agent work out what that actually involves.

Getting there is a lot less simple, which is most of the fun (and debugging) of this project.

This repository is the public-facing side of the experiment: demonstrations, progress updates, and a history of what I've built so far. The actual development code and ARK DevKit project are kept separately for now.

## Latest milestone — Collecting All Resources Course

**October 12, 2026**

I recently got the first full resource-gathering course working from start to finish. It combines several actions that I'd previously only tested on their own: gathering foliage, picking up loose stones, equipping a tool, harvesting trees, and mining stone and metal-rich rocks.

To see how it compared to actually playing the game, I recorded myself completing the same course and put the two runs side by side.

| | Completion time |
| --- | ---: |
| **Human player** | **2:38** |
| **Autonomous agent** | **3:17.04** |

The agent took about **1.25× as long** as the human run. I'm pretty happy with that for now. There are still some slightly odd movement choices here and there, but the full sequence works, and it doesn't feel painfully slow to watch.

**[Watch the human-vs-agent comparison and read the release notes →](https://github.com/AveryMillard/ARK-agent-showcase/releases/tag/demo-001)**

[Download the full MP4](https://github.com/AveryMillard/ARK-agent-showcase/releases/download/demo-001/human-vs-agent-comparison.mp4) *(around 285 MiB)*

**One important clarification:** this is a **scripted course**, meaning I defined the sequence of tasks in advance. The agent carries out the navigation and interactions using the game world and real feedback, but it isn't independently planning the whole course yet. That's the next big goal.

## How it works

The project currently has two main sides:

- **ARK / Unreal Engine Blueprints:** The character itself, movement, animation, native inventory, resource detection, harvesting, and interaction with the game world.
- **Python:** A controller that communicates with ARK through a local WebSocket bridge, requests actions, watches the results, and coordinates tasks.

Some of the trickier parts have been getting native ARK harvesting to work correctly, building movement recovery for when normal navigation gets stuck, and making sure an action actually succeeded instead of just assuming it did.

The bigger design goal is to keep the agent's planning separate from the game-specific mechanics. ARK is the testing ground, but I'd love for the higher-level logic to eventually be reusable in other projects.

## Current progress

So far, the agent can:

- Exist as its own native character, with real inventory and equipment interactions.
- Detect nearby resources and observe changes to inventory and resource health.
- Gather foliage and loose stones, harvest trees, and mine rocks with native game interactions.
- Navigate to targets, with local A* recovery when direct movement gets stuck.
- Run a complete multi-stage gathering course without me manually steering it through each action.

**Still to come:** independent high-level planning, more general inventory and crafting support, stronger action feedback and recovery, and eventually reusable routines the agent can assemble for itself.

## Want the longer story?

**[Read the project history →](PROJECT_HISTORY.md)**

That page starts with the original keyboard-controlled proof of concept and walks through the move to a native character, the less-successful experiments, pathfinding, harvesting, and the first full benchmark.

I'll keep adding demonstrations and development notes here as the project grows. This is as much a record of figuring things out as it is a showcase of the parts that work.

---

*Unofficial personal engineering project built around ARK: Survival Ascended. Not affiliated with the game's developers.*
