<p align="center">
  <img src="docs/hero.svg" alt="ShapeShifter Solver: paste the board, get the answer, squares light up" width="100%">
</p>

<p align="center">
  <a href="https://fang89.github.io/shapeshifter-solver/"><b>▶ Open the solver</b></a>
  &nbsp;·&nbsp; <a href="#how-to-use">How to use</a>
  &nbsp;·&nbsp; <a href="#how-it-solves-a-board">How it works</a>
  &nbsp;·&nbsp; <a href="#free-and-unlocked-levels">Unlock codes</a>
</p>

---

**ShapeShifter Solver** works out the answer to a Neopets ShapeShifter board and shows you
where each piece goes. It's one web page. The solver runs on your own computer, so nothing
is uploaded. Levels 1–39 are free. For level 40 and up, email **gregoryfang89@gmail.com**
for an unlock code.

## Free and unlocked levels

| Levels | Access |
|---|---|
| **1 – 39** | Free |
| **40 and up** | Unlock code |

Enter your unlock code under **License type** at the top right of the solver.

## How to use

| | Step | What you do |
|:-:|---|---|
| **1** | **Copy the board** | Open the game from the solver page, press <kbd>Ctrl</kbd>+<kbd>A</kbd> then <kbd>Ctrl</kbd>+<kbd>C</kbd> (<kbd>⌘</kbd> on a Mac), come back and press <kbd>Ctrl</kbd>+<kbd>V</kbd>. |
| **2** | **Solve** | It starts by itself. Most boards take a second or two. Big late-game boards take longer. |
| **3** | **Place the pieces** | Follow the answer on the solver page, **or** drag the orange **Level N Solution** button to your bookmarks bar and click it once in the game tab. The square to click then lights up after every piece. |

> The solver only highlights squares. It never clicks for you and never sends a score.

## How it solves a board

<img src="docs/rules.svg" alt="Worked example: a 4×4 board solved by four pieces" width="100%">

Once you see the rules as numbers, the puzzle turns into arithmetic. Each square needs a number
of hits, from 0 up to the cycle length − 1. Each piece adds one hit to every square it covers. A
square that gets a full lap of extra hits ends up where it started. So the goal is to place the
pieces so that, on every square, the total number of hits matches what that square needs
(counting in laps).

The order of the pieces doesn't change the final board, only where each one goes. That's what
lets the solver use the shortcuts below.

### 1 · Count before searching

<img src="docs/budget.svg" alt="Hit budget: board needs 11 hits, pieces deliver 14, spare 3 = one full lap" width="100%">

Before trying any position, the solver knows how many "spare" hits the pieces bring. Every move
spends from that budget. It drops a branch straight away if:

- it spends more than the budget allows, or
- a square still needs *k* hits but fewer than *k* of the remaining pieces can reach it. This
  shows up most often in corners and along edges.

It places the largest pieces first because they use up the budget fastest and so rule out wrong
branches soonest. Identical pieces are only tried in one order, so the same layout isn't searched
twice.

### 2 · An endgame table for the widest part of the tree

<img src="docs/endgame.svg" alt="Endgame table replaces the bottom levels of the search tree with one lookup" width="100%">

Small pieces fit in the most places, so the last few levels of the search are where nearly all the
work is. Before searching, the solver lists every way the last few pieces can be placed. It
stores each one in a table, keyed by the board leftover it would fix. When the search gets down
to those pieces, one lookup tells it whether they can finish the board and where they go. This is
a partial *meet-in-the-middle*.

### 3 · Three solvers race

<img src="docs/engines.svg" alt="Exhaustive search, belief-guided search and simulated annealing run in parallel" width="100%">

No single method wins on every board, so three run at once, each in its own background worker:

- **Exhaustive search**: everything above. It's the only one of the three that can also prove a
  board has *no* answer.
- **Belief-guided search**: the board is the sum of where the pieces went, so it can be read
  backwards. *Belief propagation* passes messages between pieces and squares to work out how
  likely each spot is for each piece. The search then tries the most likely layouts first.
  Along the true answer, the right spot is usually its first or second guess. After about 8
  pieces are down, it's nearly always right. This is the one that cracks the big 10×10 and
  11×10 boards.
- **Simulated annealing**: put every piece down at random, then keep moving one piece at a time
  toward fewer wrong squares. Now and then it accepts a worse move so it doesn't get stuck. It
  does best on boards with many answers, where exhaustive search can wander for a long time.

The first answer wins. Before you see it, the answer is replayed on a copy of the board to
confirm it actually clears it.

<img src="docs/benchmark.svg" alt="On 34 real boards: exhaustive 16, annealing 13, both 20, plus belief-guided 27" width="100%">

*Measured on 34 real boards with the native multi-threaded build. The browser page runs the same
three engines compiled to WebAssembly, so on a slower computer it may need more time on the
hardest boards.*

## Under the hood

- **Solver:** about 1,300 lines of C compiled to **WebAssembly** with Emscripten, inlined into the
  page. Nothing to install.
- **Reading the board:** from the HTML you paste. The board, the icon cycle, the pieces and the
  level number all come straight from the page's own markup, not from a screenshot.
- **The bookmark:** a one-click bookmarklet that keeps the game in a full-page frame and talks to
  the solver tab with `postMessage`. It sends the solver only the parsed board, never the page
  itself, so nothing from your session leaves the game tab.
