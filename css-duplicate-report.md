# CSS Duplicate Audit — NOWTOBOOK

**Files checked:** `css/variables.css`, `css/base.css`, `css/components.css`, `css/home.css`, `css/results.css`, `css/booking.css`
**Method:** Full CSS parse (820 rules, 3138 declarations flattened out of `@media` blocks), then selector-level, block-level and declaration-level comparison.

---

## TL;DR — Seedha jawab

| Sawal | Jawab |
|---|---|
| Kitni CSS **exact copy-paste** duplicate hai cross-file? | **0 rules** |
| Kitne **selectors** 2+ files mein repeat hote hain? | **1** — sirf `:root` |
| Same selector + same media context dobara define? | **0** |
| Same **declaration body** alag selectors ke under? | **32 groups** (18 cross-file) = **127 redundant declarations** |
| Kitne `property:value` pairs 2+ files mein share hote hain? | **221** |
| Total declarations jo pehle se use ho chuke pair ka repeat hain? | **2208 / 3138 = 70%** |

**Bottom line:** Aapki 6 files mein **koi bhi rule dusri file se copy-paste nahi hua**. Jo 144 "duplicate selectors" dikhte hain wo sab **responsive `@media` overrides** hain — ye duplication nahi, ye sahi tarika hai.

**Asli duplication selector-level par nahi, declaration/pattern level par hai** — 6 files ne apna apna "card", "flex-row", "truncate" pattern dobara likh liya hai. Aur saath mein design-system ka leakage hai (19 unused variables, 34 hardcoded z-index).

---

## 1. File-by-file size

| File | Rules | Declarations |
|---|---:|---:|
| `base.css` | 23 | 97 |
| `variables.css` | 1 | 88 |
| `components.css` | 177 | 663 |
| `booking.css` | 140 | 485 |
| `results.css` | 185 | 718 |
| `home.css` | 294 | 1087 |
| **TOTAL** | **820** | **3138** |

Unique selectors: **659** · Distinct `property:value` pairs: **930**

---

## 2. Cross-file duplication: what actually exists

### 2.1 Exact duplicate rules — **ZERO**

Koi bhi `selector { ... }` block do files mein identical nahi hai. `@import` order `style.css` mein sahi hai (variables → base → components → home → results → booking), isliye cascade bhi saaf hai.

### 2.2 Sirf **EK** selector 2+ files mein hai: `:root`

```
variables.css:1     :root { --section-py: 80px; ... }
home.css:1733       @media (max-width: 767px) { :root { --section-py: 56px; } }
```

Ye **intentional override** hai, lekin **galat jagah** rakha hai:
- Ek design token ko page-level file (`home.css`) mein override kiya gaya hai.
- `--section-py` **kisi bhi file mein use hi nahi hota** (dead token) — to ye override bhi bekaar hai.
- Agar baad mein `results.css` / `booking.css` bhi `--section-py` use karein, to wo silently galat value uthayenge.

**Fix:** override ko `variables.css` ke andar usi `@media` block mein move karo, ya token delete kar do.

### 2.3 144 repeated selectors — **ye duplication NAHI hai**

Har repeated selector ek **alag `@media` context** mein hai. Example:

```
components.css:263   .ntb-search-row            (base)
components.css:804   .ntb-search-row            @media (min-width: 1024px)
components.css:1060  .ntb-search-row            @media (max-width: 991px)
components.css:1128  .ntb-search-row            @media (max-width: 767px)
```

Ye mobile-first responsive design ka normal aur correct pattern hai. **Inhe merge karne ki zaroorat nahi.**

> ⚠️ Ek chhoti si baat: `components.css` mein **chaar** breakpoint bands hain — `min-width:1024`, `max-width:991`, `max-width:767`, plus `max-width:400`. `min-width` aur `max-width` mix hone se overlapping ranges ban sakti hain. Ye check karne layak hai (neeche section 5 dekho).

---

## 3. Asli duplication: declaration-level (yahan paisa pada hai)

### 3.1 32 identical declaration bodies, alag selectors ke under

**18 groups cross-file hain.** Agar inhe shared utility class mein nikaal dein to **127 redundant declarations** bach jaati hain.

Sabse bada group — **7 rules, 3 files, bilkul same body**:

