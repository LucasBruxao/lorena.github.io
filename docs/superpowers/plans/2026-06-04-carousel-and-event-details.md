# Carousel + Configurable Events Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the static 3-column day timeline in `index.html` with a single-day carousel driven by inline JSON, supporting expandable event details and embedded OpenStreetMap views.

**Architecture:** All changes happen inside `index.html` to preserve the no-build-step / single-file constraint of this GitHub Pages project. JSON data is embedded as `<script type="application/json" id="schedule-data">`. CSS is appended to the existing `<style>` block. JS rendering and interaction code is appended to the existing `<script>` block (which already hosts the music player). Manual browser verification is used in lieu of an automated test suite — no test framework exists in this project, and adding one is out of scope.

**Tech Stack:** Vanilla HTML/CSS/JS. No frameworks. No build step. OpenStreetMap via official `embed.html` iframe.

**Spec:** `docs/superpowers/specs/2026-06-04-carousel-and-event-details-design.md`

---

## File Structure

Only one file is modified:

- **Modify:** `index.html` — three additive regions:
  - `<style>` block: append new carousel/accordion/map CSS at the end of the existing styles (before `</style>`).
  - `<body>` markup: replace the static `<section class="timeline">` block with a `<section class="carousel">` scaffold and add an inline `<script type="application/json" id="schedule-data">` block.
  - `<script>` block: append `renderSchedule()`, navigation, expansion, and map helpers after the existing music-player code.

No new files are created. No assets are added (event images, if any, are referenced from the existing `images/` folder and added by the user later).

---

## Task 1: Seed JSON data and scaffold carousel markup

**Files:**
- Modify: `index.html` (replace lines 631–683, the `<section class="timeline">` block and its preceding `<h2>`)
- Modify: `index.html` (insert a JSON script block before the existing `<script>` tag)

- [ ] **Step 1: Read `index.html` to confirm current state**

Run: `wc -l /Users/gabrielnaoto/Projects/lorena.github.io/index.html`
Expected: ~705 lines. Confirm lines 631–683 contain `<h2 class="section-title">Nossa programação</h2>` followed by the three static `<article class="day-card">` blocks.

- [ ] **Step 2: Replace the static timeline with the carousel scaffold**

Use Edit to replace this exact block:

```html
    <h2 class="section-title">Nossa programação</h2>

    <section class="timeline">
      <article class="day-card">
        <div class="day-top">
          <div class="day-number">12</div>
          <div class="day-date">
            <strong>12/06/2026</strong>
            <span>Sexta-feira</span>
          </div>
        </div>

        <ul class="plan-list">
          <li><span class="icon">🌷</span><span><strong>Primeiro:</strong> Pegar flores da Gazebo</span></li>
          <li><span class="icon">📸</span><span><strong>Depois:</strong> Tirar fotos lindas juntos</span></li>
          <li><span class="icon">🍹</span><span><strong>Jantar:</strong> Âmbar (18:30)</span></li>
        </ul>
      </article>

      <article class="day-card">
        <div class="day-top">
          <div class="day-number">13</div>
          <div class="day-date">
            <strong>13/06/2026</strong>
            <span>Sábado</span>
          </div>
        </div>

        <ul class="plan-list">
          <li><span class="icon">✂️</span><span><strong>De Manhã:</strong> Ir no Go Coffee Batel e depois Soaking (10:00)</span></li>
          <li><span class="icon">🍦</span><span><strong>Caminho de Tarde:</strong> Irado Gelateria e depois comprar cartinha da Copa na Praça da Espanha</span></li>
          <li><span class="icon">🍔</span><span><strong>Almoço:</strong> TT Burger</span></li>
          <li><span class="icon">⚽</span><span><strong>Jantar:</strong> Céu Bar no jogo do Brasil, Início (17:00)</span></li>
        </ul>
      </article>

      <article class="day-card">
        <div class="day-top">
          <div class="day-number">14</div>
          <div class="day-date">
            <strong>14/06/2026</strong>
            <span>Domingo</span>
          </div>
        </div>

        <ul class="plan-list">
          <li><span class="icon">☕</span><span><strong>De manhã:</strong> Cor Café</span></li>
          <li><span class="icon">🧑‍🍳</span><span><strong>Almoço:</strong> Vou cozinhar pra você</span></li>
          <li><span class="icon">😴</span><span><strong>De tarde:</strong> Descansar</span></li>
          <li><span class="icon">🍔</span><span><strong>Jantar:</strong> MC Donald's</span></li>
        </ul>
      </article>
    </section>
```

With:

```html
    <h2 class="section-title">Nossa programação</h2>

    <section class="carousel" aria-label="Programação dos dias">
      <button class="carousel-nav carousel-nav-prev" type="button" aria-label="Dia anterior">‹</button>
      <div class="carousel-viewport">
        <div class="carousel-track" aria-live="polite"></div>
      </div>
      <button class="carousel-nav carousel-nav-next" type="button" aria-label="Próximo dia">›</button>
      <div class="day-indicators" role="tablist" aria-label="Selecionar dia"></div>
    </section>
```

- [ ] **Step 3: Insert the JSON schedule data block**

Use Edit to replace this exact block (the start of the existing music script tag):

```html
  </main>
  <script>
    const music = document.getElementById("bgMusic");
```

With:

```html
  </main>

  <script type="application/json" id="schedule-data">
{
  "days": [
    {
      "date": "2026-06-12",
      "weekday": "Sexta-feira",
      "events": [
        { "icon": "🌷", "label": "Primeiro", "title": "Pegar flores da Gazebo" },
        { "icon": "📸", "label": "Depois", "title": "Tirar fotos lindas juntos" },
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30" }
      ]
    },
    {
      "date": "2026-06-13",
      "weekday": "Sábado",
      "events": [
        { "icon": "✂️", "label": "De Manhã", "title": "Ir no Go Coffee Batel e depois Soaking", "time": "10:00" },
        { "icon": "🍦", "label": "Caminho de Tarde", "title": "Irado Gelateria e depois comprar cartinha da Copa na Praça da Espanha" },
        { "icon": "🍔", "label": "Almoço", "title": "TT Burger" },
        { "icon": "⚽", "label": "Jantar", "title": "Céu Bar no jogo do Brasil", "time": "17:00" }
      ]
    },
    {
      "date": "2026-06-14",
      "weekday": "Domingo",
      "events": [
        { "icon": "☕", "label": "De manhã", "title": "Cor Café" },
        { "icon": "🧑‍🍳", "label": "Almoço", "title": "Vou cozinhar pra você" },
        { "icon": "😴", "label": "De tarde", "title": "Descansar" },
        { "icon": "🍔", "label": "Jantar", "title": "MC Donald's" }
      ]
    }
  ]
}
  </script>

  <script>
    const music = document.getElementById("bgMusic");
```

- [ ] **Step 4: Manual verification — JSON parses correctly**

Run: `open /Users/gabrielnaoto/Projects/lorena.github.io/index.html`

In browser DevTools console, run:

```js
JSON.parse(document.getElementById('schedule-data').textContent).days.length
```

Expected: `3`

The page itself shows no day cards yet (carousel is empty — that is correct for this task).

- [ ] **Step 5: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Seed inline JSON schedule and scaffold carousel container"
```

---

## Task 2: Render day cards from JSON

**Files:**
- Modify: `index.html` (append CSS rules before `</style>`)
- Modify: `index.html` (append JS at end of existing `<script>` tag, before `</script>`)

- [ ] **Step 1: Append carousel layout CSS**

Use Edit to replace this exact block (the very end of the `<style>` block):

```css
      .music-player button {
        padding: 9px 12px;
        font-size: 0.82rem;
      }
    }
  </style>
```

With:

```css
      .music-player button {
        padding: 9px 12px;
        font-size: 0.82rem;
      }
    }

    /* Carousel */
    .carousel {
      position: relative;
      margin-bottom: 24px;
    }

    .carousel-viewport {
      overflow: hidden;
      border-radius: 28px;
    }

    .carousel-track {
      display: flex;
      transition: transform 250ms ease;
      will-change: transform;
    }

    .carousel-track > .day-card {
      flex: 0 0 100%;
      width: 100%;
    }

    .carousel-nav {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      width: 44px;
      height: 44px;
      border-radius: 50%;
      background: rgba(255, 255, 255, 0.78);
      color: #7a5a4f;
      border: 1px solid rgba(255, 255, 255, 0.92);
      font-size: 1.6rem;
      line-height: 1;
      cursor: pointer;
      box-shadow: 0 10px 24px rgba(124, 97, 81, 0.16);
      backdrop-filter: blur(8px);
      -webkit-backdrop-filter: blur(8px);
      z-index: 2;
      display: grid;
      place-items: center;
      padding-bottom: 4px;
    }

    .carousel-nav:hover {
      background: rgba(255, 255, 255, 0.92);
    }

    .carousel-nav-prev {
      left: -10px;
    }

    .carousel-nav-next {
      right: -10px;
    }

    .day-indicators {
      display: flex;
      justify-content: center;
      gap: 8px;
      margin-top: 14px;
    }

    .day-indicator {
      width: 10px;
      height: 10px;
      border-radius: 50%;
      border: none;
      background: rgba(199, 143, 122, 0.32);
      cursor: pointer;
      padding: 0;
      transition: background 0.2s ease, transform 0.2s ease;
    }

    .day-indicator.is-active {
      background: var(--accent);
      transform: scale(1.25);
    }

    @media (max-width: 560px) {
      .carousel-nav {
        width: 38px;
        height: 38px;
        font-size: 1.3rem;
      }

      .carousel-nav-prev {
        left: 6px;
      }

      .carousel-nav-next {
        right: 6px;
      }
    }
  </style>
