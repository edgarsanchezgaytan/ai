# Activity 5 — Reflection Questions: The Whispering Cave

**Name:** Edgar Jesús Sánchez Gaytán
**Course:** Universidad Anáhuac Mayab — Ingeniería en Tecnologías de la Información y Negocios Digitales
**App used:** https://uam-aiclass-a5.streamlit.app/

---

## Part 1 — Facts, Rules, and Recursion (Session 10)

**1. Write out one `tunnel(X, Y)` fact directly from the Tunnel Rules tab, and report whether `reachable(Entrance, Treasure Room)` came back TRUE or FALSE when you asked the Oracle.**

One fact straight from the Tunnel Rules tab is:

```
tunnel(bat_roost, entrance).
```

When I asked the Oracle about `reachable(Entrance, Treasure Room)`, it came back **TRUE**. The derivation the app printed followed this chain: `entrance → bat_roost → spider_tunnel → old_mine_shaft → bottomless_pit` (dead end, backtrack), then back up to `bat_roost → echo_chamber → torch_hallway → crystal_cavern → treasure_room`, where the base case finally fired because `tunnel(crystal_cavern, treasure_room)` exists directly. So the Oracle had to backtrack out of one failed branch before it found the successful one, but it still confirmed the two chambers are connected.

**2. Which of the two `reachable` lines is the base case, and which is the recursive case? Explain what makes the second line "recursive."**

The base case is `reachable(X, Y) :- tunnel(X, Y)`. It says "X can reach Y if there is a direct tunnel from X to Y" — no further searching needed, the answer is right there in one fact.

The recursive case is `reachable(X, Y) :- tunnel(X, Z), reachable(Z, Y)`. It says "X can reach Y if there is a tunnel from X to some other chamber Z, and — from Z — Y is also reachable." What makes this recursive is that the definition of `reachable` calls `reachable` again inside itself, just with a smaller/closer version of the problem (one tunnel closer to the goal). Each time the rule fires, it peels off one tunnel and hands off a slightly shorter question to a fresh copy of itself, until eventually one of those copies lands exactly on the base case (a direct tunnel to the goal) or runs out of tunnels to try and fails.

---

## Part 2 — The Search Space (Session 11)

**3. What is the initial state and the goal state in this activity?**

The initial state is **Entrance** — that's where the explorer starts, flashlight in hand. The goal state is **Treasure Room** — that's the chamber the search is trying to reach. Everything the app does, from the Prolog-style `reachable` queries to the DFS step-through, is ultimately just different ways of answering the same question: can we get from the initial state to the goal state, and if so, how?

**4. How many total possible paths exist from the Entrance to the Treasure Room? List every path.**

According to the app's brute-force count, there are **exactly 2** possible paths (with no repeated chambers) from Entrance to Treasure Room:

1. `Entrance → Torch Hallway → Crystal Cavern → Treasure Room` (3 tunnels)
2. `Entrance → Bat Roost → Echo Chamber → Torch Hallway → Crystal Cavern → Treasure Room` (5 tunnels)

**5. Explain the difference between the search space and the single path a search algorithm like DFS actually finds. Why can these be different?**

The search space is the complete map of *every* route that could ever be walked from the initial state to the goal — in this cave, that's both of the paths listed above (and, more generally, every dead-end detour too). It's the full set of possibilities, laid out all at once, the way you'd see it if you could look at the whole cave from above.

A search algorithm like DFS, on the other hand, doesn't get that bird's-eye view. It only sees one chamber at a time and has to decide, on the spot, which tunnel to try next. So DFS ends up walking through only *one* actual sequence of chambers — the one path it happens to commit to based on the order it explores things — even though many other paths exist in the search space. These can be different because DFS has no way of comparing paths before it starts; it just goes as deep as it can down whichever tunnel it picks first, and it stops the instant it reaches the goal, whether or not that happened to be the best route available in the search space.

---

## Part 3 — Depth-First Search (Session 12)

**6. List, in order, every chamber DFS actually visited before finding the treasure. How many chambers did it visit in total?**

In order, DFS visited:

1. Entrance
2. Bat Roost
3. Spider Tunnel
4. Old Mine Shaft
5. Bottomless Pit
6. Echo Chamber
7. Torch Hallway
8. Crystal Cavern
9. Treasure Room