```css
background: var(--orange);
border-color: var(--orange);
color: var(--white);
```

```
components.css:682   .ntb-pax-counter button:last-child
components.css:731   .ntb-pax-classes button:hover, .ntb-pax-classes button.active
components.css:936   .ntb-social:hover
results.css:79       .ntb-panel-close-btn:hover
results.css:332      .ntb-filter-drawer-close:hover
results.css:1151     .ntb-modify-panel .ntb-panel-close-btn:hover   @media(max-width:900px)
home.css:205         #article-pagination .page-item[class~="active"] .page-link
```

Baaki notable cross-file groups:

| Body | Selectors | Files |
|---|---|---|
| `height:100%; object-fit:contain; width:100%` | `.ntb-provider-logo img`, `.ntb-segment-logo img`, `.ntb-resultcard-logo img`, `.ntb-route-airline img` | booking ×2, results, home |
| `align-items:center; display:flex; flex-wrap:wrap; gap:var(--space-2)` | `.ntb-provider-name-row`, `.ntb-summary-chips` | booking, results |
| `background:white; border:1px solid var(--border); border-radius:lg; box-shadow:sm; overflow:hidden` | `.ntb-booking-segment`, `.ntb-sitemap-group` | booking, home |
| `background:var(--orange-light); color:var(--orange)` | `.ntb-calendar-nav:hover`, `.ntb-calendar-day:hover`, `.ntb-provider-toggle:hover`, `.ntb-step-icon-orange` | components, booking, home |
| `background:var(--navy); border-color:var(--navy); color:var(--white)` | `.ntb-swap-btn:hover`, `.ntb-sort-btn.active`, `.ntb-airline-chip:hover` | components, home |
| `color:var(--text-muted); font-size:var(--fs-sm)` | `.ntb-segment-dot`, `.ntb-segment-route i`, `.ntb-addon-price` | booking ×3 |
| `overflow:hidden; text-overflow:ellipsis; white-space:nowrap` | `.ntb-airport-selected-name`, `.ntb-article-title` | components, home |
| `color:var(--navy); font-size:var(--fs-md); font-weight:var(--fw-bold)` | `.ntb-resultcard-time small b`, `.ntb-reviewer-name` | results, home |
| `height:32px; width:32px` | `.ntb-segment-logo`, `.ntb-resultcard-leg-logo img` | booking, results |

> **Note:** `results.css` ka filter panel apne aap ko repeat kar raha hai — `[data-filter="stops"]` aur `[data-filter="airlines"]` ke liye `label`, `input[type=checkbox]`, `::-webkit-scrollbar-track`, `::-webkit-scrollbar-thumb` — sab **100% identical** blocks. Ye 4 groups sirf ek attribute selector se collapse ho sakte hain (`[data-filter="stops"], [data-filter="airlines"]`).

### 3.2 Sabse zyada repeat hone wali declarations

| Count | Declaration | Files |
|---:|---|---|
| 120× | `display: flex` | components 20, booking 26, results 32, home 42 |
| 103× | `align-items: center` | base 3, components 20, booking 18, results 29, home 33 |
| 74× | `color: var(--navy)` | base 5, components 12, booking 18, results 21, home 18 |
| 55× | `color: var(--text-muted)` | base 1, components 9, booking 13, results 17, home 15 |
| 50× | `font-weight: var(--fw-bold)` | base 6, components 6, booking 11, results 13, home 14 |
| 47× | `background: var(--white)` | base 2, components 7, booking 7, results 13, home 18 |
| 43× | `color: var(--white)` | base 3, components 12, results 11, home 17 |
| 42× | `cursor: pointer` | base 3, components 15, booking 5, results 15, home 4 |
| 40× | `font-size: var(--fs-sm)` | base 1, components 2, booking 15, results 14, home 8 |
| 40× | `justify-content: center` | base 3, components 8, booking 3, results 10, home 16 |
| 40× | `font-size: var(--fs-md)` | components 8, booking 8, results 13, home 11 |

### 3.3 Repeating design patterns (utility class candidates)

