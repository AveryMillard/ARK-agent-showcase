# Project History

*A running look at how this project got here, including the parts that didn't work the first time.*

This started in September 2026 as an experiment: could I get an AI to actually play **ARK: Survival Ascended**?

Not just send a few commands, but eventually have its own character, understand what's around it, decide what it needs, and carry out tasks in the game world. ARK seemed like a pretty interesting place to try that. There are resources to find, things to craft, terrain to navigate, and plenty of opportunities for something to go wrong.

This page is the bigger story behind the project. It's organized around the milestones that actually changed how the system worked, rather than every little bug fix or commit. For the current state of the project, start with the [README](README.md).

---

## September 2026 — The original proof of concept

The first version was much simpler than what I'm building now. Python could observe some information from the game and control the **player character** through keyboard inputs.

The early goal was resource gathering: find something useful, move toward it, interact with it, and keep track of whether the task succeeded. I used small test courses to see whether the whole process could work, including a three-tree gathering test that took about **118.5 seconds**.

It was a useful proof of concept, but it had a pretty big limitation: the agent was essentially borrowing the player's body.

That wasn't really what I wanted long-term. I wanted an independent character that could exist **alongside** the player. One that could carry its own items, move on its own, and interact with the same world rather than having Python pretend those interactions happened.

So the next step was giving the agent an actual body.

## September 2026 — Giving the agent a character

I moved into the ARK DevKit and started experimenting with a native character controlled through Unreal Engine Blueprints.

My first approach borrowed behavior from an Ankylosaurus. It made sense at the time: dinosaurs already know how to move around and harvest resources, so maybe I could reuse those systems while giving the agent a human appearance.

Turns out a dinosaur-based character and a human animation setup don't exactly get along.

There were skeleton and animation problems, awkward poses, and a lot of assumptions built into the original character. I spent a while trying to make that setup work before deciding it was creating more problems than it solved.

The eventual solution was a cleaner character based directly on ARK's native dinosaur-character framework, with a human mesh, a small custom animation setup, its own inventory, and native actions.

This was one of the more important architectural changes. Instead of fighting to adapt the wrong base character, I had a much simpler foundation to build on.

## September–October 2026 — Getting real ARK interactions working

The next challenge was making sure the character wasn't just *looking* like it was playing ARK.

If it picked up a stone, I wanted that stone to exist in its actual inventory. If it harvested a bush or hit a tree, I wanted ARK itself to handle the resource and health changes. The Python side shouldn't be inventing those results.

This took more investigation than I expected.

For example, some loose resources and foliage don't behave like ordinary world objects. Regular collision traces weren't giving me the information I needed to interact with them reliably. I eventually had to work through ARK's own foliage-search and harvesting systems, including some rather unintuitive indexing behavior.

Once that was sorted out, native gathering started to work: resources were depleted in the world, real items were delivered to the character, and I could verify the results through game state.

It also reinforced a rule I've tried to follow throughout this project: **let the game handle its own mechanics, and let Python handle the agent's decisions and task logic.**

## October 6 — Movement gets a lot more serious

Walking to a target sounds like it should be one of the easy parts. And sometimes it is! ARK's built-in movement can handle a straightforward route pretty well.

The trouble starts when it can't.

Early movement attempts could get stuck on terrain or obstacles, and simply issuing the same movement request again wasn't much of a solution. I wanted the character to recognize when progress had stopped and try to recover without forgetting where it was originally going.

That became the second version of the native movement system.

The general idea was:

1. Try normal movement toward the destination.
2. Detect when the character isn't making meaningful progress.
3. Stop and search for a safe local detour using a grid and A* pathfinding.
4. Validate the route before following it.
5. Rejoin the original route and continue toward the same goal.

An early implementation could cause noticeable hitches while it worked through potential paths. Moving that validation across multiple frames fixed the worst of those pauses.

By the end of this stage, the character could handle obstacles that had previously stopped it entirely. It still takes an occasional weird route, but being able to recover reliably mattered much more than having perfectly elegant movement.

## October 10 — Connecting Python to the native character

With movement, inventory, and harvesting working in the DevKit, I could finally start connecting those pieces back to the Python controller.

