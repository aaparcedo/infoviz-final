# AI Now — Presentation Notes

CAP 6938 Final Project. ~7 min talk + Q&A. Read top to bottom or pull what you need.

---

## 1. Open (30s)

> "This is *AI Now* — five charts on the measurable consequences of the AI boom. Capital, infrastructure, applications, public reaction, and the most intimate corner: how American teenagers use it."

**Pivot worth naming:**
- Originally planned an 8-chart historical timeline starting with Rosenblatt's perceptron in 1958.
- Cut it. The history was carrying weight the data couldn't support — a timeline of dots adds little beyond what the prose already says.
- New framing: open in the present, treat the boom as the assumed context, walk through measurable consequences. Less narration, more chart, more polish per chart.

---

## 2. The five charts (~4 min, ~45s each)

For each chart: the claim, the source, the one design choice worth pointing out.

### THE MONEY — $286 billion in a single year
- **Claim:** Private investment in U.S. AI nearly tripled in 2025 vs. 2024.
- **Source:** Stanford HAI, AI Index Report 2026.
- **Design choice:** The bar for 2025 dwarfs everything before it. The chart's geometry *is* the editorial point — no annotation needed.

### WHERE IT LIVES — Most of it runs through Virginia
- **Claim:** A handful of metros account for the bulk of U.S. data center capacity. Northern Virginia alone holds nearly 3× the next largest market.
- **Source:** CBRE H2 2025 North America Data Center Trends.
- **Design choice:** Bubble map rather than abstract bar chart. Makes "the cloud" concrete — these are buildings drawing electrons in specific places.

### INSIDE THE HOSPITAL — 1,430 medical devices and counting
- **Claim:** While public attention is on chatbots, the FDA has authorized 1,430 AI medical devices, three out of four reading radiology images.
- **Source:** FDA AI Enabled Medical Device List, March 2026 release.
- **Design choice (worth demoing):** A scroll-driven brush. As you scroll through this beat, a highlighted window slides across the timeline from 2015 toward 2025, and the panel mix below updates in lockstep. The reader doesn't need to interact — the scroll *is* the interaction.
- **Why this matters:** Borrowed the "scroll-as-steps" pattern from Bloomberg's *What's Really Warming the World* (Lecture 8 example). The chart confirms what the prose just said, just as Segel & Heer recommend.

### HOW THE WORLD FEELS — The countries building it are the most uneasy
- **Claim:** Indonesia 85%, U.S. 42%. The countries investing the most are the most skeptical.
- **Source:** Ipsos AI Monitor 2025, 30 countries, n = 23,216.
- **Design choice:** U.S. bar highlighted in coral against a faded gray field. "Population dim" rather than "population remove" — non-selected bars stay visible at low opacity for context.

### IN THE BEDROOM — One in three has chosen the chatbot
- **Claim:** Among American teen users of AI companions, one in three has chosen the chatbot over a person for a serious conversation.
- **Source:** Common Sense Media, "Talk, Trust, and Trade-Offs" (July 2025), n=1,060.
- **Design choice:** 100-cell waffle. Each cell = 1% of teens. Cells fill in cumulatively as you scroll, never flash. The slowness is editorial — it makes the count feel earned.

---

## 3. The design system (~1 min)

- **Single accent color throughout** — a warm coral. Reserved for the primary data series. Never decorative.
- **Serif body, sans labels** — newspaper register. Charts feel instrumental; prose feels human.
- **Sticky chart, scrolling text** — classic NYT graphics layout. The chart pins to the right; the prose advances on the left.
- **References:** NYT graphics desk, Bloomberg Businessweek, The Pudding. Specifically borrowed the scroll-driven brush from Bloomberg's "What's Really Warming the World," and the cumulative cell-fill pattern from Pudding's "24 Hours in an Invisible Epidemic."

---

## 4. Decisions worth mentioning (~1 min)

- **Cut from 7 charts to 5.** Dropped the adoption-curves chart (data was thin for the multi-decade comparison) and the energy-projection chart (felt redundant once geography made the same point concretely).
- **Cut the "drag the brush" interaction** from the FDA chart. Replaced with scroll-driven motion. Reader-driven exploration didn't fit the author-driven editorial register; making the brush move on its own kept the tour cohesive.
- **Cut the self-referential close.** Earlier draft ended with "the data is the most current available, conclusions are the reader's to draw." Replaced with a takeaway: *the order is the story.* Capital arrived first, infrastructure second, applications third, public hesitation last — after the buildings were already drawing power and the children were already talking.

---

## 5. Demo flow (open the page, scroll slowly)

1. Hero. Read the dek aloud — "Three years ago, generative AI barely existed outside research labs. Today, more than two in five working age Americans have used it."
2. Scroll to THE MONEY. Show the bar geometry doing the work.
3. Skip ahead to INSIDE THE HOSPITAL. **This is the demo moment** — scroll through it slowly to show the brush sliding and the panel mix updating.
4. Skip to IN THE BEDROOM. Scroll to show the waffle filling.
5. Land on the close. Read the takeaway aloud.

---

## 6. Q&A prep — likely questions

**"Why scrollytelling and not a dashboard?"**
> Author-driven beats reader-driven for an editorial argument with a thesis (Segel & Heer). The reader has one job: keep scrolling. The piece does the curatorial work.

**"Why drop the historical framing?"**
> The historical chart was just dots on a timeline — it told the reader something the prose already said. Cutting it gave back time to polish the consequence charts.

**"Where did the data come from?"**
> Each chart names its primary source on screen. The full provenance with URLs is in the project's MANIFEST.md. Stanford HAI for capital, CBRE for geography, FDA's public list for medical devices, Ipsos for perception, Common Sense Media for teens.

**"What would you do with more time?"**
> Add the data-center geography chart's morph from map to cartogram to grid-of-squares (SVT *Swedish Vote* technique). And ship distinct mobile chart geometries instead of CSS-shrinking the desktop ones.

**"What did you learn?"**
> The hardest call wasn't building any one chart — it was killing the ones that didn't earn their place. Five strong charts beat seven mixed ones.

---

## Closing line (~10s)

> "The boom is no longer a forecast. The order it arrived in — capital first, public reaction last — is the story of the next decade. Thanks. Happy to take questions."
