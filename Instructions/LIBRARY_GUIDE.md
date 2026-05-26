# TikZ AI Plans Library — User Guide

A TikZ library for drawing AI partial-order planning diagrams: action nodes with
precondition/effect label lines, init/goal bars, and causal links.

You will need the following files:

- `tikzlibraryaiplans.code.tex` (library code)
- `main.tex` (the main document)
- `domain.tex` (for defining the action schemas)
- `plan.tex` (the plan that uses the defined actions) 

---

## Loading the library

**When the library file is in your project folder:**
```latex
\usetikzlibrary{aiplans}
```

**When the library file is in a different folder (e.g. two levels up):**
```latex
\input{../../LibraryCode/tikzlibraryaiplans.code.tex}
```

Also add the required TikZ shape libraries for your document if needed:
```latex
\usepackage{tikz}
\usetikzlibrary{positioning, shapes.symbols, arrows.meta}
```

---

## Overview of node types

| Style | Description |
|---|---|
| `action={precs}{effs}{name}` | Rounded rectangle action node |
| `init={labels}` | Vertical black bar — horizontal layout init |
| `goal={labels}` | Vertical black bar — horizontal layout goal |
| `init vertical={labels}` | Horizontal black bar — vertical layout init |
| `goal vertical={labels}` | Horizontal black bar — vertical layout goal |

---

## 1. Action nodes

### Basic syntax

```latex
\node[action={prec1,prec2}{eff1,eff2}{Action Name},minimum height=1cm] (a) {};
```

- **Argument 1** — comma-separated precondition labels (empty space `  ` for none)
- **Argument 2** — comma-separated effect labels (empty space `  ` for none)
- **Argument 3** — the action name shown inside the node
![Basic syntax](pic\basic-syntax.png)


### Named anchors

After a node is placed, the library creates named coordinates for every label line tip:

- `(nodename-pre-1)`, `(nodename-pre-2)`, … — precondition line tips
- `(nodename-eff-1)`, `(nodename-eff-2)`, … — effect line tips

These are the attachment points for causal links.

### Default appearance

- Rounded corners, cyan fill, `minimum width=3cm`
- Label lines auto-sized to fit their text (widest label + 0.3 cm padding)
- Labels sit at the **midpoint** of each line, above it (horizontal) or to the right (vertical)

### Overriding node size

Pass standard TikZ keys alongside the style:

```latex
\node[action={p}{q}{A}, minimum width=4cm, minimum height=2cm, fill=blue!30] (a) {};
```
![Overriding node size demo 1](pic\overriding-nodesize-1.png)

```latex
 \node[action={p1,p2}{q1,q2}{B}, minimum width=4cm, minimum height=2cm, fill={rgb,255:red,0; green,76; blue,0}, prec label lengths={2cm,1.5cm}, eff label lengths={1cm,2cm}] (a) {};
```
![Overriding node size demo 2](pic\overriding-nodesize-2.png)

---

## 2. Layout: horizontal vs vertical

### Horizontal (default)

Precondition lines extend **left** from the west side; effect lines extend **right** from the east side.

```latex
\node[action={p}{r}{MyAction}, minimum height=0.5cm] (a) {};
```
![Horizontal layout](pic\layout-horizontal.png)

### Vertical

Precondition lines extend **upward** from the top; effect lines extend **downward** from the bottom.

Add the `vertical` key anywhere in the node options:

```latex
\node[action={p}{r}{MyAction}, vertical, minimum height=0.5cm] (a) {};
```
![Vertical layout](pic\layout-vertical.png)

You can bake it into a shared style (also you could baked into shared style for horizontal style see Section 9 example):

```latex
\tikzset{
    VSTYLE/.style n args={3}{
        action={#1}{#2}{#3},
        vertical, minimum width=2cm, minimum height=1cm
    }
}
\node[VSTYLE={p}{r}{MyAction1}] (a) {};
\node[VSTYLE={e}{f}{MyAction2}] (b) [right=0.5cm of a] {};
```
![Vertical layout with share style](pic\layout-vertical-double.png)

---

## 3. Init and goal nodes

### Horizontal layout — `init` and `goal`

Tall thin vertical bars. Effect lines go **right** from init; precondition lines go **left** from goal.