```

- [ ] **Step 2: Append the rendering JS**

Use Edit to replace this exact block (the existing `<script>` end):

```js
    musicToggle.addEventListener("click", function () {
      if (music.paused) {
        music.play();
        musicToggle.textContent = "Pausar música ♫";
      } else {
        music.pause();
        musicToggle.textContent = "Tocar música ♫";
      }
    });
  </script>
```

With:

```js
    musicToggle.addEventListener("click", function () {
      if (music.paused) {
        music.play();
        musicToggle.textContent = "Pausar música ♫";
      } else {
        music.pause();
        musicToggle.textContent = "Tocar música ♫";
      }
    });

    const scheduleData = JSON.parse(
      document.getElementById("schedule-data").textContent
    );

    function formatDateBR(isoDate) {
      const [y, m, d] = isoDate.split("-");
      return `${d}/${m}/${y}`;
    }

    function dayNumberFromIso(isoDate) {
      return isoDate.split("-")[2].replace(/^0/, "");
    }

    function buildDayCard(day) {
      const card = document.createElement("article");
      card.className = "day-card";

      const top = document.createElement("div");
      top.className = "day-top";

      const num = document.createElement("div");
      num.className = "day-number";
      num.textContent = dayNumberFromIso(day.date);

      const dateBlock = document.createElement("div");
      dateBlock.className = "day-date";
      const strong = document.createElement("strong");
      strong.textContent = formatDateBR(day.date);
      const span = document.createElement("span");
      span.textContent = day.weekday;
      dateBlock.appendChild(strong);
      dateBlock.appendChild(span);

      top.appendChild(num);
      top.appendChild(dateBlock);

      const list = document.createElement("ul");
      list.className = "plan-list";

      day.events.forEach(function (event) {
        const li = document.createElement("li");

        const iconSpan = document.createElement("span");
        iconSpan.className = "icon";
        iconSpan.textContent = event.icon;

        const textSpan = document.createElement("span");
        const labelStrong = document.createElement("strong");
        labelStrong.textContent = event.label + ":";
        textSpan.appendChild(labelStrong);
        textSpan.appendChild(document.createTextNode(" " + event.title));
        if (event.time) {
          textSpan.appendChild(document.createTextNode(" (" + event.time + ")"));
        }

        li.appendChild(iconSpan);
        li.appendChild(textSpan);
        list.appendChild(li);
      });

      card.appendChild(top);
      card.appendChild(list);
      return card;
    }

    function renderSchedule() {
      const track = document.querySelector(".carousel-track");
      const indicators = document.querySelector(".day-indicators");
      track.innerHTML = "";
      indicators.innerHTML = "";

      scheduleData.days.forEach(function (day, index) {
        track.appendChild(buildDayCard(day));

        const dot = document.createElement("button");
        dot.className = "day-indicator";
        dot.type = "button";
        dot.setAttribute("role", "tab");
        dot.setAttribute("aria-label", "Dia " + dayNumberFromIso(day.date));
        if (index === 0) {
          dot.classList.add("is-active");
        }
        indicators.appendChild(dot);
      });
    }

    renderSchedule();
  </script>
```

- [ ] **Step 3: Manual verification — all 3 day cards render**

Reload `index.html` in the browser.

Expected:
- The first day (12/06/2026 Sexta-feira) is visible in the carousel viewport.
- The other two days exist in DOM but are off to the right (clipped by `overflow: hidden`).
- Three indicator dots appear below the card; the leftmost is active (filled with accent color).
- Arrow buttons appear left and right of the card.
- Browser console shows no errors.

In DevTools console, run:

```js
document.querySelectorAll('.carousel-track > .day-card').length
```

Expected: `3`

- [ ] **Step 4: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Render day cards from JSON into carousel track"
```

---

## Task 3: Wire carousel navigation (arrows, dots, keyboard, initial day)

**Files:**
- Modify: `index.html` (extend the existing `<script>` block)

- [ ] **Step 1: Replace the `renderSchedule()` call with navigation wiring**

