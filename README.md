# Feaffer

*A tool for the faffing of feathers.*

Feaffer is the first stage of a game — imaginary feathers scattered over a map
of the moors, to be found by walking. But first the feathers themselves had to
be found, and it turns out feather-faffing is its own reward.

The house rule: **if is feather, we should try.** Failure and fudging is always
an option. Feaffer is for fun.

It is one HTML file, no dependencies, no build. It opens on a different feather
every time. A grid reference conjures a specific one; **Wander** rolls a fresh
reference and conjures that. Two drawers of controls — **Form** (the shape of the
thing) and **Colour** (pigment, pattern, and the structural sheen) — faff it from
there, each colour dialled on a built-in **H·S·L picker** made for the muted,
greyed-off tones feathers actually wear.

Three ways to take a feather away: **Specimen** (the vector, as SVG), **Portrait**
(a shareable card as PNG — the feather stood on the sky-grey with its grid
reference at the foot, like a museum tag), and **Record** (the whole state as
plain text). A Record is round-trippable: **Restore** reads one back and
reconjures that exact feather, draw seed and all — so a specimen faffed up while
the kettle boils is never lost to the browser again.

## How the feather is made

Feaffer draws no bitmaps and stamps no textures. A feather is a few hundred thin
strokes on an SVG, each one placed by the same handful of equations a real
feather more or less obeys. What follows is the whole of it, plainly — because
the pleasure of the thing is partly in seeing the grain of the maths through the
finish, and because someone may want to grow their own.

### The shaft — a curve with two ends

The rachis is a single quadratic Bézier from the calamus at the base to the tip:
`B(t) = (1−t)²·base + 2(1−t)t·control + t²·tip`, with the control point pushed
sideways by the **curvature**, which is the wind the feather grew against. At any
`t` along it we can take the tangent, and turn it a right angle for the *normal*
— the across-vane direction the barbs comb out along. Under **asymmetry** the
whole shaft drifts toward the leading edge (by `asym · breadth · 0.22`), the way
a flight feather's does. The visible quill stops at `t = 0.9`; the last tenth is
closed by barbs, not by drawing the shaft to a point.

### The vane — a width, and three ways to close

A single function `width(t)` gives the vane's half-breadth all along the shaft.
Below the widest point (`peak ≈ 0.63`) it eases up as `sin(u·π/2)^0.85`. Above
it, a feather closes in one of **three families** — because a feather's tip is
categorical, not a slider, and the old continuous pointy-to-round blend always
ran two closing mechanisms at once and lived in the lumps between them. Field
guides know three, and each has its own mathematics:

- **Rounded** is a quarter of a **superellipse**, `w = √(1 − uᵖ)`. For any power
  `p ≥ 1` it is convex everywhere, so a dome has nowhere to dent inward — blunt
  or slender, never lumpy. And `p` is measured in *pixels*, not chosen in the
  abstract: it is set so the dome turns over as deep along the shaft as the vane
  is wide across, so a magpie's long tail-feather stays as round at the tip as a
  short body feather. It closes by **wrap** (see the fan, below).
- **Acuminate** is a taper, `w = (1−u)ᵏ`. Raising `k` bends the edge *concave* —
  the sweeping hollow needle of a pheasant's tail. Concavity is honest here; it
  is what *acuminate* means. It closes by taper alone: the vane shortens to
  nothing, and a fine sprig of bare quill carries the very point past where a
  drawable barb gives out.
- **Truncate** is the rounded body, cut. The barbs **shear** off along a growth
  contour — level with the cut at the shaft, drooping gently down each web — and
  the corner where the cut meets the side of the vane is rounded by a *smooth
  minimum* of the two edges, whose radius is the one degree of freedom: crisp
  square to soft-shouldered.

Asymmetry then steals breadth from the leading web and gives it to the trailing
one, and `cos(barb angle)` sets how much of a barb's length is spent reaching
*across* the vane versus *up* it.

### The barbs — combed, and made to land on the edge

