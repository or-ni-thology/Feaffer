# The Growing of a Feather

*Numerus pennam facit — the number makes the feather.*

A plain-language map from each slider in Feaffer back to the maths underneath,
and to the paper that started it: **L. Streit & W. Heidrich, *A
Biologically-Parameterized Feather Model* (Eurographics, 2002).** Read it slowly.
You are not meant to catch every line on the first pass — it's built to seep.

The one thing to hold onto: **a feather here is not drawn, it is *grown*.** Every
mark on screen is computed from the numbers on the sliders. Nothing is a picture
of a feather; it is the *rule* for a feather, run.

---

## First, the words

The paper — and this tool — take apart a real feather into a small nested
vocabulary. Worth knowing five words; the rest is decoration.

- **Rachis** — the central shaft. The spine.
- **Calamus** (or quill) — the bare, hollow base that plugs into the skin.
- **Barb** — one of the many side-branches that comb off the rachis.
- **Barbule** — a *branch off a branch*: tiny hooked filaments off each barb that
  zip neighbouring barbs together. **We don't draw these yet.** More below.
- **Vane** (or vexillum) — the flat blade the barbs make between them.

Two more you'll meet: **pennaceous** (the tidy, zipped, flat part of the vane)
and **plumaceous** (the soft, downy, *unzipped* part near the base). And, in
passing: **remige** (a flight feather) and **rectrix** (a tail feather).

Streit & Heidrich's real contribution wasn't just listing these — it was
**parameterising** them so you can slide *smoothly between* feather types and
ages. That sliding-between is the whole reason Feaffer can become a morphospace
for your game. Every slider is one axis of that space.

---

## Part I — the structure *(this is the paper's territory)*

Everything in this part is geometry: where things are, how long, what angle. This
is the part Streit & Heidrich actually model.

### Rachis length — *how tall the feather stands*

The shaft runs from a base point up to a tip point. `length` simply sets how far
the tip is above the base:

```
tipY = baseY − length
```

Longer feather, taller drawing. That's all. It's the plainest slider, and it's
the paper's first parameter too.

### Shaft curvature — *"the wind it grew against"*

The rachis isn't a straight line — it's a **quadratic Bézier curve**. Three
points define it: the base **P₀**, the tip **P₂**, and a single *control point*
**P₁** floating out to the side. The curve is pulled toward that control point:

```
B(t) = (1−t)²·P₀ + 2(1−t)t·P₁ + t²·P₂        for t from 0 (base) to 1 (tip)
```

`curvature` is just how far sideways P₁ sits. Zero, and the shaft is straight.
Push it, and the whole feather bows like a scythe. One number, one bend.

That little `t` — a number walking from 0 at the base to 1 at the tip — is the
hero of this whole document. **Almost everything is measured in `t`.** "Where
along the feather are we?" is always answered by `t`.

### Vane breadth — *the silhouette*

The blade isn't a rectangle; it swells and tapers. A single **envelope function**
decides the half-width of the vane at every height `t`:

```
broadest at t ≈ 0.58 (a little above the middle),
easing up from the base as a sine curve,
easing down to a point at the tip as a cosine curve.
```

`breadth` scales that whole envelope. Small, and you get a slim pin; large, and a
broad paddle. The gentle "shoulder" you see — fuller than a plain triangle — is
those sine/cosine eases with a slight power-curve on them, so the edge is
*rounded*, not angular. This is the paper's "barb-length profile" — the outline is
made by giving each barb the right length for its height.

### Barb angle — *swept-out or swept-back*

Each barb leaves the rachis at an angle. We build it from the shaft's own
direction at that point — its **tangent** (pointing up toward the tip) and its
**normal** (pointing straight out to the side) — and blend between them:

```
barb direction = (sideways · cos angle) + (up-the-shaft · sin angle)
```

At a low angle the barbs stand almost **straight out** to the side. At a high
angle they **sweep sharply toward the tip**, like the teeth of a swept comb. Real
barbs always lean a little toward the tip; the slider lets you say how much.