```latex
\node[init={p,q,r}] (init) {};
\node[goal={$\neg$ p,$\neg$ q,$\neg$ r}] (goal) [right=10cm of init] {};
```
![Horizontal layout of init and goal](pic\horizontal-init-goal.png)

Default size: `minimum height=8cm`. Override freely:

```latex
\node[init={p,q,r}, minimum height=4cm] (init) {};
\node[goal={$\neg$ p,$\neg$ q,$\neg$ r},minimum height=4cm] (goal) [right=10cm of init] {};
```
![Horizontal layout of init and gaol overriding demo](pic\horizontal-init-goal-override.png)

### Vertical layout — `init vertical` and `goal vertical`

Wide thin horizontal bars. Effect lines go **downward** from init; precondition lines go **upward** from goal. 
Default size: `minimum width=3cm`.

```latex
\node[init vertical={water,tea bag,milk}, minimum width=5cm] (init) {};
\node[goal vertical={tea ready}, minimum width=5cm] (goal) [below=8cm of init] {};
```
![Vertical layout of init and goal](pic\vertical-init-goal.png)


---

## 4. Causal links

### Basic causal link

```latex
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm] (a) {};
\node[action={b}{c}{B}, minimum width=1cm, minimum height=1cm] (b) [right=2 cm of a]{};
\draw[causalLink] (a-eff-1) to (b-pre-1);
```
Draws an arrow with filled-circle endpoint marks at both ends, shortened slightly from each anchor.

![Causallink](pic\causalLink.png)


### Causal link with a mid-path label

```latex
\draw[causalLink with label={cat}] (a-eff-1) to (b-pre-1);
```

Places a small label above the midpoint of the link.

![Causallink middle label](pic\causalLink-label.png)

### Bending and routing

Standard TikZ path options work normally:

```latex
\node[action={a}{b1,b2,b3}{A}, minimum width=1cm, minimum height=1cm] (a) {};
\node[action={b1,b2,b3}{c}{B}, minimum width=1cm, minimum height=1cm] (b) [right=2 cm of a]{};
\draw[causalLink, bend left] (a-eff-1) to (b-pre-1);
\draw[causalLink] (a-eff-2) to (b-pre-2);
\draw[causalLink, bend right] (a-eff-3) to (b-pre-3);
```
![Causallink bend](pic\causalLink-bend.png)


You can also connect directly to a whole node rather than a specific anchor:

```latex
\draw[causalLink, bend right, causal mark=-] (a.south) to (b.south);
```
![Causallink node removal](pic\causalLink-bend-node.png)

---

## 5. Causal link endpoint marks

By default every causal link draws a **filled circle** at both ends.
Override with `causal mark=X` placed **after** `causalLink` or `causalLink with label`:

| Key | Mark |
|---|---|
| *(default / omitted)* | Filled circle |
| `causal mark=*` | Filled 5-pointed star |
| `causal mark=@` | Hollow circle |
| `causal mark=^` | Filled triangle (direction-aware) |
| `causal mark={&}` | Filled square (braces required around `&`) |
| `causal mark=-` | No mark (plain arrow only) |

```latex
\node[action={a}{b1,b2,b3,b4,b5,b6}{A}, minimum width=1cm, minimum height=2cm] (a) {};
\node[action={b1,b2,b3,b4,b5,b6}{c}{B}, minimum width=1cm, minimum height=2cm] (b) [right=2 cm of a]{};

\draw[causalLink, bend left] (a-eff-1) to (b-pre-1);
\draw[causalLink, bend left, causal mark=*] (a-eff-2) to (b-pre-2);
\draw[causalLink, bend left, causal mark=@] (a-eff-3) to (b-pre-3);
\draw[causalLink, bend left, causal mark=^] (a-eff-4) to (b-pre-4);
\draw[causalLink, bend left, causal mark={&}] (a-eff-5) to (b-pre-5);
\draw[causalLink, bend left, causal mark=-] (a-eff-6) to (b-pre-6);
```
![Causallink marks](pic\causalLink-mark.png)

The triangle mark (`^`) automatically rotates to point along the link direction in both horizontal and vertical diagrams.

![Vertical Causallink marks](pic\causalLink-mark-vertical.png)