The barbs are combed in rows up the shaft (the rounded crown drinks extra
half-step rows so its weave doesn't thin where it turns over). Each barb leaves
the shaft at the **barb angle** and has to be exactly long enough to land its tip
*on* the vane's outline — and here is the one properly subtle sum in the tool. A
barb climbs as it reaches out, so it lands its tip higher up the edge than its
root; reading the width at the root let a barb on a short broad feather climb
clean past the crown at full breadth (a square-shouldered fat head). So the
length is **solved, not read** — a damped fixed-point iteration,
`bl ← ½·bl + ½·width(t + bl·sinθ/L)`, a few turns until the tip sits on the
envelope. Damped, because the envelope's steep tip-end would otherwise make the
bare iteration ring.

Two more honesties keep the vane silky: the tip **jitter** is scaled back by a
barb's slope (`≈ 0.12 · min(1, tan46°/tanθ)`, 46° being the default comb), so a
near-vertical barb's shake
doesn't fling its tip wide of the edge; and on rounded tips a **closing fan**
bends each near-apex barb so it *leaves* the shaft like its neighbours but
*arrives* skimming the crown — its arc's control point placed where those two
directions cross. Because only the rounded family fans, the fan can be served
properly, and the old habit of every strand collapsing onto the axis (re-
sharpening the tip into a central plume with a cleft down each web) dissolves.

### The down — a second, softer weather

The **plumaceous down** at the base is not strokes but *wander*: each strand is a
chain of gentle turns around its own curl, with a whisper of gravity added every
segment, and interior points sprout **barbules** — the micro-wisps that make down
*down* — all pooled into one shared path so a cloud of them costs the SVG almost
nothing. Just above the down, each vane barb keeps a downy base for a short way
(the **afterfeather**), so fluff eases into clean vane instead of hitting a hard
waterline. The down draws from its *own* stream of dice, so faffing it never
re-rolls the vane.

### Wear — the vane unzips

Age is more than missing barbs. A flown feather **unzips**: neighbouring barbs
let go of one another and the edge bites in a V-shaped notch. The notches live on
their own stream too, so a specimen's wear is part of its fixed identity, not a
thing that reshuffles every time you nudge another slider.

### The markings — grown in, not laid on

Nothing is painted on top. A speck, a bar, the terminal band, the eye — each is
made by walking every barb along its own curve and **redrawing the stretches that
fall inside the mark** in the mark's colour. So a marking is built of the same
strokes as the vane and moves with them: shear the vane and its bars shear with
it. Barring is `sin(t·π·n)` for `n` bands, its edges wobbled and mottled by a
per-row hash kept off the main dice (so the bars never jog the wear), and biasable
onto one web. The terminal band follows a **growth contour** — a chevron, level
at the shaft and drooping down each web — because that is how the pigment was laid
as the feather grew, not as a flat waterline. And the eye is its own chapter.

### The sheen — a lens, not a paint

**Iridescence** is a lens, not a pigment: a radial gradient of two hues laid over
the vane with `mix-blend-mode: screen`, so it *lightens toward* its colours like
light on a beetle's back rather than covering them. It is **masked** — softly,
through a blur — to an inset blade of the vane, so its edge feathers away inside
the barbs instead of catching on their ragged tips.

### Colour — hex, worn as H·S·L

Every colour is a hex triple under the bonnet — the thing the drawing actually
reads — and shown to you as **hue, saturation and lightness**, which is the
honest way to hunt a greyed-off dove-grey: saturation on its own axis, lightness
on its own axis, nothing crammed into the corner of a square. The **Record**
writes both, so a clutch of finds that all sit at low saturation *look* like a
cluster on the page.

### The dice — seeded, and kept in order

Randomness is seeded: a hash of the grid reference feeds a **mulberry32**
generator, so a coordinate always grows the same feather. The draw is
deterministic, and *its order is the specimen's identity* — the exact sequence in
which barbs are dropped for wear, jittered, and fluffed **is** the feather. So new
features are always appended at the *end* of the draw order, and every coordinate
conjured before a feature keeps the feather it always had (the peacock's eye
arrives last of all). Three streams are forked off the one seed by XOR
(`⊕ 0x9E3779B9` for the down, `⊕ 0x51633E2D` for the wear-notches), so faffing
the fluff never re-rolls the vane, and adding wear never disturbs the barbs.

### Truths, and 'patameters

The **truths** are the places where the maths is not decoration but the actual
grain of the thing: a real ocellus does hold its rings near `1/φ²` and `1/φ`; a
barb does land its tip on the vane's edge, not its root; a dome is convex
everywhere. The **'patameters** — the pataphysical parameters — are where a real
number is pushed past its reason and something truer-than-true falls out: an eye
grown until it stops being a spot and becomes weather crossing the vane; a
position sent negative until the ocellus stands off the quill; the finding that
*one mark*, sized and cast and coloured right, is the answer to every bird. The
tool is built so both live in the same equations, and you cross from one to the
other by sliding.

## The One Mark

Feaffer does not have a slider for every pattern a feather can wear. It has
bars, a terminal band, specks — and beyond those, exactly one marking: the
**ocellus**, the peacock's eye, grown into the barbs rather than laid on top.
Grown large and sent to stand in the right place wearing the right colours, the
eye turns out to be the answer to all birds. Not perfect — better than that.

The eye is golden, because the real spot's is: its ring boundaries sit at 1/φ²
and 1/φ of the radius, and its ellipse holds its axes in the ratio φ. A phi
eye, which is worth saying to yourself from time to time.

## Field craft

Notes gathered while faffing. All of these are arrangements of the one eye.

- **To 100, the eye is the vane's creature** — sized by the vane's breadth.
  The spot, the ring, the semicircle. **Past 100 it changes allegiance** and
  takes its scale from the feather's whole length, growing until the rings are
  country the feather merely crosses.
- **A russet halo is just a large eye with a vane-coloured pupil.** The pupil
  darkens toward the shade pigment, so a heart matched to the ground sinks
  clean into the vane and leaves bare halo.
- **The partridge:** shuffle the eye off the top of the vane (position 100)
  and a clean semicircle remains at the apex.
- **The snipe:** a peacock with a rust eye slipped nearly completely off the
  end (position 104–108) — only its lower rim still clings to the feather.
- **The jay:** a giant eye **cast** off a web. The ring boundary runs *along*
  the feather, banding the edge.
- **The pheasant: don't cast — drop.** Stand the eye *below* the feather
  (position low, or negative — off the quill), uncast, sized 140–220. Its ring
  arcs cross the vane perpendicular to the shaft as soft-edged transverse
  bands, curved the way a ring should be. Size chooses which ring crosses
  where; position slides the band along the feather.
- **Above 250 is colour-field country:** the rings grow so vast that a whole
  feather fits inside a single band and comes out dipped in one colour, with
  at most one soft boundary crossing it somewhere. Which is also a feather.
- **Roof-birds:** every so often the Wander brings a feather already wearing
  the eye. A peacock landed on Dad's roof eight years ago, so the odds are
  real — but roof-rare. Coordinates conjured before the peacock's arrival
  keep the feathers they always had; the draw order sees to it.

## A feathered object (bird)

The bones of all this are borrowed from a real paper — **L. Streit and W.
Heidrich, *A Biologically-Parameterized Feather Model*** (Imager Computer
Graphics Lab, University of British Columbia). Its idea is that a feather, unlike
hair or fur or scales, has a *definite structure*, and that you can parameterize
that structure — rachis, barbs, barbules, the interpolation between different
ages and forms — and grow a whole range of feathers from it. The rachis-and-vane
frame here, the fanned tip, the reading of a barb as something with its own angle
and length: those are theirs.

The paper is written in careful academic prose, and then, once, it names its
subject *"a feathered object (bird)"* — and the formal language curls up for a
moment into a small feral poetry creature. Feaffer lives in that parenthesis: the
walk from the equation back to the bird, and sometimes a step or two past it.

Feaffer itself — the maths reasoned through, the code written and rewritten, the
failures named and chased down (the square-shouldered fat head, the parted
central plume, the eye that would not settle into the vane) — was faffed out in a
long, cheerful conversation with **Claude**, Anthropic's AI. The naturalist's eye
and the house rules are the faffer's; the fixed-point solves and the golden rings
are the collaboration's; the decision that an eye grown past all reason should
become weather was, as such decisions should be, a joint act of nonsense.

## Provenance

The grey of the page is a Northumberland sky. The barn swallow feathers are
from the big barn archway, this summer.
