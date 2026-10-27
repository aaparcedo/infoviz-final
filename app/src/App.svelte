<script>
  import Beat from './lib/Beat.svelte';
  import Chart1Investment from './lib/charts/Chart1Investment.svelte';
  import Chart2Geography from './lib/charts/Chart2Geography.svelte';
  import Chart3FDA from './lib/charts/Chart3FDA.svelte';
  import Chart4Perception from './lib/charts/Chart4Perception.svelte';
  import Chart5Teen from './lib/charts/Chart5Teen.svelte';
</script>

<main>
  <!-- Hero -->
  <section class="hero">
    <div class="column">
      <p class="meta">CAP 6938 · Information Visualization · Final Project</p>
      <h1>AI Now</h1>
      <p class="dek">
        Three years ago, generative artificial intelligence barely existed outside research labs.
        Today, more than two in five working age Americans have used it.
      </p>
      <p class="dek-sub">
        What it costs, where it lives, what it does, how the world sees it,
        and where it has already taken root.
      </p>
      <p class="scroll-hint">↓ scroll</p>
    </div>
  </section>

  <!-- Beat 1: The money -->
  <Beat
    kicker="The money"
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

  <!-- Beat 3: Inside the hospital (with brushing + linking) -->
  <Beat
    kicker="Inside the hospital"
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

  <!-- Close -->
  <section class="close" id="close">
    <div class="column">
      <h2>The order is the story</h2>
      <p>
        The boom is not a forecast. By the measures above, it is a balance sheet of $286 billion,
        a corridor of warehouses outside Washington, 1,430 cleared devices already inside American
        hospitals, and a chatbot that one in three teenage users has chosen over a person for a
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

      <h3>Methods notes</h3>
      <ul class="methods">
        <li>
          <strong>Capital.</strong> Private AI investment, in nominal U.S. dollars.
          Excludes publicly traded firms, internal corporate R&amp;D and public sector funding.
        </li>
        <li>
          <strong>Geography.</strong> Total inventory in megawatts, primary CBRE markets.
        </li>
        <li>
          <strong>Applications.</strong> "AI enabled medical device" as defined by the F.D.A.'s
          Digital Health and A.I. Glossary; the agency notes that its list is not comprehensive.
        </li>
        <li>
          <strong>Perception.</strong> "Agree" combines top two boxes. Ipsos samples in
          middle income countries skew urban, educated and affluent, and are not nationally representative.
        </li>
        <li>
          <strong>Teens.</strong> An AI companion is an AI tool used for conversation rather than
          search. The one in three figure has a base of teenagers ages 13 to 17 who have used such
          a tool (n = 758); across all American teenagers, the figure is closer to one in four.
        </li>
      </ul>

      <p class="meta credit">
        AI Now · CAP 6938 Information Visualization · Final Project · April 2026
      </p>
    </div>
  </section>
</main>

<style>
  .hero {
    padding: 8rem 0 5rem;
    border-bottom: 1px solid var(--rule);
  }
  .dek {
    font-size: 1.18rem;
    color: var(--ink-soft);
    margin-top: 1.6rem;
  }
  .dek-sub {
    font-size: 1.02rem;
    color: var(--ink-soft);
  }
  .scroll-hint {
    font-family: var(--sans);
    font-size: 0.78rem;
    color: var(--ink-mute);
    letter-spacing: 0.12em;
    text-transform: uppercase;
    margin-top: 4rem;
  }
  .close {
    padding: 6rem 0 8rem;
    border-top: 1px solid var(--rule);
  }
  .methods {
    list-style: none;
    padding: 0;
    margin: 1.2rem 0 2rem;
    max-width: var(--measure);
  }
  .methods li {
    border-top: 1px solid var(--rule);
    padding: 0.8rem 0;
    font-size: 0.94rem;
    color: var(--ink-soft);
  }
  .methods li:last-child { border-bottom: 1px solid var(--rule); }
  .methods strong { color: var(--ink); }
  .credit { margin-top: 2.4rem; }

  :global(.steps .hint) {
    font-family: var(--sans);
    font-size: 0.72rem;
    color: var(--ink-mute);
    letter-spacing: 0.1em;
    text-transform: uppercase;
    margin-top: 1rem;
  }
</style>