| Pattern | Rules | Distribution |
|---|---:|---|
| flex + align-items:center | **78** | components 14, booking 13, results 23, home 28 |
| flex + flex-direction:column | **30** | components 2, booking 9, results 10, home 9 |
| position:absolute | **28** | components 8, booking 1, results 3, home 16 |
| flex + align:center + justify:center | **24** | components 3, booking 2, results 7, home 12 |
| orange hover (`bg:orange; color:white`) | **16** | base 1, components 5, results 6, home 4 |
| card (`white bg + border + radius-lg`) | **14** | components 1, booking 3, results 3, home 7 |
| truncate (`overflow/ellipsis/nowrap`) | **4** | components 3, home 1 |

### 3.4 Per-file pair overlap (shared `prop:value` pairs)

| File pair | Shared pairs | Jaccard |
|---|---:|---:|
| `results.css` ↔ `home.css` | 121 | 21% |
| `components.css` ↔ `home.css` | 119 | 19% |
| `booking.css` ↔ `home.css` | 105 | 20% |
| `components.css` ↔ `results.css` | 100 | 21% |
| `booking.css` ↔ `results.css` | 100 | **28%** |
| `components.css` ↔ `booking.css` | 88 | 21% |

`booking.css` ↔ `results.css` sabse zyada overlap rakhte hain (28%) — in dono mein sabse zyada shared component patterns hain.

---

## 4. Design-system leakage (duplication ka asli root cause)

### 4.1 19 out of 88 variables **kabhi use hi nahi hue**

```
--bg-card, --container-px, --emirates-color,
--lh-tight, --lh-snug, --lh-normal, --lh-relaxed,
--section-py, --shadow-hover, --shadow-xl, --space-20, --warning,
--z-base, --z-dropdown, --z-sticky, --z-header, --z-panel, --z-modal, --z-toast
```

🔴 **Sabse bada issue: saare 7 `--z-*` tokens unused hain** — aur CSS mein **35 hardcoded `z-index` declarations** hain (components 9, home 13, results 13):

```
components.css  1000, 200, 20, 4, 50, 70, 60, 10, 1100
results.css      999, 998, 5, 1, 1, 2, 40, 1200, 2, 1200, 5, 1150, 1150
home.css           0, 2, 2, 1, 2, 1, 2, 2, 1, 0, 1, 1, 1
```

**Risk — documented scale vs actual code:**

| Token (`variables.css`) | Documented | Actual code mein |
|---|---|---|
| `--z-header` | 1000 | `.ntb-navbar` = **1000** ✅ match |
| `--z-dropdown` | 50 *(comment: "airport picker, calendar, pax menu")* | `.ntb-airport-dropdown` = **50** ✅ · `.ntb-pax-menu` = **60** ❌ · `.ntb-calendar-menu` = **70** ❌ · `.ntb-currency-menu` = **200** ❌ |
| `--z-panel` | 1100 | `#modifySearchPanel .ntb-pax-menu` = **1100** ✅ match |
| `--z-sticky` / `--z-modal` / `--z-toast` | 100 / 2000 / 3000 | kahin use nahi |

Teen "dropdown" elements — jo ek hi comment ke under aate hain — teen alag values (50, 60, 70) par hain. Aur `results.css` mein `999` / `998` / `1200` / `1150` scale se poori tarah bahar hain. Ye **real stacking bugs** paida kar sakta hai (jab calendar aur pax menu dono open hon, ya modify panel ke andar dropdown khule).

**Fix:** har `z-index` ko `var(--z-*)` se replace karo, aur naye zaroorat wale levels `variables.css` mein add karo.

### 4.2 `--section-py` aur `--container-px` dead hain

- `--section-py: 80px` — define hai, use kahin nahi (aur `home.css` mein 56px override bhi bekaar).
- `--container-px: 2%` — define hai, lekin `base.css:40-41` hardcode `padding: 2%` likhta hai.

Yani spacing/container ka poora token system **bypass** ho raha hai — isi wajah se magic numbers har file mein repeat ho rahe hain.

### 4.3 Hardcoded colors jo tokens ko duplicate karte hain

