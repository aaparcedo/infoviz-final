<script>
  import { onMount } from 'svelte';

  let { kicker, headline, intro, steps = [], source, children } = $props();

  let stepRefs = $state([]);
  let activeStep = $state(0);
  let stepProgress = $state(0);

  // Total scroll progress through the beat: 0 at top of intro, 1 at bottom of last step.
  let scrollProgress = $derived(
    (activeStep + stepProgress) / Math.max(1, steps.length + 1)
  );

  // Manual scroll-driven progress tracker. Replaces scrollama so we can read
  // continuous progress (not just step-enter events) reliably across beats.
  let rafId = 0;
  function update() {
    const refs = stepRefs.filter(Boolean);
    if (refs.length === 0) return;
    const trigger = window.innerHeight * 0.55;
    let active = 0;
    let progress = 0;
    for (let i = 0; i < refs.length; i++) {
      const rect = refs[i].getBoundingClientRect();
      if (rect.top <= trigger) {
        active = i;
        const nextRef = refs[i + 1];
        const stepStart = rect.top;
        const stepEnd = nextRef ? nextRef.getBoundingClientRect().top : rect.top + rect.height;
        const range = stepEnd - stepStart;
        progress = range > 0 ? Math.max(0, Math.min(1, (trigger - stepStart) / range)) : 0;
      } else {
        break;
      }
    }
    activeStep = active;
    stepProgress = progress;
  }

  function onScroll() {
    if (rafId) cancelAnimationFrame(rafId);
    rafId = requestAnimationFrame(update);
  }

  onMount(() => {
    window.addEventListener('scroll', onScroll, { passive: true });
    window.addEventListener('resize', onScroll);
    update();
    return () => {
      window.removeEventListener('scroll', onScroll);
      window.removeEventListener('resize', onScroll);
      if (rafId) cancelAnimationFrame(rafId);
    };
  });
</script>

<section class="beat">
  <article class="steps">
    <div class="intro" bind:this={stepRefs[0]} class:is-active={activeStep === 0}>
      <p class="kicker">{kicker}</p>
      <h2>{headline}</h2>
      {#if intro}
        <p class="lede">{@html intro}</p>
      {/if}
    </div>

    {#each steps as step, i}
      <div class="step" bind:this={stepRefs[i + 1]} class:is-active={activeStep === i + 1}>
        {@html step}
      </div>
    {/each}

    {#if source}
      <p class="source">{@html source}</p>
    {/if}
  </article>

  <div class="graphic">
    <div class="graphic-inner">
      {@render children?.(activeStep, scrollProgress)}
    </div>
  </div>
</section>

<style>
  .beat {
    position: relative;
    display: flex;
    gap: 3rem;
    align-items: flex-start;
    max-width: 80rem;
    margin: 0 auto;
    padding: 4rem 1.5rem 6rem;
  }

  .steps {
    flex: 0 0 32%;
    max-width: 32%;
    position: relative;
    z-index: 1;
  }

  .graphic {
    flex: 0 0 calc(68% - 3rem);
    position: sticky;
    top: 0;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .graphic-inner {
    width: 100%;
  }

  .intro {
    min-height: 60vh;
    display: flex;
    flex-direction: column;
    justify-content: flex-end;
    padding-bottom: 1.5rem;
  }

  .step {
    min-height: 75vh;
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 1rem 0;
    transition: opacity 400ms ease, transform 400ms ease;
    opacity: 0.45;
    transform: translateY(8px);
  }

  .step.is-active {
    opacity: 1;
    transform: translateY(0);
  }

  .intro.is-active h2 {
    color: var(--ink);
  }

  :global(.steps p) {
    max-width: none;
    margin: 0 0 0.8em;
    line-height: 1.55;
    font-size: 1.02rem;
    color: var(--ink-soft);
  }

  :global(.steps .lede) {
    font-size: 1.08rem;
    color: var(--ink);
  }

  .source {
    font-family: var(--sans);
    font-size: 0.74rem;
    color: var(--ink-mute);
    font-style: italic;
    border-top: 1px solid var(--rule);
    padding-top: 1rem;
    margin-top: 2rem;
    line-height: 1.5;
  }

  /* Mobile: stack vertically, lose sticky */
  @media (max-width: 640px) {
    .beat {
      flex-direction: column;
      gap: 0;
      padding: 2rem 1rem 4rem;
    }
    .steps, .graphic {
      flex-basis: 100%;
      max-width: 100%;
    }
    .graphic {
      position: relative;
      height: auto;
      margin-bottom: 2rem;
    }
    .step { min-height: auto; padding: 1rem 0 2rem; }
  }
</style>