The `shorten <=<dim>, shorten >=<dim>` could used for adjust gap of causual link with start node and end node. `<=` for the start node gap, and `=>` for the end node gap:
```latex
\node[init={,,,}, minimum height=4cm] (init) {};
\node[goal={,,,},minimum height=4cm] (goal) [right=5cm of init] {};

\draw[causalLink, bend left] (init-eff-1)  to (goal-pre-1);
\draw[causalLink, bend left, shorten <=20pt, shorten >=20pt] (init-eff-2)  to (goal-pre-2);
\draw[causalLink, bend left, shorten <=20pt] (init-eff-3)  to (goal-pre-3);
\draw[causalLink, bend left, shorten >=20pt] (init-eff-4)  to (goal-pre-4);
```

![CausalLink different length](pic\casualLink-length.png)

---

## 6. Label position — `label side`

By default labels sit at the **midpoint** of their line.

Add `label side` to move labels to the **tip** of the line (the far end from the node):


- Horizontal prec labels → placed at the left tip, right-aligned
- Horizontal eff labels → placed at the right tip, left-aligned
- Vertical labels → placed at the top/bottom tip with direction-aware anchoring

```latex
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm,label side] (a) {};
```
![Label text side](pic\labelside.png)


`label side` could set inside a shared style so it applies to every node at once:

```latex
\tikzset{
    STYLE/.style n args={3}{
        action={#1}{#2}{#3},
        minimum width=2cm,
        label side
    }
}
```

---

## 7. Vertical label orientation

These keys only affect **vertical-mode** label nodes. They control how label text is rotated/positioned relative to the line.

| Key | Orientation |
|---|---|
| *(default)* | Horizontal text, to the right of the line midpoint |
| `vertical text` | Rotated −90°, text reads top-to-bottom along the right of the line |
| `vertical text right` | Same as `vertical text` |
| `vertical text left` | Rotated +90°, text reads bottom-to-top along the left of the line |
| `vertical text top` | Horizontal text, placed at the line end (above prec tips / below eff tips) |

```latex
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm, vertical] (a) {};
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm, vertical,vertical text] (b) [right=0.5cm of a] {};
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm, vertical,vertical text left] (c) [right=0.5cm of b] {};
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm, vertical,vertical text top, vertical text top shift=8pt] (d) [right=0.5cm of c] {};
```
![Vertical graph vertical text display](pic\verticaltext.png)

Combine with shared style to place the text at any of these orientations:

```latex
\tikzset{
    VSTYLE/.style n args={3}{
        action={#1}{#2}{#3},
        vertical, minimum width=2cm,
        vertical text left    
    }
}
```

### Adjusting the `vertical text top` gap

When using `vertical text top`, the label is placed a small distance from the line end. The default shift is `3pt`.
Control that gap with:

```latex
vertical text top shift=<dim>   
```

Example — decrease the gap to 2pt combine with 0.1cm label length on pre and eff:

```latex
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm, vertical,vertical text top, vertical text top shift=2pt, prec label length=0.1cm, eff label length=0.1cm] (a) {};
```

![Vertical graph text top adjustion](pic\verticaltext-modify.png)

Or set it globally for all nodes in the picture:

```latex
\begin{tikzpicture}[vertical text top, vertical text top shift=6pt]
    ...
\end{tikzpicture}
```

---

## 8. Label line length

### Automatic sizing (default)

The library measures the widest label text among all prec (or eff) labels for a node and sets all lines of that group to the same length (widest text width + 0.3 cm).

### Override all prec lines to a fixed length

```latex
prec label length=<dim>     % applies to action precs AND goal precs
eff label length=<dim>      % applies to action effs  AND init effs
```

For example:
```latex
\node[action={a}{b}{A}, minimum width=1cm, minimum height=1cm, prec label length=1cm, eff label length=0.5cm] (a) {};
```

![Label length adjust](pic\labellength.png)


For `init` / `goal` nodes the unified keys apply too:
- `init` and `init vertical` → use `eff label length`
- `goal` and `goal vertical` → use `prec label length`

### Override individual lines (per-line lengths)

Supply a comma-separated list; leave entries empty to keep auto-sizing for those positions:

```latex
prec label lengths={<dim1>,<dim2>,...}
eff label lengths={<dim1>,<dim2>,...}
```

Examples:

```latex
\node[init={p,q,r}, minimum height=4cm, eff label lengths ={,0.3cm,0.5cm}] (init) {};
\node[goal={$\neg$ p,$\neg$ q,$\neg$ r},minimum height=4cm, prec label lengths ={,1cm,0.5cm}] (goal) [right=5cm of init] {};
```

![Individual label length adjust](pic\labellength-individual.png)

---

## 9. Complete style example — horizontal diagram
**More examples in Examples folder**
```latex
% Define commonly used style for action
\tikzset{
    STYLE/.style n args = {3}{
        action={#1}{#2}{#3},
        minimum height=2cm, 
        minimum width=2.5cm
    }
}

% Define actions
\tikzset{
    % init and goal:
    INIT/.style = {
        init={gF,onT(B),onT(C),onT(D),clr(A),clr(B),clr(D),{on(A,C)}}, 
        minimum height=8cm
        },
    GOAL/.style = {
        goal={{on(A,B)},{on(D,C)}}, 
        minimum height=8cm
        },
    % normal actions:
    PICKUP/.style n args  ={1}{
        STYLE={gF,onT(#1),clr(#1)}{$\neg$gF,gH(#1),$\neg$clr(#1),$\neg$onT(#1)}{pickup(#1)}
    },
    PUTDOWN/.style n args ={1}{
        STYLE={gH(#1)}{gF,$\neg$gH(#1),onT(#1),clr(#1)}{putdown(#1)}
        },
    STACK/.style n args   ={2}{
        STYLE={gH(#1),clr(#2)}{gF,$\neg$gH(#1),onT{(#1,#2)},clr(#1),$\neg$clr(#2)}{stack{(#1,#2)}}
        },
    UNSTACK/.style n args ={2}{
        STYLE={gF,clr(#2),on(#1)}{$\neg$gF,gH{(#1)},$\neg$on(#1),$\neg$clr(#1),clr(#1)}{unstack{(#1,#2)}}
        }
}

\node[INIT]           (init) {};
\node[UNSTACK={A}{C}] (unstackAC) [right=3.5cm of init]      {};
\node[PUTDOWN={A}]    (putdownA)  [right=3.5cm of unstackAC] {};
\node[PICKUP={A}]     (pickupA)   [right=3.5cm of putdownA]  {};
\node[STACK={A}{B}]   (stackAB)   [right=3.5cm of pickupA]   {};
\node[PICKUP={D}]     (pickupD)   [right=3.5cm of stackAB]   {};
\node[STACK={D}{C}]   (stackDC)   [right=3.5cm of pickupD]   {};
\node[GOAL]           (goal)      [right=3.5cm of stackDC]   {};

\draw [causalLink, bend left]     (init-eff-1)      to (unstackAC-pre-1);
\draw [causalLink, bend left]     (init-eff-5)      to (unstackAC-pre-2);
\draw [causalLink, bend right]    (init-eff-8)      to (unstackAC-pre-3);
\draw [causalLink, bend left]     (unstackAC-eff-2) to (putdownA-pre-1);
\draw [causalLink, bend left]     (putdownA-eff-1)  to (pickupA-pre-1);
\draw [causalLink, bend right]    (putdownA-eff-3)  to (pickupA-pre-2);
\draw [causalLink, bend right]    (putdownA-eff-4)  to (pickupA-pre-3);
\draw [causalLink, bend left]     (pickupA-eff-2)   to (stackAB-pre-1);
\draw [causalLink, bend right]    (init-eff-6)      to (stackAB-pre-2);
\draw [causalLink, bend left]     (stackAB-eff-1)   to (pickupD-pre-1);
\draw [causalLink, bend right=50]     (init-eff-4)      to (pickupD-pre-2);
\draw [causalLink, bend right=20] (init-eff-7)      to (pickupD-pre-3);
\draw [causalLink, bend left]     (pickupD-eff-2)   to (stackDC-pre-1);
\draw [causalLink, bend right=20] (unstackAC-eff-5) to (stackDC-pre-2);
\draw [causalLink, bend left=50]     (stackAB-eff-3)   to (goal-pre-1);
\draw [causalLink, bend left]     (stackDC-eff-3)   to (goal-pre-2);
```

![Blocksworld 1 example](pic\blocksworld1.png)

---

## 10. Complete style example — vertical diagram
**More examples in Examples folder**