### Barb density — *how fine the comb*

Simply how many barbs we plant along the rachis. More barbs, finer and more solid
the vane; fewer, and you can see daylight between them. In a real feather this is
the barb *spacing*; here it's a count, which comes to the same thing.

### Vane asymmetry — *the mark of a flight feather*

This is the loveliest true thing in the tool. A flight feather (a remige) is
**lopsided**: the *leading* edge — the one that cuts into the wind — has a
**narrow** vane, and the *trailing* edge has a **broad** one. Two effects, both
tied to the same `asym` number:

```
leading (front) half-width  = base × (1 − 0.62·asym)      ← robbed of width
trailing (back) half-width  = base × (1 + 0.42·asym)      ← given width
and the shaft itself drifts toward the leading edge by 0.22·asym·breadth
```

Slide it up and a plump, symmetrical body-feather visibly becomes a canted,
purposeful flight feather. Slide it to zero and the bird has stopped flying and
gone back to being warm. This is straight from the paper, and it's real biology.

### Plumaceous down — *the soft base*

`down` sets the fraction of the feather, measured up from the base, that is
**downy** rather than tidy. In that lower band the barbs are:

- longer and splayed at a **wider angle** (× 1.5),
- **jittered** — each one nudged by a random wobble,
- drawn **paler, thinner and softer**.

Above the band, the barbs snap into the neat, combed pennaceous vane. This is the
paper's pennaceous/plumaceous split — and, as you'll see at the end, it's *really*
a story about barbules hooking or not hooking. We fake it for now with wobble.

### Age & wear — *fraying at the tip*

Old feathers lose barbs. `wear` is a **probability that any given barb is simply
missing**, and — crucially — that probability *rises toward the tip*:

```
chance a barb is gone ∝ wear × (0.4 + 0.6·t)
```

Because `t` is largest at the tip, the exposed end frays first while the sheltered
base stays whole — exactly how a real feather ages. Turn it up and the specimen
looks like it's had a hard winter on the moor.

---

## Part II — the pigment *(here the paper ends and our own painting begins)*

Be honest about this line: **Streit & Heidrich model structure, not colour.**
Everything below is *our* garnish, painted onto their geometry. It's kept
true-ish to biology, but it's ours, not theirs. That matters for the family motto.

### Ground & shade tones — *two pigments, mixed by depth*

Every barb is coloured by mixing your two swatches. How much of the dark one it
gets depends on where it is:

```
darkness = 0.15 + (1−t)·0.35 + (band darkness)·0.7
```

So barbs are **darker toward the base** (where `1−t` is large) and lighter toward
the tip — the faint gradient you see for free. Real feathers really are often
darker at the base; we lean into it.

### Barring — *bands across the vane*

A wave run up the feather decides where the dark stripes fall:

```
phase = sin(t · π · bars)        → a dark band wherever the wave is near its crest
```

`bars` is how many crests, so how many bands. Biologically this is melanin laid
down in pulses *as the feather grows* — timing becomes pattern. Ours is a clean
sine, which is a polite lie, but a defensible one.

### Terminal band — *a dark tip*

Force the top slice of the feather — the top `tipband` fraction — to be dark,
regardless of the barring. That's the black-dipped tip of a grouse or pheasant
feather. A special case of barring, pinned to the tip.

### Speckling — *spots & splodges (the newest, roughest slider)*

Scatter a number of little ellipses across the vane, each at a random height `t`,
a random side, and a random distance out toward the edge, sized and coloured from
your pigments. `speck` controls how many and how big.

This is the **taster** — and you've already spotted its flaw: a few splodges drift
off the blade, because right now they're placed by *reaching out* from the shaft
without checking they've landed on feather. Your instinct is right and biological:
**spots belong on the feather.** The fix for the second pass is to **clip** them to
the vane's outline (we already compute that outline for the iridescence — we can
reuse it), so no fleck can fall into empty air.