| Color | Uses | Files |
|---|---:|---|
| `rgba(2,25,59,.18)` (= navy alpha) | 4 | components 3, results 1 |
| `rgba(255,255,255,.12)` | 3 | components, results, home |
| `rgba(253,112,20,.12)` (= orange alpha) | 2 | components, home |
| `rgba(2,25,59,.2)` | 2 | components, home |
| `rgba(255,255,255,.20)` | 2 | components, home |
| `rgba(2,25,59,.5)` | 2 | results ×2 |
| `rgba(253,112,20,.45)` | 2 | home ×2 |

### 4.4 🔴 Alpha values ka koi system nahi — 23 alag white alphas

`rgba(255,255,255,α)` ke liye **23 different alpha values** use hue hain, aur kuch jagah same value do tarah likhi gayi hai:

```
0.5  AND  0.50      ← same value, do formats
0.3  AND  0.30-ish overlapping variants
0.06, 0.08, 0.10, 0.12, 0.15, 0.20, 0.22, 0.25, 0.32,
0.40, 0.46, 0.50, 0.60, 0.62, 0.65, 0.70, 0.72, 0.75, 0.82, 0.84, 0.85
```

Ye duplicate nahi, **random noise** hai. Iske liye 4–5 tokens banane chahiye:
`--overlay-weak: .08` · `--overlay-soft: .20` · `--overlay-mid: .50` · `--overlay-strong: .70` · `--overlay-solid: .85`

---

## 5. Ek aur cheez jo mili (duplication nahi, lekin bug)

`components.css` mein `min-width` aur `max-width` media queries **mix** hain:

```
components.css:804   @media (min-width: 1024px)
components.css:1060  @media (max-width: 991px)
components.css:1128  @media (max-width: 767px)
components.css:1221  @media (max-width: 400px)
```

`991px` aur `1024px` ke beech **32px ka gap** hai (992px–1023px) jahan na `min-width:1024` chalega na `max-width:991`. Is range mein `.ntb-field`, `.ntb-search-row`, `.ntb-search-form`, `.ntb-search-btn` sab apni base values par rahenge. Tablet landscape par layout tootne ka chance hai.

**Fix:** ek hi approach chuno — ya sab `min-width` (mobile-first), ya sab `max-width`. Gap ke liye `min-width: 992px` use karo.

---

## 6. Priority-wise action plan

| # | Kaam | Impact | Effort |
|---|---|---|---|
| 1 | **`z-index` tokens apply karo** (35 hardcoded → `var(--z-*)`) | 🔴 High — real stacking bugs | Medium |
| 2 | `components.css` ke breakpoint gap (992–1023px) ko fix karo | 🔴 High — layout bug | Low |
| 3 | `--section-py` / `--container-px` ko ya use karo ya delete karo | 🟡 Medium — dead code | Low |
| 4 | `results.css` ke `[data-filter="stops"]`/`[data-filter="airlines"]` duplicate blocks ko ek attribute selector mein merge karo | 🟡 Medium — 4 groups, ~17 decls | Low |
| 5 | White-alpha values ko 5 tokens mein consolidate karo | 🟡 Medium — ~23 variants | Medium |
| 6 | Utility classes banao: `.u-flex-center`, `.u-flex-between`, `.u-truncate`, `.u-card`, `.u-stack` | 🟢 Long-term — 127 redundant decls | Medium |
| 7 | 18 cross-file identical bodies ko shared class mein nikaalo | 🟢 Long-term | Medium |
| 8 | 19 unused variables clean karo | 🟢 Cleanup | Low |
| 9 | `home.css` ka `:root` override `variables.css` mein move karo | 🟢 Cleanup | Low |

---

## 7. Kya **nahi** karna chahiye

- ❌ **Responsive overrides ko merge na karo.** `.ntb-search-row` ka base + 3 media variants — ye sahi hai. Inhe ek jagah club karne se cascade tootegi.
- ❌ **`display:flex` (120×) ko utility class mein convert karne ki jaldi na karo.** Ye 5 files mein 120 jagah hai; blindly replace karne se specificity aur `@media` override conflicts paida honge. Pehle sirf woh cases lo jahan poora pattern (flex + align + gap) repeat ho raha hai.
- ❌ **Files ko ek mein merge na karo.** Current separation (base / components / page-specific) sahi hai aur koi cross-file selector collision nahi hai.

---

*Generated from a full CSS parse of all 6 files — 820 rules / 3138 declarations analyzed.*
