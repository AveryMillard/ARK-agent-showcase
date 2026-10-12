# Project History

I figured I'd keep a bit of a history of this project, because the original idea and what I've actually spent the last month building are two pretty different things.

I started working on it in September 2026, but I'd been thinking about it for years before that.

## Before I started coding

I think I was initially inspired by AIs being able to take on the roles of characters. Back when stuff like AI Dungeon and Character.AI was new, I started imagining a future where AI could actually play video games with you.

Me and a friend ended up talking about ARK specifically because of its story. I remember reading the explorer notes on the original map and thinking about how cool it would be to experience something like that for yourself.

The problem was that I also understood pretty well that human players weren't going to act like the people in those notes, because they know they're playing a game. And option B of having actual people on an island full of dinosaurs was pretty obviously out of the question XD

So option C was basically to wait around until AI got good enough to try something like it.

My original idea was pretty much "show an AI every frame, give it a personality, and convince it what it's seeing is real." But even years before I started coding this, I knew the whole 60 screenshots a second thing wasn't going to work. There's no way you can expect an LLM to process that much visual information and still react quickly enough to play a game.

I figured there needed to be some kind of reactive system sitting between the game and the LLM's higher-level thinking. The fast stuff, like movement and responding to what's happening right now, would be handled separately from the bigger decisions.

That's still what I'm working toward. Funny enough, the high-level AI isn't actually integrated yet. Most of the work so far has been making the game side capable of doing the things I'd eventually want to ask it to do.

## September 2026: The first proof of concept

When I finally started making this, the first version controlled my own player character.

I had Python receiving information from ARK and issuing keyboard inputs, which was enough to start testing the basic idea. The early tests were mostly about finding resources, moving over to them, interacting with them, and checking whether any of it actually worked.

One of the first little benchmarks was collecting wood from three trees. The old version finished that in about 118.5 seconds.

It wasn't exactly fast, but it did give me something concrete to work with.

The bigger problem was that it was controlling me, not a separate character. I'd always imagined this as another survivor playing alongside you, and borrowing my keyboard wasn't really going to get me there.

So I moved over to the ARK DevKit to see if I could give the agent its own character.

## September 2026: Making an actual character

This part got complicated pretty quickly.

ARK already has a bunch of native systems for creatures to move around, attack, harvest resources, and hold items. I figured I could take advantage of those rather than rebuild all of ARK's mechanics in Python.

My first attempt used an Ankylosaurus as a sort of donor character. Since an Anky already interacts with resources, I thought I could borrow those systems and then put a human character on top.

That worked about as well as you might expect once animations got involved.

I could get a human mesh to appear, but the skeleton, animation blueprint, and underlying dinosaur behavior didn't all agree on how a person was supposed to move. At one point I had a character who was technically idling but whose legs were stuck in an A-pose. I spent a while trying to work around that before deciding the whole setup was more trouble than it was worth.

Eventually I went with a cleaner character based directly on ARK's native character framework, using a human mesh and a much smaller custom animation setup. It still uses native ARK mechanics, but without all the extra assumptions from the Ankylosaurus.

That gave me a base to start adding inventory, attacks, equipment, and actual resource interactions.

## September to October: Getting harvesting to work

One thing I wanted to avoid was having Python pretend something happened when ARK didn't actually do it.

If the character picked up a rock, it needed to be a real item in its inventory. If it harvested a bush or chopped down a tree, the resources needed to come from the actual game world.

I thought some of that would be relatively straightforward since the mechanics already existed in ARK. Turns out there were a lot of details hiding behind them.

Loose stones and bushes were especially annoying. The normal world traces I tried weren't reliably finding the foliage instances that ARK itself could interact with. Eventually I had to use ARK's own foliage-search and harvesting systems, including figuring out some weird indexing behavior in the results.

There were also cases where a resource would lose health or disappear but the items didn't end up in the right inventory. So just seeing a harvesting animation, or even seeing the resource disappear, wasn't enough proof that the action was working properly.

Once that was figured out, I could actually see the character collect resources and carry them around in its own inventory. That was a lot closer to what I'd been trying to make.

## October 6: Movement and pathfinding

Movement was another one of those things that seemed simple until I started testing it.

ARK's built-in MoveTo works fine when it can find a normal route. But if the character gets stuck trying to reach something, sending the same request over and over isn't very helpful.

