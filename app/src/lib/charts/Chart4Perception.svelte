<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let { activeStep = 0 } = $props();

  let svgEl = $state(null);
  let width = $state(900);
  const m = { top: 28, right: 80, bottom: 50, left: 130 };

  let rows = $state([]);
  let hovered = $state(null);

  $effect(() => {
    if (rows.length && svgEl) {
      void activeStep;
      render();
    }
  });

  onMount(async () => {
    rows = await d3.csv('/data/04-perception.csv', (d) => ({
      country: d.country,
      iso3: d.iso3,
      pct: +d.pct_agree,
      year: +d.year,
      isUS: d.is_us === '1',
    }));

    const ro = new ResizeObserver(() => {
      if (!svgEl) return;
      width = svgEl.parentElement.clientWidth;
      render();
    });
    if (svgEl) ro.observe(svgEl.parentElement);
    width = svgEl ? svgEl.parentElement.clientWidth : 900;
    render();
  });

  function render() {
    if (!svgEl || rows.length === 0) return;

    const data = [...rows].sort((a, b) => b.pct - a.pct);
    const rowH = 18;
    const height = m.top + m.bottom + data.length * rowH;

    const svg = d3.select(svgEl)
      .attr('viewBox', `0 0 ${width} ${height}`)
      .attr('width', width)
      .attr('height', height);

    let root = svg.select('g.root');
    if (root.empty()) {
      svg.selectAll('*').remove();
      root = svg.append('g').attr('class', 'root');
      root.append('g').attr('class', 'grid');
      root.append('g').attr('class', 'x-ticks');
      root.append('g').attr('class', 'bars');
      root.append('g').attr('class', 'country-labels');
      root.append('g').attr('class', 'value-labels');
      root.append('text').attr('class', 'x-title label');
    }

    const x = d3.scaleLinear().domain([0, 100]).range([m.left, width - m.right]);
    const y = d3.scaleBand().domain(data.map(d => d.country)).range([m.top, height - m.bottom]).padding(0.22);

    // Step-driven highlighting
    // 0: all visible normal
    // 1: highlight top 3
    // 2: highlight US
    // 3: highlight bottom 4
    const top3 = ['Indonesia', 'China', 'Thailand'];
    const bottom4 = ['United States', 'Belgium', 'France', 'Canada'];
    const colorFor = (d) => {
      if (activeStep === 1) return top3.includes(d.country) ? '#d65a3a' : '#a8a299';
      if (activeStep === 2) return d.isUS ? '#d65a3a' : '#a8a299';
      if (activeStep === 3) return bottom4.includes(d.country) ? '#d65a3a' : '#a8a299';
      // Step 0 default — keep US accent
      return d.isUS ? '#d65a3a' : '#a8a299';
    };
    const opacityFor = (d) => {
      if (activeStep === 1) return top3.includes(d.country) ? 1 : 0.35;
      if (activeStep === 2) return d.isUS ? 1 : 0.55;
      if (activeStep === 3) return bottom4.includes(d.country) ? 1 : 0.35;
      return 1;
    };

    // Grid
    root.select('g.grid')
      .selectAll('line')
      .data([20, 40, 60, 80, 100])
      .join('line')
      .attr('x1', d => x(d)).attr('x2', d => x(d))
      .attr('y1', m.top - 4).attr('y2', height - m.bottom)
      .attr('stroke', '#e3ddd2').attr('stroke-dasharray', '2 3');

    // X axis ticks (top)
    root.select('g.x-ticks')
      .selectAll('text')
      .data([0, 20, 40, 60, 80, 100])
      .join('text')
      .attr('x', d => x(d)).attr('y', m.top - 10)
      .attr('text-anchor', 'middle')
      .attr('font-size', 10).attr('fill', '#8b857d')
      .attr('class', 'axis')
      .text(d => `${d}%`);

    // Bars
    root.select('g.bars')
      .selectAll('rect')
      .data(data, d => d.country)
      .join(
        enter => enter.append('rect')
          .attr('x', x(0))
          .attr('y', d => y(d.country))
          .attr('width', 0)
          .attr('height', y.bandwidth())
          .attr('fill', d => colorFor(d))
          .style('cursor', 'pointer')
          .on('mouseenter', (e, d) => hovered = d)
          .on('mouseleave', () => hovered = null),
        update => update,
      )
      .transition().duration(600)
      .attr('y', d => y(d.country))
      .attr('width', d => x(d.pct) - x(0))
      .attr('height', y.bandwidth())
      .attr('fill', d => colorFor(d))
      .attr('opacity', d => opacityFor(d));

    // Country labels
    root.select('g.country-labels')
      .selectAll('text')
      .data(data, d => d.country)
      .join('text')
      .attr('class', 'annotation')
      .attr('x', m.left - 8)
      .attr('y', d => y(d.country) + y.bandwidth() / 2)
      .attr('text-anchor', 'end').attr('alignment-baseline', 'middle')
      .attr('font-size', 12)
      .text(d => d.iso3 === 'CHN' ? `${d.country}†` : d.country)
      .transition().duration(600)
      .attr('font-weight', d => {
        if (activeStep === 1 && top3.includes(d.country)) return 700;
        if (activeStep === 2 && d.isUS) return 700;
        if (activeStep === 3 && bottom4.includes(d.country)) return 700;
        if (d.isUS) return 700;
        return 400;
      })
      .attr('fill', d => {
        if (activeStep === 1 && top3.includes(d.country)) return '#d65a3a';
        if (activeStep === 2 && d.isUS) return '#d65a3a';
        if (activeStep === 3 && bottom4.includes(d.country)) return '#d65a3a';
        return d.isUS ? '#d65a3a' : '#4a4641';
      })
      .attr('opacity', d => opacityFor(d));

    // Value labels
    root.select('g.value-labels')
      .selectAll('text')
      .data(data, d => d.country)
      .join('text')
      .attr('class', 'annotation')
      .attr('y', d => y(d.country) + y.bandwidth() / 2)
      .attr('alignment-baseline', 'middle')
      .attr('font-size', 11)
      .text(d => `${d.pct}%`)
      .transition().duration(600)
      .attr('x', d => x(d.pct) + 5)
      .attr('font-weight', d => {
        if (activeStep === 1 && top3.includes(d.country)) return 700;
        if (activeStep === 2 && d.isUS) return 700;
        if (activeStep === 3 && bottom4.includes(d.country)) return 700;
        if (d.isUS) return 700;
        return 500;
      })
      .attr('fill', d => {
        if (activeStep === 1 && top3.includes(d.country)) return '#d65a3a';
        if (activeStep === 2 && d.isUS) return '#d65a3a';
        if (activeStep === 3 && bottom4.includes(d.country)) return '#d65a3a';
        return d.isUS ? '#d65a3a' : '#4a4641';
      })
      .attr('opacity', d => opacityFor(d));

    // X title
    root.select('text.x-title')
      .attr('x', (m.left + width - m.right) / 2)
      .attr('y', height - 18)
      .attr('text-anchor', 'middle')
      .attr('font-size', 11).attr('fill', '#8b857d')
      .text('% who agree AI offers more benefits than drawbacks');

    // China dagger note rendered inline in the country label; the source
    // citation below the chart spells out the year-mismatch caveat in full.
  }
</script>

<div class="chart">
  <svg bind:this={svgEl}></svg>
  {#if hovered}
    <div class="tooltip">
      <div class="name">{hovered.country}</div>
      <div class="val">{hovered.pct}% agree</div>
    </div>
  {/if}
</div>

<style>
  .chart {
    width: 100%;
    position: relative;
  }
  svg {
    width: 100%;
    height: auto;
  }
  .tooltip {
    position: absolute;
    top: 12px;
    right: 12px;
    background: var(--paper);
    border: 1px solid var(--rule);
    border-radius: 3px;
    padding: 0.45rem 0.6rem;
    font-family: var(--sans);
    font-size: 0.78rem;
    pointer-events: none;
    box-shadow: 0 2px 6px rgba(0,0,0,0.05);
    min-width: 140px;
  }
  .name {
    font-weight: 700;
    color: var(--ink);
    margin-bottom: 0.15rem;
  }
  .val {
    color: var(--accent);
    font-weight: 600;
  }
</style>
