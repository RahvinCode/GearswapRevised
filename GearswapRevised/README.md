# GearswapRevised

GearswapRevised is my fork of GearSwap, built on GearSwap 0.940 and meant to be used in place of GearSwap's Live and Dev versions. It runs the same job files and answers to the same commands as Gearswap. What it changes is how GearSwap keeps track of the items in your bags, which makes GearSwap's own work several times cheaper. Version 0.960 also brings in upstream GearSwap's fixes through its development branch of 2026-10-20 and trims more of GearSwap's routine work, with every gear choice unchanged.

## Load GearswapRevised instead of Gearswap. 

**GearswapRevised and GearSwap are mutually exclusive. Never load them at the same time.** Both answer to the `gs` and `gearswap` commands, both handle every game event, and both load your job files and send equip commands. With both loaded, every event is handled twice and the two fight over your gear. Unload GearSwap before you load GearswapRevised, and make sure nothing loads GearSwap again when the game starts.  Turn off the auto load for Gearswap within your Windower profile and add "lua load GearswapRevised" to your init.txt without the quotes.

## What's Different with GearswapRevised?

- **Your bags are not rebuilt at every refresh.** Before GearSwap hands an event to your job file, it refreshes what it knows about your character. The live version rebuilds every bag you own, item by item, at every refresh, whether anything changed or not. GearswapRevised keeps each bag's item list between refreshes. A bag nothing has changed is not touched, a changed item is updated in place, and a bag is rebuilt only when the items in it may have changed names.
- **Gear is found through an index.** Around every event, the live version walks all 720 rows of your equippable bags, the inventory and eight wardrobes, looking for the gear your job file asked for, even when it asked for nothing. GearswapRevised keeps an index of those bags by item name and goes straight to the rows that hold the items asked for, in the order the full walk would reach them. So it picks the same copy of an item you carry twice.
- **Your job files see one change.** In the live version the bag tables under `player` (`player.inventory`, `player.wardrobe` and the rest) are new tables after every refresh. In GearswapRevised each one stays the same table until that bag changes, and is updated in place. A job file that only reads them sees no difference. A job file that writes into them finds its writes still there until that bag next changes. Changes a job file makes directly to GearSwap's internal item cache are not picked up.
- **It has its own data folder.** GearswapRevised looks for job files in its own `data` folder (`addons/GearswapRevised/data`), and in the same `%APPDATA%/Windower/GearSwap` folders the live version uses.

## What's new in 0.960

**Upstream GearSwap's fixes, through its development branch of 2026-10-20.** GearswapRevised now carries every GearSwap change made upstream since the 0.950 release:

- **Items used on an NPC or a Trust.** An `/item` command aimed at an NPC or a Trust counts as a trade only when the item sits unequipped in your main inventory. Otherwise it is an ordinary item use, and your job file sees it as one.
- **A trade beyond 6 yalms goes to the game to judge.** 0.950 answered it with its own "Target out of range." message.
- **A target changed in pretarget** is labeled again, so an `/item` command moved to an NPC in pretarget is judged at its new target.
- **Aura refreshes no longer fire buff change events.** A short aura the server refreshes is matched to the one already up instead of being reported as lost and gained again.
- Upstream's cleanup of how typed commands, Monstrosity TP moves and ranged attacks are read.

**Less routine work, the same choices.** Each of these removes work GearSwap repeated without need:

- **Other players' actions are not decoded.** Every action packet in range used to be decoded in full before GearSwap checked whose it was. Now a melee round, or an action by anyone but you and your pet, is set aside after reading two numbers from the packet.
- **Names are converted once.** Buff names, bag names and item names were lowercased again at every refresh and every event. Each is now converted once per language and remembered.
- **Augments are decoded once.** A set that names augments made GearSwap decode the worn item's augments at every event. Each item's decoded augments are now remembered until the item changes.
- **Fewer temporary tables.** Debug text is built only when debugging is on, each buff shares one metatable, a confirmed equip clears its pending list in place, and `equip()` and `set_combine()` called with one or two sets skip three helper tables.