```latex
\tikzset{
    VSTYLE/.style n args={3}{
        action={#1}{#2}{#3},
        vertical, minimum width=3.5cm, minimum height=1cm, 
        vertical text top,
        vertical text top shift=0.25cm,
        prec label length={0.5cm},
        eff label length={0.5cm},
    },
    VINIT/.style={
        init vertical={water,tea bag,milk},
        minimum width=5cm, minimum height=0.1cm, 
        vertical text top,
        vertical text top shift=0.25cm,
        eff label length={0.5cm}
    },
    VGOAL/.style={
        goal vertical={tea ready},
        minimum width=5cm, minimum height=0.03cm, 
        vertical text top,
        vertical text top shift=0.2cm,
        prec label length={0.5cm}
    },
    HEAT/.style={
        VSTYLE={water}{hot water}{heat water}
        },
    STEEP/.style={
        VSTYLE={{hot water},{tea bag}}{steeped tea}{steep tea}
        },
    POUR/.style={
        VSTYLE={{steeped tea},milk}{tea ready}{pour milk}
        }
}

\node[VINIT]  (init)  {};
\node[HEAT]   (heat)  [below=4cm of init]  {};
\node[STEEP]  (steep) [below=4cm of heat]  {};
\node[POUR]   (pour)  [below=4cm of steep] {};
\node[VGOAL]  (goal)  [below=4cm of pour]  {};

\draw[causalLink, bend left, shorten <=10pt, shorten >=10pt] (init-eff-1)  to (heat-pre-1);
\draw[causalLink,bend left=70, shorten <=20pt, shorten >=20pt] (init-eff-2)  to (steep-pre-2);
\draw[causalLink,bend left=60, shorten <=10pt, shorten >=10pt] (init-eff-3)  to (pour-pre-2);
\draw[causalLink,bend left, shorten <=10pt, shorten >=10pt] (heat-eff-1)  to (steep-pre-1);
\draw[causalLink,bend left, shorten <=10pt, shorten >=10pt] (steep-eff-1) to (pour-pre-1);
\draw[causalLink,bend left, shorten <=10pt, shorten >=10pt] (pour-eff-1)  to (goal-pre-1);
```

![Teamake vertical example](pic\vertical-teamake-example.png)

---

## 11. Key reference summary

### Node styles

| Style | Arguments | Description |
|---|---|---|
| `action` | `{precs}{effs}{name}` | Horizontal or vertical action node |
| `init` | `{labels}` | Vertical bar, effects go right |
| `goal` | `{labels}` | Vertical bar, precs go left |
| `init vertical` | `{labels}` | Horizontal bar, effects go down |
| `goal vertical` | `{labels}` | Horizontal bar, precs go up |

### Layout and label keys

| Key | Effect |
|---|---|
| `horizontal` | Precs left, effs right *(default)* |
| `vertical` | Precs up, effs down |
| `label side` | Labels at line tips instead of midpoints |
| `vertical text` / `vertical text right` | Labels rotated −90° (top-to-bottom) |
| `vertical text left` | Labels rotated +90° (bottom-to-top) |
| `vertical text top` | Labels horizontal at line ends |
| `vertical text top shift=<dim>` | Gap between label and line end (default `3pt`) |

### Line length keys

| Key | Effect |
|---|---|
| `prec label length=<dim>` | Fixed length for all prec lines (action + goal) |
| `eff label length=<dim>` | Fixed length for all eff lines (action + init) |
| `prec label lengths={d1,d2,...}` | Per-line lengths for prec lines; empty entry = auto |
| `eff label lengths={d1,d2,...}` | Per-line lengths for eff lines; empty entry = auto |

### Causal link keys

| Key | Effect |
|---|---|
| `causalLink` | Arrow with endpoint marks |
| `causalLink with label={text}` | Arrow with mid-path label |
| `causal mark=-` | No endpoint mark |
| `causal mark=@` | Hollow circle mark |
| `causal mark=*` | Star mark |
| `causal mark=^` | Triangle mark |
| `causal mark={&}` | Square mark |

### Named anchor coordinates

| Coordinate | Where |
|---|---|
| `(nodename-pre-N)` | Tip of the N-th precondition line |
| `(nodename-eff-N)` | Tip of the N-th effect line |