Use Edit to replace this exact block:

```js
    function renderSchedule() {
      const track = document.querySelector(".carousel-track");
      const indicators = document.querySelector(".day-indicators");
      track.innerHTML = "";
      indicators.innerHTML = "";

      scheduleData.days.forEach(function (day, index) {
        track.appendChild(buildDayCard(day));

        const dot = document.createElement("button");
        dot.className = "day-indicator";
        dot.type = "button";
        dot.setAttribute("role", "tab");
        dot.setAttribute("aria-label", "Dia " + dayNumberFromIso(day.date));
        if (index === 0) {
          dot.classList.add("is-active");
        }
        indicators.appendChild(dot);
      });
    }

    renderSchedule();
  </script>
```

With:

```js
    let currentDayIndex = 0;

    function pickInitialDayIndex() {
      const now = new Date();
      const pad = function (n) { return n < 10 ? "0" + n : "" + n; };
      const todayIso =
        now.getFullYear() + "-" + pad(now.getMonth() + 1) + "-" + pad(now.getDate());
      const match = scheduleData.days.findIndex(function (d) {
        return d.date === todayIso;
      });
      return match >= 0 ? match : 0;
    }

    function goToDay(index) {
      const count = scheduleData.days.length;
      currentDayIndex = ((index % count) + count) % count;
      const track = document.querySelector(".carousel-track");
      track.style.transform = "translateX(-" + (currentDayIndex * 100) + "%)";
      document.querySelectorAll(".day-indicator").forEach(function (dot, i) {
        dot.classList.toggle("is-active", i === currentDayIndex);
      });
    }

    function renderSchedule() {
      const track = document.querySelector(".carousel-track");
      const indicators = document.querySelector(".day-indicators");
      track.innerHTML = "";
      indicators.innerHTML = "";

      scheduleData.days.forEach(function (day, index) {
        track.appendChild(buildDayCard(day));

        const dot = document.createElement("button");
        dot.className = "day-indicator";
        dot.type = "button";
        dot.setAttribute("role", "tab");
        dot.setAttribute("aria-label", "Dia " + dayNumberFromIso(day.date));
        dot.addEventListener("click", function () {
          goToDay(index);
        });
        indicators.appendChild(dot);
      });

      document
        .querySelector(".carousel-nav-prev")
        .addEventListener("click", function () {
          goToDay(currentDayIndex - 1);
        });
      document
        .querySelector(".carousel-nav-next")
        .addEventListener("click", function () {
          goToDay(currentDayIndex + 1);
        });

      document
        .querySelector(".carousel")
        .addEventListener("keydown", function (e) {
          if (e.key === "ArrowLeft") {
            goToDay(currentDayIndex - 1);
          } else if (e.key === "ArrowRight") {
            goToDay(currentDayIndex + 1);
          }
        });
    }

    renderSchedule();
    goToDay(pickInitialDayIndex());
  </script>
```

- [ ] **Step 2: Manual verification — navigation works**

Reload `index.html` in browser.

Verify:
- Clicking the right arrow (`›`) slides to day 13. Clicking again slides to day 14. Clicking once more wraps back to day 12.
- Clicking the left arrow (`‹`) navigates in reverse and wraps.
- Clicking any indicator dot jumps directly to that day; the active dot updates.
- The slide animation is smooth (~250 ms).
- Focus an arrow button, then press the left/right arrow keys — the day changes.
- Today is 2026-06-04 (outside the trip range) → initial day should be 12.

To test the initial-day-in-range logic, in DevTools console run:

```js
const orig = Date;
window.Date = function () { return new orig("2026-06-13T10:00:00"); };
window.Date.prototype = orig.prototype;
location.reload();
```

(Actually `location.reload()` resets the override — instead simply open the file and stub before render. Alternative: temporarily edit the JSON so one of the days matches today's date and reload.)

Simpler manual check: temporarily change one day's `date` field in the inline JSON to today's date string (e.g. `"2026-06-04"`), reload, confirm that day is shown first, then revert the change.

- [ ] **Step 3: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Wire carousel arrows, indicators, keyboard nav, and initial day selection"
```

---

## Task 4: Add touch swipe support

**Files:**
- Modify: `index.html` (extend the `renderSchedule()` function inside `<script>`)

- [ ] **Step 1: Add touch handlers inside `renderSchedule()`**

Use Edit to replace this exact block:

```js
      document
        .querySelector(".carousel")
        .addEventListener("keydown", function (e) {
          if (e.key === "ArrowLeft") {
            goToDay(currentDayIndex - 1);
          } else if (e.key === "ArrowRight") {
            goToDay(currentDayIndex + 1);
          }
        });
    }
