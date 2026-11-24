# Design 11: Responsive, Unguided 🎯

Same format as design 10. Each stage tells you **what "done" looks like**. The **Answer key** under each stage is collapsed. Open it only when you're stuck or want to check your work.

**What's new this time:** design 10 was flex plus grid. This one has **two absolutely positioned images overlapping inside a flex column**. That's the thing that breaks, and fixing it means turning px positions into % positions inside a box that has its own shape (you did a version of this on design 12's Part 0).

**Files:**
- HTML: `design-11.html`
- Your original CSS: `css/design-11.css` (leave it alone, it's your backup)

**Test widths for the end:** 1440 · 1000 · 900 · 768 · 600 · 390 · 360

---

## Stage 0: Setup

**Done when:** you have `css/design-11-responsive.css` and the HTML links to it. The page should look the same as before.

<details>
<summary>Answer key</summary>

Duplicate `css/design-11.css` and rename the copy `design-11-responsive.css`. Then in the HTML:

```html
<link rel="stylesheet" href="css/design-11-responsive.css" />
```

</details>

---

## Stage 1: Look first (no code)

Use DevTools → device toolbar (`Ctrl + Shift + M`) and drag slowly from 1440 down to 360.

**Done when:** you've listed every width where something goes wrong, plus your guess at the CSS line causing each one.

**Extra credit:** find the **2 lines in your CSS that do nothing at all**, at any width.

<details>
<summary>Answer key</summary>

| Around | What goes wrong | The cause |
|---|---|---|
| ~800px | The headline wraps to 3 lines; the text column and the images fight for space | the `.hero` flex row never stacks |
| ~600px and below | The staircase image **runs off the right edge**, so the page gets a horizontal scroll | `.media-circle { left: 90px; width: 200px; }`: px values that don't shrink |
| ~600px and below | The arch image **lands on top of the paragraph** | the images are 200px wide no matter how narrow their column gets |
| ~500px | The 3 feature columns get crushed: "Title" and "1" on separate lines, one word per line | `.features` stays a row of 3 |

**Why the images escape:** `.hero-media` has no size of its own. Its width comes from `flex: 1` (half the row), and its height comes from the row. Inside it, the two images are **fixed** at 200px and placed with fixed offsets (`left: 90px`, `right: 10px`). When the column shrinks to ~150px wide, a 200px image placed 90px from the left has to stick out.

**Extra credit, the 2 lines that do nothing:**
- `.hero { height: 50%; }`
- `.features { height: 35%; }`

A % height means "% of my parent's height." But `.page` has no set height (it grows with its content), so 50% of "auto" can't be calculated, and the browser ignores it. You can delete both.

</details>

---

## Stage 2: Fix the root cause (no media query)

**Done when:** the two overlapping images scale down **as one unit**. They keep the same overlap and the same spacing at every width, like one picture shrinking. The desktop at 1440 should look (almost) the same.

Hint, only if you want one: the box the images sit in needs its own shape. After that, every px number inside it becomes a % of that box.

<details>
<summary>Answer key</summary>

**Step 1: give `.hero-media` its own shape.**

```css
.hero-media {
  aspect-ratio: 450 / 290;
  align-self: center;
  /* ...keep the rest of your rule as it is */
}
```

- On desktop, `.page` is capped at `max-width: 1000px`, so `.hero-media` is about **450px wide**: (1000 − 30 − 30 padding − 40 gap) ÷ 2 = 450. On any screen wider than 1000px it stays 450, because the page stops growing. Also, and the two images together need about **290px of height**. So `450 / 290` locks that shape: whatever the width, the height follows.
- `align-self: center` matters. In a flex row, children **stretch** to the row's height by default. A stretched child already has its height set from outside, and when both width and height are set, the browser **ignores aspect-ratio**. `center` turns off the stretch, so aspect-ratio gets to decide the height.

**Step 2: turn every px inside the box into a % of the box.**

```css
.media-circle {
  width: 44%;
  position: absolute;
  top: 0;
  left: 20%;
}
.media-arch {
  width: 44%;
  position: absolute;
  bottom: 3%;
  right: 2%;
  z-index: 1;
}
```

The math (box = 450 wide × 290 tall):

| Old | Calculation | New |
|---|---|---|
| `width: 200px` | 200 ÷ 450 | `44%` |
| `left: 90px` | 90 ÷ 450 | `20%` |
| `bottom: 10px` | 10 ÷ 290 (bottom uses the **height**) | `3%` |
| `right: 10px` | 10 ÷ 450 (right uses the **width**) | `2%` |

- **Delete `height: 200px`** on both images. The `<img>` inside already has `height: auto`, so it gets its real height from its width. A fixed 200px height on the div would fight that.
- Left and right % are measured against the box's **width**. Top and bottom % are measured against its **height**. That's why `bottom` and `right` come out to different % numbers even though both were 10px.

**Check:** drag the width again. The images now shrink together and never leave the page. The headline and the feature columns still need media queries (Stages 3 and 4).

</details>

---

## Stage 3: The stacking breakpoint

**Done when:**
- below your breakpoint, the text sits on top and the image pair sits **centered underneath** at a sensible size (not stretched across a 768px screen)
- there's no white strip above the light-blue page on tablet

<details>
<summary>Answer key</summary>

**Where:** about **820px**. That's where the text column gets narrow enough that "New inspirations" is about to wrap. Anything from 780 to 860 is correct.

```css
/* ============== TABLET: 820px ============== */
@media (max-width: 820px) {
  .page {
    margin: 0 auto;
  }
  .hero {
    flex-direction: column;
    gap: 48px;
  }
  .hero-copy,
  .hero-media {
    width: 100%;
  }
  .hero-media {
    max-width: 480px;
  }
}
```

- `flex-direction: column` stacks the copy and the images.
- `width: 100%` overrides your desktop `calc(50% - 40px)`, which was made for side by side.
- `max-width: 480px` on the images: full width at 768 would make the image pair huge. This caps them near their desktop size.
- **Why are they centered?** You didn't write any centering. `align-self: center` from Stage 2 is still there, and in a **column** the cross axis is horizontal, so "center" now means centered left to right. That's the axis flip from your flexbox drills.
- `.page { margin: 0 auto; }` removes the 40px top margin, so there's no white strip on tablets and phones.

</details>

---

## Stage 4: Phone polish

**Done when** (check at 390 and 360):
- the 3 features stack **one per row** inside the dark card, each full width
- the headline is 2 lines, not 4
- the side padding is the phone standard
- there's no horizontal scroll

<details>
<summary>Answer key</summary>

```css
/* ============== PHONE: 600px ============== */
@media (max-width: 600px) {
  .page {
    padding: 40px 16px;
    gap: 24px;
  }
  .hero-title {
    font-size: 36px;
  }
  .hero-text {
    font-size: 15px;
  }
  .features {
    flex-direction: column;
    align-items: stretch;
    gap: 28px;
    margin: 0;
    padding: 32px 24px;
  }
  .feature {
    width: 100%;
  }
}
```

- `.page padding: 40px 16px`: 16px sides is the phone standard again.
- `.hero-title 36px`: "New designs" / "New inspirations" fits in 2 lines at 360.
- `.features flex-direction: column` stacks the 3 features. `align-items: stretch` makes each one full width. Your desktop has `align-items: center`, which in a column would shrink them and center them.
- `.feature width: 100%` overrides `calc(33% - 40px)`, which was made for 3 side by side.
- `.features margin: 0`: the desktop `margin: 40px auto` adds a 40px gap on top of the `.page` gap. That's too much space on a phone.

</details>

---

## Stage 5: Final test

**Done when**, at **1440 · 1000 · 900 · 768 · 600 · 390 · 360**:
- there's no horizontal scroll
- the images never cover the text
- the images always overlap the same way (one shrinking picture)
- the features read cleanly at every width

<details>
<summary>Answer key</summary>

| Width | What you should see |
|---|---|
| 1440 / 1000 / 900 | the original two-column hero, 3 features in a row |
| 820 and below | stacked: text on top, image pair centered underneath (max 480px), 3 features still in a row |
| 600 and below | 2-line headline, 16px sides, features stacked one per row |

**2 base-CSS fixes + 2 media queries (820, 600). Same shape as design 10.** You're starting to see the pattern: fix the fixed sizes first, then stack, then polish for phone.

</details>

---

## 🏆 Bonus: images first on phone (optional)

On a phone, the visitor sees a wall of text before any picture. Try making the image pair show up **above** the headline on phones only, **without touching the HTML**.

Hint: you used this property in design 9.

<details>
<summary>Answer key</summary>

In your 600 query:

```css
.hero-media {
  order: -1;
}
```

Every flex child starts at `order: 0`. `-1` means "go before everyone at 0," so the images jump above the copy. The HTML doesn't change, only where the item is shown. Screen readers still read it in HTML order (headline first), which is what you want.

Designer's call: some pages want the picture first to grab attention, others want the headline first to explain. Decide which one this page needs and say why.

</details>

---

When you're done, tell me and I'll render your version at all 7 widths and review it.