I ended up building a recovery system around that. Most of the time it still uses ARK's normal navigation. If it stops making progress, it pauses, searches a local grid for a way around the obstruction using A*, checks that the route is actually safe, and then tries to rejoin the path toward its original destination.

I specifically didn't want it to forget its original goal just because it ran into something.

The first versions weren't particularly graceful. I had a point where checking possible paths would cause a big hitch in the editor because it was doing too much work at once. Eventually I spread that validation across multiple frames so it could think through a route without freezing everything.

It isn't perfect. Sometimes it takes a slightly weird route when a shorter one looks obvious to me. But it can recover from obstacles that would have stopped the earlier movement system entirely, so I was pretty happy to leave the smaller route-quality problems for later.

## October 10 and 11: Making Python control all of this

At this point I had a character that could move, harvest, and use an inventory, but I still needed the Python side to actually tell it what to do.

The two sides communicate over a local WebSocket connection. Python can request an action, and the ARK side can send information back about movement, nearby resources, inventory changes, and other results.

The first proper gathering loop was for wood. Python would find a tree, send the character toward it, wait for movement to finish, start attacking, and check whether the tree was getting harvested.

The first native three-tree test finished in 78.17 seconds, compared with 118.5 seconds for the old player-controlled version. I wouldn't call those perfectly controlled comparisons, but it was still a pretty nice improvement.

Then I noticed a lot of extra time was coming from waiting for resource scans in between attacks. The game didn't need to rescan everything after every single swing, so I separated attack timing from the slower process of checking inventory and resource health.

After a bunch of tests with different intervals, I got the three-tree course down to 48.59 seconds with a timing setting I was comfortable using. For comparison, my own human run of that smaller course was around 43 seconds.

I did get faster experimental results, but some of those settings appeared to produce swings that didn't actually connect. I decided I'd rather keep a little extra timing margin than squeeze out every possible second.

Plus, I'm not really trying to make the agent faster than a person at everything. If it's only a little slower and doesn't get stuck constantly, that's already a pretty good result to me.

## October 12: Putting the actions together

Once wood gathering was working, I started getting the other resources and actions into place.

That meant bushes, loose stones, regular rocks, metal-rich rocks, and equipment changes. Most of them had their own little problems.

The loose stones were a good example. I initially assumed that if Python picked a specific stone to collect, pressing the interaction key near it would collect that exact stone. But ARK can decide to pick up a different nearby stone instead, which made my tracking logic pretty confused.

The fix was to stop assuming I knew exactly which stone had been collected. If the inventory count was going up, the action was working. If it stopped going up, then it was time to scan again and find somewhere else to try.

After getting those individual pieces working, I put together a full resource gathering course. The scripted order was bushes, loose stones, equipping a Stone Pick, trees, regular rocks, and finally one metal-rich rock.

The first complete run passed in 2:57.87. I later recorded another run alongside a human attempt so I could put them side by side.

| Run | Time |
| --- | ---: |
| Human | 2:38 |
| Autonomous agent | 3:17.04 |

The agent took about 1.25 times as long as I did. Watching the recording, I didn't really get the feeling that it was taking forever. It had two slightly strange pathfinding moments, but most of the difference was just small delays across a lot of individual actions.

[Here's that comparison video](https://github.com/AveryMillard/ARK-agent-showcase/releases/tag/demo-001).

I do want to be clear about what this test is, though. The order of the tasks was scripted in Python. The character was doing the actual moving, interacting, harvesting, and checking results in ARK, but it wasn't an LLM independently figuring out the course from scratch.

That's a separate milestone, and one I haven't reached yet.

## What I'm planning next

Now that the game side can complete a full sequence, I want to start giving the higher-level AI more responsibility.

The example I keep coming back to is telling it "let's get some basic tools" and having it figure out what to do from there. It should be able to check what it has, decide what it still needs, gather materials, craft what it can, and change plans when something doesn't work.

Before that, there are still some less exciting things to sort out, like more general inventory reporting, better confirmation that actions succeeded or failed, and a proper crafting interface.

Longer term I'd also love for the AI to start building reusable routines of its own. Instead of having to explain how to collect wood every single time, it could learn that a sequence of smaller actions is useful and reuse it later.

I also like that the Python side of this doesn't necessarily have to be tied to ARK forever. Obviously the game-specific actions won't carry over directly, but a lot of the higher-level planning and routine logic might.

For now, though, I've got plenty left to do in ARK.

The actual source code and DevKit project are kept separately. This repo is where I'm keeping the public demos and notes as I go.