That's **9 chambers visited** out of the 10 chambers that exist in the cave.

**7. Name the chamber where DFS hit a genuine dead end and had to backtrack. Separately, name the chamber that was never visited at all, and explain why not.**

The genuine dead end was **Bottomless Pit**. The app's own message at that step said: *"Bottomless Pit is a dead end — nowhere new to go. Time to backtrack!"* — DFS had gone Entrance → Bat Roost → Spider Tunnel → Old Mine Shaft → Bottomless Pit and simply ran out of new tunnels, so it had to pop back up the stack and try a different branch.

The chamber that was never visited before the treasure was found is **Underground Lake**. Underground Lake only connects to Torch Hallway. By the time DFS finally reached Torch Hallway (through the long detour via Bat Roost → Echo Chamber), it pushed both Underground Lake and Crystal Cavern onto the stack, but Crystal Cavern happened to end up closer to the top. DFS explored Crystal Cavern first, found the Treasure Room immediately through it, and the search stopped right there — so Underground Lake was still sitting in the stack, never popped, when the algorithm quit.

**8. What is the exact path DFS used to reach the treasure? Is it the same as the shortest path from Question 4? If not, explain why DFS doesn't always find the shortest path.**

The exact path DFS walked was:

```
Entrance → Bat Roost → Echo Chamber → Torch Hallway → Crystal Cavern → Treasure Room
```

This is **not** the shortest path. The shortest path (from Question 4) is only 3 tunnels long: `Entrance → Torch Hallway → Crystal Cavern → Treasure Room`. DFS instead took the 5-tunnel route.

This happens because DFS doesn't know or care about distance — it just commits to the first unexplored tunnel it sees and dives as deep as possible down that choice before trying anything else. At the Entrance, DFS happened to push/explore the Bat Roost branch before the Torch Hallway branch, so it went all the way down that side (even detouring into the dead-end Bottomless Pit) before eventually looping back around through Echo Chamber and Torch Hallway to reach the treasure. The moment it reaches the goal, it stops — it never goes back to check whether a shorter route existed. That's the key trade-off of DFS: it's simple and guaranteed to find *a* path if one exists, but not necessarily the *shortest* one.

---

## Part 4 — Synthesis

**9. Explain, as if to a 10-year-old, why the recursive `reachable(X, Y)` rule and the stack-based search from Session 12 are really "the same idea" wearing two different costumes.**

Imagine you ask a friend, "Can you get from your house to the park?" Your friend doesn't magically know the answer — they check: "Is there a street straight to the park? No? Okay, is there a street to my neighbor's house? Yes! Let me ask my neighbor the exact same question: can *you* get to the park?" And the neighbor does the exact same thing, asking the next person down the street, and so on, until someone can finally say "yes, the park is right here!" — and that "yes" travels all the way back to you.

That's exactly what the `reachable(X, Y)` rule does: it keeps asking a smaller version of the same question, one tunnel at a time, until it hits a chamber with a direct tunnel to the goal.

Now imagine instead of asking friends, you're the explorer yourself, and every time you reach a new chamber with more than one tunnel, you write the other tunnels down on a sticky note and stick it to your flashlight before picking one to walk down. If you hit a dead end, you peel off the top sticky note and try that tunnel instead. That pile of sticky notes is the stack from Session 12.

Both are doing the identical thing — trying one tunnel, and if it doesn't pan out, going back to the last place you had another option and trying that instead — just described two different ways: the recursive rule describes it as "questions asking smaller questions," and the stack-based search describes it as "notes piling up and getting peeled off." Same idea, different costume.

**10. Name one real-world use of Depth-First Search other than cave/maze exploration, and briefly explain how "go as deep as possible, then backtrack" applies.**

One real-world use of DFS is **exploring a computer's file system**, for example when an antivirus program scans every file on a hard drive. Starting from the root folder, the program opens the first subfolder it finds and immediately dives into *its* first subfolder, and then that subfolder's first subfolder, going as deep as possible down one branch of folders before it ever looks at a sibling folder. Once it reaches a folder with no more subfolders inside (the "dead end"), it backtracks up one level and moves on to the next unexplored subfolder at that level, repeating the process. This is exactly the same "go deep, then backtrack" behavior as the cave explorer with one flashlight — the program only has one "path" it can follow at a time, so it fully exhausts one branch of the folder tree before trying the next.