```

With:

```js
      document
        .querySelector(".carousel")
        .addEventListener("keydown", function (e) {
          if (e.key === "ArrowLeft") {
            goToDay(currentDayIndex - 1);
          } else if (e.key === "ArrowRight") {
            goToDay(currentDayIndex + 1);
          }
        });

      const viewport = document.querySelector(".carousel-viewport");
      let touchStartX = null;
      let touchStartY = null;

      viewport.addEventListener("touchstart", function (e) {
        if (e.touches.length !== 1) return;
        touchStartX = e.touches[0].clientX;
        touchStartY = e.touches[0].clientY;
      }, { passive: true });

      viewport.addEventListener("touchend", function (e) {
        if (touchStartX === null) return;
        const t = e.changedTouches[0];
        const dx = t.clientX - touchStartX;
        const dy = t.clientY - touchStartY;
        touchStartX = null;
        touchStartY = null;
        if (Math.abs(dx) < 40 || Math.abs(dx) < Math.abs(dy)) return;
        if (dx < 0) {
          goToDay(currentDayIndex + 1);
        } else {
          goToDay(currentDayIndex - 1);
        }
      });
    }
```

- [ ] **Step 2: Manual verification — swipe works**

Reload `index.html` in browser. Open DevTools, toggle device toolbar (mobile emulation).

Verify:
- Swiping left on the carousel viewport advances to the next day.
- Swiping right goes back.
- A purely vertical swipe (scrolling the page) does NOT change the day.
- A very short horizontal drag (< 40 px) does NOT change the day.

- [ ] **Step 3: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Add touch swipe navigation to carousel"
```

---

## Task 5: Implement expandable event panels (chevron + accordion)

**Files:**
- Modify: `index.html` (append CSS for expandable rows and panels)
- Modify: `index.html` (update `buildDayCard()` to render expandable rows)

- [ ] **Step 1: Append accordion CSS**

Use Edit to replace this exact block (the new carousel CSS block just before `</style>`):

```css
    @media (max-width: 560px) {
      .carousel-nav {
        width: 38px;
        height: 38px;
        font-size: 1.3rem;
      }

      .carousel-nav-prev {
        left: 6px;
      }

      .carousel-nav-next {
        right: 6px;
      }
    }
  </style>
```

With:

```css
    @media (max-width: 560px) {
      .carousel-nav {
        width: 38px;
        height: 38px;
        font-size: 1.3rem;
      }

      .carousel-nav-prev {
        left: 6px;
      }

      .carousel-nav-next {
        right: 6px;
      }
    }

    /* Expandable events */
    .plan-list li.is-expandable {
      flex-direction: column;
      align-items: stretch;
      gap: 0;
      padding: 0;
      cursor: pointer;
      transition: background 0.2s ease;
    }

    .plan-list li.is-expandable:hover {
      background: rgba(255, 255, 255, 0.5);
    }

    .plan-list li.is-expandable .event-row {
      display: flex;
      align-items: flex-start;
      gap: 10px;
      padding: 12px 14px;
    }

    .plan-list li .chevron {
      margin-left: auto;
      font-size: 0.95rem;
      color: var(--muted);
      transition: transform 0.2s ease;
      align-self: center;
    }

    .plan-list li.is-open .chevron {
      transform: rotate(180deg);
    }

    .event-expand-panel {
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.25s ease;
      padding: 0 14px;
    }

    .event-expand-panel.is-open {
      max-height: 720px;
      padding-bottom: 14px;
    }

    .event-expand-inner {
      display: flex;
      flex-direction: column;
      gap: 10px;
      padding: 14px;
      background: rgba(255, 255, 255, 0.42);
      border: 1px solid rgba(255, 255, 255, 0.5);
      border-radius: 16px;
    }
  </style>
```

- [ ] **Step 2: Update `buildDayCard()` to render expandable rows**

Use Edit to replace this exact block:

```js
      day.events.forEach(function (event) {
        const li = document.createElement("li");

        const iconSpan = document.createElement("span");
        iconSpan.className = "icon";
        iconSpan.textContent = event.icon;

        const textSpan = document.createElement("span");
        const labelStrong = document.createElement("strong");
        labelStrong.textContent = event.label + ":";
        textSpan.appendChild(labelStrong);
        textSpan.appendChild(document.createTextNode(" " + event.title));
        if (event.time) {
          textSpan.appendChild(document.createTextNode(" (" + event.time + ")"));
        }

        li.appendChild(iconSpan);
        li.appendChild(textSpan);
        list.appendChild(li);
      });
```