The systems communicate over a local WebSocket connection. Python requests an action, ARK performs it through its native character systems, and state updates or action results come back so Python can decide what to do next.

One of the first useful end-to-end tasks was collecting wood:

**Find a tree → navigate to it → confirm arrival → attack → check the resource/inventory changes → continue or move on.**

This was a pretty big moment for me. The character wasn't following a pre-recorded movement path; it was responding to resource scans and the actual outcome of its actions.

The first native three-tree gathering test took **78.17 seconds**, compared with roughly **118.5 seconds** for the older player-controlled proof of concept. Those weren't intended as a perfectly controlled performance comparison, but they made it clear the new approach was worth pursuing.

## October 11 — Making the gathering loop less clunky

Once gathering worked, I started paying more attention to *how* it worked.

One bottleneck was surprisingly simple: the controller was waiting on resource scans between too many individual attacks. The scans were useful for checking progress, but they didn't need to dictate the timing of every swing.

I separated the attack timing from the slower state-update loop, while keeping checks for stale data and actions that weren't making progress.

The three-tree course went from around **78 seconds** in the first native test to **48.59 seconds** in a later test using a more practical attack interval. The human reference for that short course was around **43 seconds**.

I also tested faster settings, but the quickest-looking result wasn't automatically the most reliable. Some settings started producing apparent swings that didn't connect. I ended up favoring a little extra timing margin over shaving every possible fraction of a second.

The goal isn't necessarily to make the agent faster than a human. I mainly want it to feel competent, reasonably responsive, and able to finish the task without getting stuck.

## October 12 — Collecting all resources

By this point, there were a bunch of individual actions that worked: harvesting bushes, collecting loose stones, equipping a Stone Pick, clearing trees, and mining regular and metal-rich rocks.

Getting each one working separately was nice. Getting all of them to work **in one uninterrupted course** was the real test.

The course uses a fixed sequence:

1. Harvest the bushes.
2. Collect loose stones.
3. Request the Stone Pick.
4. Clear the trees.
5. Mine the ordinary rocks.
6. Mine a metal-rich rock.

Even the loose-stone step had its own little surprise. I initially assumed that interacting near a selected stone meant ARK would collect *that exact stone*. It doesn't always work that way; the game may pick another nearby resource instead. The better approach was to trust real inventory changes, then rescan and reposition when collecting stopped working.

Eventually, the full scripted course passed.

I also recorded a side-by-side comparison with a human player completing the course:

| Run | Completion time |
| --- | ---: |
| Human | **2:38** |
| Autonomous agent | **3:17.04** |

The agent finished in about **1.25× the human's time**. Watching the entire run, the difference felt pretty reasonable. Most individual actions were only a little slower; that time just adds up across a whole course.

**[Watch the full human-vs-agent comparison](https://github.com/AveryMillard/ARK-agent-showcase/releases/tag/demo-001)**

### An important distinction

This course is a **scripted integration test**, not a demonstration of independent AI planning.

I defined the order of the stages ahead of time. The character handles the actual navigation, interactions, harvesting, and checking of results, but a higher-level AI isn't yet deciding what to do next from an open-ended goal.

I think that's an important distinction to make, because the next stage of the project is specifically about closing that gap.

---

## Where the project goes next

Now that the individual actions are working together, I want to give the planning side more responsibility.

Instead of handing it the entire gathering course in advance, I'd like to be able to say something like **"let's get some basic tools"** and have the agent figure out the necessary steps: what resources it needs, what to gather first, whether it already has the right equipment, and what to try when something doesn't work.

Some of the next pieces include more general inventory reporting, clearer success/failure feedback from native actions, crafting, and a way for the agent to chain smaller actions into useful routines.

One of the things I like most about this project is that the higher-level agent architecture doesn't have to stay tied to ARK forever. The game-specific interactions will change, but the ideas behind task decomposition, planning, recovery, and reusable routines should be useful elsewhere too.

That's the hope, anyway. There's still quite a lot to build.

---

*This is a public project journal. The development source code and DevKit project are currently maintained separately and aren't included in this repository.*
