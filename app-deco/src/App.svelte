<script>
  import Beat from './lib/Beat.svelte';
  import Chart1Investment from './lib/charts/Chart1Investment.svelte';
  import Chart2Geography from './lib/charts/Chart2Geography.svelte';
  import Chart3FDA from './lib/charts/Chart3FDA.svelte';
  import Chart4Perception from './lib/charts/Chart4Perception.svelte';
  import Chart5Teen from './lib/charts/Chart5Teen.svelte';
</script>

<main>
  <!-- Tactical status bar — frames the entire piece -->
  <header class="topbar">
    <span class="topbar-id">▶ AI NOW // COMMAND DECO</span>
    <span class="topbar-mode">AMBER TERMINAL · FIELD 01</span>
    <span class="topbar-time">REC // 2026.05.01</span>
  </header>

  <!-- Hero — Deco Monument template -->
  <section class="hero">
    <!-- Sunburst armature -->
    <div class="sunburst" aria-hidden="true">
      <svg viewBox="-100 -100 200 200" preserveAspectRatio="xMidYMid meet">
        {#each Array.from({length: 36}, (_, i) => i * 10) as a}
          <line x1="0" y1="-15" x2="0" y2="-95" stroke="currentColor" stroke-width="0.4" transform="rotate({a})" />
        {/each}
        <circle cx="0" cy="0" r="48" fill="none" stroke="currentColor" stroke-width="0.4" opacity="0.5" />
        <circle cx="0" cy="0" r="62" fill="none" stroke="currentColor" stroke-width="0.4" opacity="0.35" />
        <circle cx="0" cy="0" r="80" fill="none" stroke="currentColor" stroke-width="0.4" opacity="0.2" />
      </svg>
    </div>
    <div class="column">
      <p class="meta">CAP 6938 · INFOVIS · FINAL PROJECT · 機密</p>
      <h1>AI<br/>NOW</h1>
      <p class="hero-sub">2026.04 ▌ FIELD REPORT</p>
      <p class="dek">
        Three years ago, generative artificial intelligence barely existed outside research labs.
        Today, more than two in five working age Americans have used it.
      </p>
      <p class="dek-sub">
        What it costs, where it lives, what it does, how the world sees it,
        and where it has already taken root.
      </p>
      <p class="scroll-hint">▼ INITIATE SCROLL // 開始</p>
    </div>
  </section>

  <!-- Beat 1: The money -->
  <Beat
    kicker="The money"
    kickerJp="資本"
    headline="$286 billion, in a single year"
    intro="That is what private investors put into American AI companies in 2025. It is more than the previous two years combined, and roughly twelve times what they invested a decade earlier."
    steps={[
      `<p>The 2024 number was already a record: $109 billion.</p>`,
      `<p>One year later, the bar is more than two and a half times taller. The chart bends sharply upward at its right edge.</p>`,
      `<p>About half of that money went to companies building generative AI, a category that did not exist before 2022. A single funding round today can be larger than the entire generative AI market was two years ago.</p>`,
    ]}
    source={`Source: Stanford HAI, AI Index Report 2026, Chapter 4. Underlying data from Quid via S&amp;P Capital IQ and Crunchbase.`}
  >
    {#snippet children(activeStep)}
      <Chart1Investment {activeStep} />
    {/snippet}
  </Beat>

  <!-- Beat 2: Where it lives -->
  <Beat
    kicker="Where it lives"
    kickerJp="拠点"
    headline="Most of it runs through Virginia"
    intro="The data centers that train and serve American AI are not evenly spread. They cluster in a handful of metropolitan regions, and most of all in a corridor of windowless warehouses outside Washington known as Data Center Alley."
    steps={[
      `<p>At the end of 2025, Northern Virginia held more than 4,000 megawatts of operational capacity. That is nearly three times the next largest market.</p>`,
      `<p>Atlanta, Dallas-Fort Worth, Chicago, Phoenix and the Pacific Northwest follow. Together they account for the bulk of the country's inventory.</p>`,
      `<p>The regional grids that feed them are now the rate limiting factor on how fast American AI can grow.</p>`,
    ]}
    source={`Source: CBRE, H2 2025 North America Data Center Trends; per-market press releases. Silicon Valley figures reflect 2024 reporting; new construction stalled in 2025 because of power constraints.`}
  >
    {#snippet children(activeStep)}
      <Chart2Geography {activeStep} />
    {/snippet}
  </Beat>

  <!-- Beat 3: Inside the hospital -->
  <Beat
    kicker="Inside the hospital"
    kickerJp="病院"
    headline="1,430 medical devices, and counting"
    intro="While public attention has fixed on chatbots, the Food and Drug Administration has been quietly clearing AI to do something narrower: read X-rays, MRIs and CT scans."
    steps={[
      `<p>As of 2025, the agency had authorized 1,430 AI enabled medical devices for use in American clinics and hospitals. A decade earlier, the number was 42.</p>`,
      `<p>Through 2017, the F.D.A. had cleared just 91 such devices, scattered across multiple specialties.</p>`,
      `<p>By 2025, three out of every four new authorizations were AI tools that read X-rays, MRIs and CT scans. Most of the rest analyze the heart or the brain.</p>`,
    ]}
    source={`Source: F.D.A. Artificial Intelligence Enabled Medical Device List, March 9, 2026. Cumulative by year computed from Date of Final Decision.`}
  >
    {#snippet children(activeStep, scrollProgress)}
      <Chart3FDA {activeStep} {scrollProgress} />
    {/snippet}
  </Beat>

  <!-- Beat 4: How the world feels -->
  <Beat
    kicker="How the world feels"
    kickerJp="世論"
    headline="The countries building it are the most uneasy"
    intro="Each year, Ipsos asks adults in 30 countries whether AI offers more benefits than drawbacks. The map of the answers does not match the map of the investment."
    steps={[
      `<p>In Indonesia, 85 percent say yes. Thailand and Singapore are nearly as high. Across the Asia Pacific, public sentiment is overwhelmingly positive.</p>`,
      `<p>In the United States, the country investing the most capital, drawing the most power and authorizing the most devices, 42 percent agree.</p>`,
      `<p>France and Belgium sit at 41 percent. Canada, at 40, is lower still. The wealthy democracies whose firms are building the technology are the ones least sure they want it.</p>`,
    ]}
    source={`Source: Ipsos AI Monitor 2025 (June 2025; 30 countries; n = 23,216). The China figure (83 percent) is carried forward from the 2024 wave; China was not surveyed in 2025.`}
  >
    {#snippet children(activeStep)}
      <Chart4Perception {activeStep} />
    {/snippet}
  </Beat>

  <!-- Beat 5: In the bedroom -->
  <Beat
    kicker="In the bedroom"
    kickerJp="私室"
    headline="One in three has chosen the chatbot"
    intro="In the spring of 2025, Common Sense Media surveyed 1,060 American teenagers about their use of AI companions, chatbots designed for conversation rather than search."
    steps={[
      `<p>72 percent had used one at least once.</p>`,
      `<p>More than half use one at least a few times a month. One in eight uses one every day.</p>`,
      `<p>Among the teenagers who use them, one in three has chosen the chatbot over a real person to talk through something important. Three years ago, the behavior did not exist.</p>`,
    ]}
    source={`Source: Common Sense Media, "Talk, Trust, and Trade-Offs," July 16, 2025. Nationally representative survey of 1,060 American teenagers ages 13 to 17, fielded April–May 2025 by NORC at the University of Chicago.`}
  >
    {#snippet children(activeStep)}
      <Chart5Teen {activeStep} />
    {/snippet}
  </Beat>

  <!-- Close — Terminal Readout -->
  <section class="close" id="close">
    <div class="column">
      <p class="kicker">FINAL READOUT <span class="jp">最終報告</span></p>
      <h2>The order is the story</h2>
      <p>
        The boom is not a forecast. By the measures above, it is a balance sheet of <strong>$286 billion</strong>,
        a corridor of warehouses outside Washington, <strong>1,430</strong> cleared devices already inside American
        hospitals, and a chatbot that <strong>one in three</strong> teenage users has chosen over a person for a
        serious conversation.
      </p>
      <p>
        The capital arrived first. The infrastructure followed. The applications came next.
        Public hesitation arrived last, after the buildings were drawing power, the devices
        were inside hospitals, and the children were already talking.
      </p>
      <p>
        The gap between what has been built and what has been asked for is the story of the next decade.
      </p>

      <h3>Methods notes <span class="jp">注釈</span></h3>
      <ul class="methods">
        <li>
          <strong>CAPITAL.</strong> Private AI investment, in nominal U.S. dollars.
          Excludes publicly traded firms, internal corporate R&amp;D and public sector funding.
        </li>
        <li>
          <strong>GEOGRAPHY.</strong> Total inventory in megawatts, primary CBRE markets.
        </li>
        <li>
          <strong>APPLICATIONS.</strong> "AI enabled medical device" as defined by the F.D.A.'s
          Digital Health and A.I. Glossary; the agency notes that its list is not comprehensive.
        </li>
        <li>
          <strong>PERCEPTION.</strong> "Agree" combines top two boxes. Ipsos samples in
          middle income countries skew urban, educated and affluent, and are not nationally representative.
        </li>
        <li>
          <strong>TEENS.</strong> An AI companion is an AI tool used for conversation rather than
          search. The one in three figure has a base of teenagers ages 13 to 17 who have used such
          a tool (n = 758); across all American teenagers, the figure is closer to one in four.
        </li>
      </ul>

      <p class="meta credit">
        AI NOW // CAP 6938 INFOVIS // FINAL PROJECT // 2026.04 // END TRANSMISSION
      </p>
    </div>
  </section>
</main>

<style>
  /* ---- Tactical status bar ---- */
  .topbar {
    position: sticky;
    top: 0;
    z-index: 100;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 1.2rem;
    background: var(--field-deep);
    border-bottom: 1px solid var(--deco-gold);
    box-shadow: 0 0 12px rgba(201, 168, 76, 0.25);
    font-family: var(--tactical);
    font-size: 0.66rem;
    letter-spacing: 0.18em;
    text-transform: uppercase;
    color: var(--phosphor);
    text-shadow: var(--glow-phosphor);
  }
  .topbar-id { color: var(--phosphor-hi); }
  .topbar-mode { color: var(--deco-gold); text-shadow: 0 0 4px var(--deco-gold-dim); }
  .topbar-time { color: var(--confirm); text-shadow: var(--glow-confirm); animation: phosphor-flicker 8s linear infinite; }

  /* ---- Hero — Deco Monument ---- */
  .hero {
    position: relative;
    padding: 7rem 0 6rem;
    overflow: hidden;
    border-bottom: 1px solid var(--deco-gold);
    box-shadow: 0 0 12px var(--deco-gold-dim);
  }
  .sunburst {
    position: absolute;
    top: 50%;
    left: 50%;
    width: min(90vw, 900px);
    aspect-ratio: 1 / 1;
    transform: translate(-50%, -50%);
    color: var(--deco-gold);
    opacity: 0.55;
    pointer-events: none;
    z-index: 0;
    animation: ring-scan 240s linear infinite;
  }
  .sunburst svg { width: 100%; height: 100%; filter: drop-shadow(0 0 6px var(--deco-gold-dim)); }

  .hero .column { position: relative; z-index: 2; text-align: center; }
  .hero h1 {
    font-size: clamp(5rem, 16vw, 13rem);
    line-height: 0.95;
    letter-spacing: 0.16em;
    margin: 1rem 0 0.5rem;
  }
  .hero-sub {
    font-family: var(--tactical);
    font-size: 0.78rem;
    letter-spacing: 0.32em;
    color: var(--deco-gold);
    text-shadow: 0 0 6px var(--deco-gold-dim);
    text-transform: uppercase;
    margin: 0 0 2.6rem;
  }
  .dek {
    font-size: 1.16rem;
    color: var(--phosphor);
    margin: 1.6rem auto 1rem;
    text-align: left;
    max-width: 38rem;
    line-height: 1.6;
  }
  .dek-sub {
    font-size: 1rem;
    color: var(--phosphor-dim);
    text-shadow: none;
    text-align: left;
    max-width: 38rem;
    margin: 0 auto;
    font-style: italic;
  }
  .scroll-hint {
    font-family: var(--tactical);
    font-size: 0.72rem;
    color: var(--confirm);
    text-shadow: var(--glow-confirm);
    letter-spacing: 0.24em;
    text-transform: uppercase;
    margin-top: 4rem;
    animation: phosphor-flicker 4s linear infinite;
  }

  /* ---- Close — Terminal Readout ---- */
  .close {
    padding: 6rem 0 8rem;
    border-top: 1px solid var(--deco-gold);
    box-shadow: 0 0 12px var(--deco-gold-dim);
    background: linear-gradient(180deg, var(--field) 0%, var(--field-deep) 100%);
  }
  .close .kicker { justify-content: flex-start; margin-bottom: 1.2rem; }
  .close .jp {
    font-family: var(--jp);
    font-size: 0.62rem;
    color: var(--phosphor-dim);
    text-shadow: none;
    text-transform: none;
    letter-spacing: 0.08em;
    margin-left: 0.6em;
  }
  .close h3 .jp { font-size: 0.68rem; }

  .methods {
    list-style: none;
    padding: 0;
    margin: 1.2rem 0 2rem;
    max-width: var(--measure);
  }
  .methods li {
    border-top: 1px solid var(--deco-gold);
    padding: 0.9rem 0 0.9rem 0.9rem;
    font-size: 0.92rem;
    color: var(--phosphor-dim);
    text-shadow: none;
    border-left: 2px solid var(--deco-gold);
    margin-bottom: 0;
    background: rgba(201, 168, 76, 0.04);
  }
  .methods li:last-child { border-bottom: 1px solid var(--deco-gold); }
  .methods strong {
    font-family: var(--tactical);
    color: var(--deco-gold);
    text-shadow: 0 0 4px var(--deco-gold-dim);
    letter-spacing: 0.18em;
    margin-right: 0.5em;
    font-weight: 700;
  }
  .credit {
    margin-top: 2.4rem;
    font-size: 0.65rem;
    letter-spacing: 0.3em;
    color: var(--deco-gold);
    text-align: center;
    border-top: 1px dashed var(--deco-gold);
    padding-top: 1.4rem;
  }

  :global(.steps .hint) {
    font-family: var(--tactical);
    font-size: 0.7rem;
    color: var(--confirm);
    text-shadow: var(--glow-confirm);
    letter-spacing: 0.16em;
    text-transform: uppercase;
    margin-top: 1rem;
  }
</style>