With:

```js
      day.events.forEach(function (event) {
        const li = document.createElement("li");

        const iconSpan = document.createElement("span");
        iconSpan.className = "icon";
        iconSpan.textContent = event.icon;

        const textSpan = document.createElement("span");
        const labelStrong = document.createElement("strong");
        labelStrong.textContent = event.label + ":";
        textSpan.appendChild(labelStrong);
        textSpan.appendChild(document.createTextNode(" " + event.title));
        if (event.time) {
          textSpan.appendChild(document.createTextNode(" (" + event.time + ")"));
        }

        const expandable = hasExpandableContent(event);
        if (expandable) {
          li.classList.add("is-expandable");
          li.setAttribute("role", "button");
          li.setAttribute("tabindex", "0");
          li.setAttribute("aria-expanded", "false");

          const row = document.createElement("div");
          row.className = "event-row";
          row.appendChild(iconSpan);
          row.appendChild(textSpan);

          const chevron = document.createElement("span");
          chevron.className = "chevron";
          chevron.textContent = "▾";
          chevron.setAttribute("aria-hidden", "true");
          row.appendChild(chevron);

          const panel = document.createElement("div");
          panel.className = "event-expand-panel";
          const inner = document.createElement("div");
          inner.className = "event-expand-inner";
          panel.appendChild(inner);
          renderExpandedContent(inner, event);

          li.appendChild(row);
          li.appendChild(panel);
          list.appendChild(li);

          const toggle = function () {
            const isOpen = li.classList.contains("is-open");
            list.querySelectorAll("li.is-expandable.is-open").forEach(function (other) {
              if (other !== li) {
                other.classList.remove("is-open");
                other.setAttribute("aria-expanded", "false");
                const otherPanel = other.querySelector(".event-expand-panel");
                if (otherPanel) otherPanel.classList.remove("is-open");
              }
            });
            li.classList.toggle("is-open", !isOpen);
            li.setAttribute("aria-expanded", !isOpen ? "true" : "false");
            panel.classList.toggle("is-open", !isOpen);
            if (!isOpen) {
              activateMapIfNeeded(panel, event);
            }
          };

          li.addEventListener("click", toggle);
          li.addEventListener("keydown", function (e) {
            if (e.key === "Enter" || e.key === " ") {
              e.preventDefault();
              toggle();
            }
          });
        } else {
          li.appendChild(iconSpan);
          li.appendChild(textSpan);
          list.appendChild(li);
        }
      });
```

- [ ] **Step 3: Add `hasExpandableContent`, `renderExpandedContent`, and `activateMapIfNeeded` stub helpers**

Place them ABOVE `buildDayCard`. Use Edit to replace this exact block:

```js
    function buildDayCard(day) {
```

With:

```js
    function hasExpandableContent(event) {
      return Boolean(
        event.description ||
        event.address ||
        event.image ||
        event.link ||
        (event.coords && event.coords.length === 2)
      );
    }

    function renderExpandedContent(inner, event) {
      // Content sub-elements added in Task 6.
    }

    function activateMapIfNeeded(panel, event) {
      // Lazy iframe src set in Task 7.
    }

    function buildDayCard(day) {
```

- [ ] **Step 4: Temporarily add a description to one event in the JSON to test expansion**

Use Edit to replace this exact block in the JSON:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30" }
```

With:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30", "description": "Bistrô francês intimista — nosso lugar especial." }
```

- [ ] **Step 5: Manual verification — chevron + accordion behavior**

Reload `index.html`.