### Iridescence — *a structural sheen (the biggest, happiest cheat)*

Real iridescence isn't pigment at all — it's **light interfering with itself**
inside microscopically-layered barbules (the same physics as oil on a puddle).
We do **not** compute that. Instead we lay a soft, two-colour **radial glow** over
the vane — clipped to its outline, blended in *screen* mode so it *adds light* —
pooled wherever you put "sheen position," with a second tighter glint offset a
little to give that shifting, oily travel. It reads as a sheen; it is honestly a
painted gradient. Named as a cheat so no one is misled.

---

## Part III — the seed, and why nothing flickers

Type a grid reference, and this happens:

1. The text is **hashed** into a single 32-bit number.
2. That number seeds a small, fast, repeatable random generator (**mulberry32**).
3. That generator sets *every slider* — and also fixes the random choices in the
   drawing itself (which barbs are missing, how the down wobbles, where the spots
   land).

So **the same grid reference always grows the identical feather**, on any device,
forever, with nothing stored anywhere. This is the family's deterministic-seeding
standard, the same trick as the eggs.

And one subtlety you can feel with your thumb but might not have named: when you
drag a slider by hand, the *drawing's* random seed is **held fixed**. Only the
geometry updates. That's why faffing feels smooth — the spots and frays don't
**re-shuffle** on every tiny move; they sit still while the shape changes around
them. Deliberate.

---

## Part IV — what's missing, and what's next

### Barbules — the real next rung

Right now the smallest thing we draw is the **barb**. Below it in a real feather
is the **barbule**: each barb carries rows of them, and on the pennaceous vane the
distal barbules end in microscopic **hooklets (hamuli)** that latch onto the next
barb and **zip the whole blade into one sheet**. Where they *don't* hook, you get
down.

Adding barbules would:

- turn "plumaceous vs pennaceous" from a wobble-fake into the **real mechanism**
  (do the barbules hook, or not),
- give the vane a fine woven **texture** instead of a comb of separate lines,
- and add a genuine new **axis** to the morphospace — feather *coherence*.

You can feel the need for them, and you're right to. It's the honest next step
down the hierarchy.

### φ and the Fibonacci question — *an honest answer*

You asked about the golden angle and Fibonacci in feathers. The truthful position,
because the maths must be true:

- The golden angle **φ = 137.5°** genuinely rules **phyllotaxis** — the spiral
  packing of seeds and leaves — and it's the family's hidden constant. It does
  **not**, strictly, set the spacing of barbs along a rachis: real barbs are a
  fairly *even* comb, not a golden spiral. So a "φ barb-spacing" would be a
  beautiful fib, not a fact.
- **Where φ can enter honestly:** as a **deliberate house conceit**, exactly like
  OuPerPo — for instance, dropping speckles by the golden angle so they scatter
  *evenly but never repeat* (which genuinely looks better than pure random), or
  bending the rachis along a logarithmic spiral. That's φ used as a *chosen rule*,
  declared out loud, not a false claim about birds.

So: **a φ button, yes — as an owned constraint, not a smuggled one.** That's the
family way, and it keeps us honest.

### The sliders *are* the morphospace

Here's the payoff for the game. Every slider is one **axis**. A particular feather
is a single **point** in that many-dimensional space. Then:

- a **species** you want to appear reliably is a small **box** — a tight range on
  each axis — that always renders as *recognisably that bird's* feather;
- the **improbable** ones are **wider** boxes, straying past the usual ranges;
- the **impossible** ones push an axis clean **past biology** — a vane with no
  shaft, wear beyond total, that sort of nonsense;
- and Streit & Heidrich's "interpolation between structures and ages" is simply
  **drawing a line between two points and walking along it** — which, on your map,
  becomes *you, walking, to collect a feather.*

Design the boxes, and you've designed the realms. That's the next real
conversation.

---

*A living document. Add to it as the feather-maths grows barbules.*