Three of these came from Voli and Az's pull request: the shared buff metatable, the remembered item names, and the pending list cleared in place. Thank you.

These trims are not yet timed in game; the figures below are 0.950's.

## What it gains

**[The full comparison, with charts](https://rahvincode.github.io/GearswapRevised/)** sets GearswapRevised beside the previous and current live versions under each of three GearSwap suites: what each costs, what that cost is made of, per character and in memory.

I have extensively tested this with offline simulations and in-game benchmarks. Two versions of Gearswap and the release version of GearswapRevised were timed in the game on a six character multibox at a busy Locus imp camp, and those timings were priced into a simulated Dynamis Divergence fight of six clients in an 18-player alliance, run under three GearSwap suites (Mirdain 1.5.12, Rahvin GS 2.1 and Selindrile). Against the public release of 2026-09-30:

| | Public release | GearswapRevised 0.950 |
|---|---|---|
| A full refresh, per character | 2.39 to 4.29 ms | 0.74 to 0.80 ms |
| An event that asks for no gear | 0.87 to 0.98 ms | 0.11 to 0.13 ms |
| An event that changes gear | 3.27 to 4.38 ms | 0.36 to 0.43 ms |
| Memory a full refresh allocates | 264 to 466 KB | 116 to 121 KB |
| Worst frame on the 60 fps client, engine work against its 16.7 ms | 130% to 295% | 19% to 48% |

Across the three suites in both of their states, the six clients' engine CPU falls 75% to 82% and the memory they allocate 58% to 70%. Under the public release the worst frame on the 60 fps client takes more work than a frame lasts on every suite. Under GearswapRevised it takes less on every suite. The frame figures are engine work set against a frame's length, not a measured frame rate.

The two versions were timed in separate sessions on the same evening, the public release two hours later, when the same code read up to 23% slower. GearswapRevised's gains stand far beyond that. It's difficult to get an exact like for like test in a live environment, but the gains are so substantial that they fall beyond any standard error constraints.

## Does it do the same thing as Gearswap?
It makes the same choices and equips the same items, just more efficiently. In a 55-minute in-game logged and monitored test, GearswapRevised's index ran beside GearSwap's walk at all 30,878 events and chose the same gear every time. Outside the game its bag lists and its index matched GearSwap's over a simulation of 120,000 random bag changes and 18,000 equip events.

0.960's trims were held to the same rule: no change to which gear is chosen or to when any job file hook fires, for any suite. Each was tested outside the game against the code it replaces, on the same inputs, and shown to fail when a difference was planted in it. Then six characters fought for about 22 minutes with the old and the new code running side by side at every call: 1,123,231 comparisons, every one identical.

## What it costs

Gains in one area often come at a cost in another. As long as the costs are manageable and negligible to the user environment, I consider those to be worthwhile tradeoffs. The cost for these efficiency gains is miniscule for modern systems and is detailed below.  This is essentially free gains if you're already running Gearswap. For a reference point, memory churn is much more impactful than total memory held, and memory churn decreased drastically with this revision.  That said, here's the tradeoff:
- **More memory held.** The kept bag lists carry a small record each, and the index is new. In the game together they hold about 75 to 167 KB more per character than the live version, and scales with the size of a character's total inventory.  75 KB is around 400 items while 170 KB is around 820 items total in inventory.
- **The item-cache writers**, which run as inventory and equip packets arrive, took 62 to 113 µs a second per character in the measured session on GearswapRevised, 9% to 31% more than under the public release, since each write is now also noted for the kept bag lists. That is 6% to 11% of GearSwap's much smaller total.
- **0.960's remembered names and augments** add a small table each: one entry per buff, bag and item name seen, and one per augmented item worn or carried, the augment table starting over past 4,096 entries.

Rahvin GS 2.2.0 is included in the data folder.  It's safe to leave there if you're using a different suite.  You can delete RahvinGS and Sample Job Files if they're not wanted. See https://github.com/RahvinCode/Gearswap for more information.