Verify:
- Day 12's "Jantar: Âmbar" row now has a chevron `▾` on the right.
- The other rows do NOT have a chevron and do NOT respond to clicks.
- Click the Âmbar row → it gets the `is-open` class (the chevron rotates 180° to `▴`). The panel below opens (it's still empty until Task 6, but the height transitions visibly).
- Click again → it collapses.
- Press Tab to focus the row, press Enter → it toggles.
- Switch to day 13 via arrow → no expansion errors.

- [ ] **Step 6: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Add expandable event rows with chevron and accordion panel"
```

---

## Task 6: Render expanded panel content (image, description, address, link)

**Files:**
- Modify: `index.html` (fill in `renderExpandedContent()`)

- [ ] **Step 1: Implement `renderExpandedContent()`**

Use Edit to replace this exact block:

```js
    function renderExpandedContent(inner, event) {
      // Content sub-elements added in Task 6.
    }
```

With:

```js
    function renderExpandedContent(inner, event) {
      if (event.image) {
        const img = document.createElement("img");
        img.className = "event-image";
        img.src = event.image;
        img.alt = event.title;
        img.loading = "lazy";
        inner.appendChild(img);
      }

      if (event.description) {
        const p = document.createElement("p");
        p.className = "event-description";
        p.textContent = event.description;
        inner.appendChild(p);
      }

      if (event.address) {
        const addr = document.createElement("p");
        addr.className = "event-address";
        addr.textContent = "📍 " + event.address;
        inner.appendChild(addr);
      }

      if (event.coords && event.coords.length === 2) {
        const iframe = document.createElement("iframe");
        iframe.className = "event-map";
        iframe.setAttribute("title", "Mapa do local");
        iframe.setAttribute("loading", "lazy");
        iframe.setAttribute("referrerpolicy", "no-referrer-when-downgrade");
        iframe.dataset.lat = event.coords[0];
        iframe.dataset.lng = event.coords[1];
        inner.appendChild(iframe);
      }

      if (event.link) {
        const a = document.createElement("a");
        a.className = "event-link";
        a.href = event.link;
        a.target = "_blank";
        a.rel = "noopener noreferrer";
        a.textContent = "Abrir site ↗";
        inner.appendChild(a);
      }
    }
```

- [ ] **Step 2: Append CSS for expanded sub-elements**

Use Edit to replace this exact block:

```css
    .event-expand-inner {
      display: flex;
      flex-direction: column;
      gap: 10px;
      padding: 14px;
      background: rgba(255, 255, 255, 0.42);
      border: 1px solid rgba(255, 255, 255, 0.5);
      border-radius: 16px;
    }
  </style>
```

With:

```css
    .event-expand-inner {
      display: flex;
      flex-direction: column;
      gap: 10px;
      padding: 14px;
      background: rgba(255, 255, 255, 0.42);
      border: 1px solid rgba(255, 255, 255, 0.5);
      border-radius: 16px;
    }

    .event-image {
      width: 100%;
      max-height: 180px;
      object-fit: cover;
      border-radius: 14px;
      border: 1px solid rgba(255, 255, 255, 0.5);
    }

    .event-description {
      color: var(--text);
      line-height: 1.55;
      font-size: 0.98rem;
    }

    .event-address {
      color: var(--muted);
      font-size: 0.95rem;
    }

    .event-map {
      width: 100%;
      height: 180px;
      border: 0;
      border-radius: 14px;
      background: rgba(255, 255, 255, 0.4);
    }

    .event-link {
      align-self: flex-start;
      padding: 8px 14px;
      border-radius: 999px;
      background: rgba(199, 143, 122, 0.18);
      border: 1px solid rgba(199, 143, 122, 0.32);
      color: var(--accent);
      font-weight: 600;
      font-size: 0.92rem;
      text-decoration: none;
    }

    .event-link:hover {
      background: rgba(199, 143, 122, 0.28);
    }
  </style>
```

- [ ] **Step 3: Temporarily add address + link to the Âmbar event to test**

Use Edit to replace this exact block:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30", "description": "Bistrô francês intimista — nosso lugar especial." }
```

With:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30", "description": "Bistrô francês intimista — nosso lugar especial.", "address": "R. Mateus Leme, 950 - São Francisco, Curitiba", "link": "https://www.instagram.com/ambar_restaurante/" }
```

(Address and link are placeholders for verification — the user will adjust later. No coords yet; that's covered in Task 7.)

- [ ] **Step 4: Manual verification — expanded content renders**

Reload `index.html`. Open day 12 and click "Jantar: Âmbar".

Verify the expanded panel contains, top to bottom:
- A description paragraph ("Bistrô francês intimista…")
- An address line prefixed with 📍
- An "Abrir site ↗" pill button at the bottom-left

Clicking the link opens Instagram in a new tab (or whatever URL was set).

No image, no map (yet) — those fields are absent.

Network tab: no failed requests.

- [ ] **Step 5: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Render expanded event panel content (image, description, address, link)"
```

---

## Task 7: Lazy-load OpenStreetMap iframe on first expand

**Files:**
- Modify: `index.html` (implement `activateMapIfNeeded()`)

- [ ] **Step 1: Implement `activateMapIfNeeded()`**

Use Edit to replace this exact block:

```js
    function activateMapIfNeeded(panel, event) {
      // Lazy iframe src set in Task 7.
    }
```

With:

```js
    function activateMapIfNeeded(panel, event) {
      if (!event.coords || event.coords.length !== 2) return;
      const iframe = panel.querySelector("iframe.event-map");
      if (!iframe || iframe.src) return;

      const lat = Number(event.coords[0]);
      const lng = Number(event.coords[1]);
      const delta = 0.004;
      const minLon = lng - delta;
      const minLat = lat - delta;
      const maxLon = lng + delta;
      const maxLat = lat + delta;
      const bbox = minLon + "," + minLat + "," + maxLon + "," + maxLat;
      iframe.src =
        "https://www.openstreetmap.org/export/embed.html?bbox=" +
        encodeURIComponent(bbox) +
        "&layer=mapnik&marker=" +
        encodeURIComponent(lat + "," + lng);
    }
```

- [ ] **Step 2: Temporarily add `coords` to the Âmbar event for testing**

Use Edit to replace this exact block:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30", "description": "Bistrô francês intimista — nosso lugar especial.", "address": "R. Mateus Leme, 950 - São Francisco, Curitiba", "link": "https://www.instagram.com/ambar_restaurante/" }
```

With:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30", "description": "Bistrô francês intimista — nosso lugar especial.", "address": "R. Mateus Leme, 950 - São Francisco, Curitiba", "coords": [-25.4274, -49.2887], "link": "https://www.instagram.com/ambar_restaurante/" }
```

- [ ] **Step 3: Manual verification — map loads only on expand**

Reload `index.html` with DevTools Network tab open and filter on `openstreetmap`.

Verify:
- On initial page load, NO requests to `openstreetmap.org` are made.
- Click "Jantar: Âmbar" to expand.
- Now an OSM tile request appears in the Network tab.
- The iframe shows a map of São Francisco neighborhood in Curitiba with a marker.
- Collapse and re-expand — no new requests fire (already cached, `src` already set).

- [ ] **Step 4: Revert the Âmbar event back to a minimal entry**

The temporary description/address/coords/link were just for verification. Restore the original seeded line.

Use Edit to replace:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30", "description": "Bistrô francês intimista — nosso lugar especial.", "address": "R. Mateus Leme, 950 - São Francisco, Curitiba", "coords": [-25.4274, -49.2887], "link": "https://www.instagram.com/ambar_restaurante/" }
```

With:

```json
        { "icon": "🍹", "label": "Jantar", "title": "Âmbar", "time": "18:30" }
```

- [ ] **Step 5: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Lazy-load OpenStreetMap iframe on first event expand"
```

---

## Task 8: Mobile responsive polish + final smoke test

**Files:**
- Modify: `index.html` (small CSS adjustments)

- [ ] **Step 1: Add small-screen tweaks for the expanded panel and indicators**

Use Edit to replace this exact block:

```css
    .event-link:hover {
      background: rgba(199, 143, 122, 0.28);
    }
  </style>
```

With:

```css
    .event-link:hover {
      background: rgba(199, 143, 122, 0.28);
    }

    @media (max-width: 560px) {
      .event-map {
        height: 160px;
      }

      .event-expand-inner {
        padding: 12px;
        gap: 8px;
      }

      .day-indicator {
        width: 9px;
        height: 9px;
      }
    }
  </style>
```

- [ ] **Step 2: Final manual smoke test**

Reload `index.html` and walk through:

Desktop view:
- Day 12 is shown first (today is 2026-06-04, outside range → default to day 12).
- Right arrow → day 13 with 4 events, then day 14, then wraps back to 12.
- Indicator dots track the active day.
- Keyboard arrows work after focusing an arrow button.
- No console errors.
- No `openstreetmap.org` requests in the Network tab (no event currently has `coords` in the seeded JSON).

Mobile view (DevTools device toolbar, e.g. iPhone 13):
- Carousel arrows are inside the card corners and smaller.
- Swiping left/right changes day.
- Indicator dots are slightly smaller.
- Carousel + day cards do not overflow horizontally.

Music player still works (toggle button starts/pauses the audio).
Hearts background animation still runs.

- [ ] **Step 3: Commit**

```bash
git -C /Users/gabrielnaoto/Projects/lorena.github.io add index.html
git -C /Users/gabrielnaoto/Projects/lorena.github.io commit -m "Add mobile responsive polish for expanded panel and indicators"
```

---

## Verification Summary

After all tasks:

- Single `index.html` file unchanged in count.
- JSON-driven content: editing any field in `#schedule-data` (icon, label, title, time, description, address, coords, image, link) reflects on next reload.
- Carousel: one day at a time, arrows + dots + keyboard + swipe + auto-pick-today.
- Expandable events: only those with optional content; one-open-at-a-time per day.
- OSM map: lazy-loaded iframe per the spec; no network calls until the user expands an event with `coords`.
- Existing music player and decoration remain untouched.

The user can now fill in `coords`, `description`, `address`, `image`, `link` on any event without touching code.
