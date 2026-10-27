<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let { activeStep = 0 } = $props();

  let svgEl = $state(null);
  let width = $state(900);
  const height = 540;
  const margin = { top: 40, right: 28, bottom: 56, left: 70 };

  let rows = $state([]);
  let hoverYear = $state(null);

  $effect(() => {
    if (rows.length && svgEl) {
      void activeStep;
      render();
    }
  });

  onMount(async () => {
    rows = await d3.csv('/data/01-investment.csv', (d) => ({
      year: +d.year,
      region: d.region,
      category: d.category,
      value: +d.value_usd_billions_nominal,
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

  function getData() {
    const usByYear = d3.group(rows.filter(r => r.region === 'us'), r => r.year);
    const data = [];
    for (const [year, items] of usByYear) {
      const total = items.find(i => i.category === 'total_ai')?.value ?? 0;
      const gen = items.find(i => i.category === 'generative_ai')?.value ?? 0;
      data.push({ year, total, gen, nonGen: total - gen });
    }
    return data.sort((a, b) => a.year - b.year);
  }

  function render() {
    if (!svgEl || rows.length === 0) return;
    const data = getData();

    const svg = d3.select(svgEl)
      .attr('viewBox', `0 0 ${width} ${height}`)
      .attr('width', width)
      .attr('height', height);

    let root = svg.select('g.root');
    if (root.empty()) {
      svg.selectAll('*').remove();
      root = svg.append('g').attr('class', 'root');
      root.append('g').attr('class', 'grid');
      root.append('g').attr('class', 'bars-non-gen');
      root.append('g').attr('class', 'bars-gen');
      root.append('g').attr('class', 'axis-x');
      root.append('g').attr('class', 'axis-y');
      root.append('g').attr('class', 'legend');
      root.append('g').attr('class', 'annotations');
      root.append('g').attr('class', 'hover-overlay');
    }

    const x = d3.scaleBand()
      .domain(data.map(d => d.year))
      .range([margin.left, width - margin.right])
      .padding(0.22);

    const yMax = d3.max(data, d => d.total) * 1.05;
    const y = d3.scaleLinear()
      .domain([0, yMax])
      .range([height - margin.bottom, margin.top])
      .nice();

    // Grid
    root.select('g.grid')
      .selectAll('line')
      .data(y.ticks(5))
      .join('line')
      .attr('x1', margin.left)
      .attr('x2', width - margin.right)
      .attr('y1', d => y(d))
      .attr('y2', d => y(d))
      .attr('stroke', 'rgba(201,168,76,0.30)')
      .attr('stroke-dasharray', '2 3');

    // Step-based emphasis
    // 0: all visible normal
    // 1: 2024 highlighted (others fade slightly)
    // 2: 2025 highlighted
    // 3: All bars at full, gen AI portion saturated
    const yearOpacity = (year) => {
      if (activeStep === 1) return year === 2024 ? 1 : 0.35;
      if (activeStep === 2) return year === 2025 ? 1 : 0.35;
      return 1;
    };

    // Non-gen AI bars
    root.select('g.bars-non-gen')
      .selectAll('rect')
      .data(data, d => d.year)
      .join(
        enter => enter.append('rect')
          .attr('x', d => x(d.year))
          .attr('width', x.bandwidth())
          .attr('y', y(0))
          .attr('height', 0)
          .attr('fill', '#C9A84C'),
        update => update,
      )
      .transition().duration(700)
      .attr('x', d => x(d.year))
      .attr('width', x.bandwidth())
      .attr('y', d => y(d.nonGen))
      .attr('height', d => y(0) - y(d.nonGen))
      .attr('opacity', d => yearOpacity(d.year));

    // Gen AI bars (stacked on top)
    root.select('g.bars-gen')
      .selectAll('rect')
      .data(data.filter(d => d.gen > 0), d => d.year)
      .join(
        enter => enter.append('rect')
          .attr('x', d => x(d.year))
          .attr('width', x.bandwidth())
          .attr('y', d => y(d.nonGen))
          .attr('height', 0)
          .attr('fill', '#FFAB2E'),
        update => update,
      )
      .transition().duration(700)
      .attr('x', d => x(d.year))
      .attr('width', x.bandwidth())
      .attr('y', d => y(d.total))
      .attr('height', d => y(d.nonGen) - y(d.total))
      .attr('opacity', d => yearOpacity(d.year));

    // Axes
    const xAxis = d3.axisBottom(x)
      .tickValues(data.map(d => d.year).filter(y => y % 2 === 1))
      .tickFormat(d => `'${String(d).slice(2)}`);
    const yAxis = d3.axisLeft(y).ticks(5).tickFormat(d => `$${d}B`);

    root.select('g.axis-x')
      .attr('transform', `translate(0,${height - margin.bottom})`)
      .call(xAxis)
      .call(g => g.select('.domain').remove());
    root.select('g.axis-y')
      .attr('transform', `translate(${margin.left},0)`)
      .call(yAxis)
      .call(g => g.select('.domain').remove())
      .call(g => g.selectAll('.tick line').remove());

    // Legend
    const legend = root.select('g.legend')
      .attr('transform', `translate(${margin.left}, ${margin.top - 22})`);
    legend.selectAll('*').remove();
    legend.append('rect').attr('x', 0).attr('y', 0).attr('width', 12).attr('height', 12).attr('fill', '#C9A84C');
    legend.append('text').attr('class', 'annotation').attr('x', 18).attr('y', 10)
      .attr('font-size', 12).attr('fill', '#FFAB2E').text('All other AI');
    legend.append('rect').attr('x', 110).attr('y', 0).attr('width', 12).attr('height', 12).attr('fill', '#FFAB2E');
    legend.append('text').attr('class', 'annotation').attr('x', 128).attr('y', 10)
      .attr('font-size', 12).attr('fill', '#FFAB2E').attr('font-weight', 600).text('Generative AI');

    // Annotations
    const ann = root.select('g.annotations');
    ann.selectAll('*').remove();

    if (activeStep === 1) {
      const d2024 = data.find(d => d.year === 2024);
      ann.append('text')
        .attr('x', x(2024) + x.bandwidth() / 2)
        .attr('y', y(d2024.total) - 10)
        .attr('text-anchor', 'middle')
        .attr('font-size', 22)
        .attr('font-weight', 700)
        .attr('fill', '#FFD27A')
        .style('opacity', 0)
        .text('$109B')
        .transition().duration(600).style('opacity', 1);
      ann.append('text')
        .attr('x', x(2024) + x.bandwidth() / 2)
        .attr('y', y(d2024.total) + 10)
        .attr('text-anchor', 'middle')
        .attr('font-size', 11)
        .attr('fill', '#0A0A0F')
        .text('2024');
    }

    if (activeStep === 2) {
      const d2025 = data.find(d => d.year === 2025);
      ann.append('text')
        .attr('x', x(2025) + x.bandwidth() / 2)
        .attr('y', y(d2025.total) - 32)
        .attr('text-anchor', 'middle')
        .attr('font-size', 26)
        .attr('font-weight', 700)
        .attr('fill', '#FFD27A')
        .style('opacity', 0)
        .text('$286B')
        .transition().duration(600).style('opacity', 1);

      // YoY label, stacked just below $286B (still above the bar)
      ann.append('text')
        .attr('x', x(2025) + x.bandwidth() / 2)
        .attr('y', y(d2025.total) - 12)
        .attr('text-anchor', 'middle')
        .attr('font-size', 11)
        .attr('font-weight', 600)
        .attr('fill', '#FFAB2E')
        .style('opacity', 0)
        .text('+162% YoY')
        .transition().duration(600).delay(200).style('opacity', 1);
    }

    if (activeStep === 3) {
      const d2025 = data.find(d => d.year === 2025);
      // Highlight gen AI portion at top
      ann.append('text')
        .attr('x', x(2025) + x.bandwidth() / 2)
        .attr('y', y(d2025.total - d2025.gen / 2))
        .attr('text-anchor', 'middle')
        .attr('font-size', 14)
        .attr('font-weight', 700)
        .attr('fill', '#0A0A0F')
        .style('opacity', 0)
        .text('~½ gen AI')
        .transition().duration(600).style('opacity', 1);

      // Side annotation
      ann.append('text')
        .attr('x', margin.left + 8)
        .attr('y', margin.top + 30)
        .attr('font-size', 13)
        .attr('font-weight', 600)
        .attr('fill', '#FFAB2E')
        .style('opacity', 0)
        .text('Gen AI now equals all other AI combined.')
        .transition().duration(600).style('opacity', 1);
    }

    // X axis title
    root.selectAll('text.x-title').data([null]).join(
      enter => enter.append('text').attr('class', 'x-title label')
        .attr('text-anchor', 'middle')
        .attr('font-size', 12)
        .attr('fill', '#C9A84C')
        .text('U.S. private AI investment, billions of nominal USD'),
    )
    .attr('x', (margin.left + width - margin.right) / 2)
    .attr('y', height - 12);

    // Hover overlay
    setupHover(root, x, y, data);
  }

  function setupHover(root, x, y, data) {
    const overlay = root.select('g.hover-overlay');
    overlay.selectAll('*').remove();
    overlay.selectAll('rect.hit')
      .data(data, d => d.year)
      .join('rect')
      .attr('class', 'hit')
      .attr('x', d => x(d.year))
      .attr('y', margin.top)
      .attr('width', x.bandwidth())
      .attr('height', height - margin.top - margin.bottom)
      .attr('fill', 'transparent')
      .style('cursor', 'pointer')
      .on('mouseenter', (e, d) => hoverYear = d)
      .on('mouseleave', () => hoverYear = null);
  }
</script>

<div class="chart">
  <svg bind:this={svgEl}></svg>
  {#if hoverYear}
    <div class="tooltip">
      <div class="tt-year">{hoverYear.year}</div>
      <div class="tt-row">
        <span class="dot all"></span>
        <span class="name">Total private AI</span>
        <span class="val">${hoverYear.total.toFixed(1)}B</span>
      </div>
      {#if hoverYear.gen > 0}
        <div class="tt-row">
          <span class="dot gen"></span>
          <span class="name">of which gen AI</span>
          <span class="val">${hoverYear.gen.toFixed(1)}B</span>
        </div>
      {/if}
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
    max-height: 560px;
  }
  .tooltip {
    position: absolute;
    top: 12px;
    right: 12px;
    background: var(--paper);
    border: 1px solid var(--rule);
    border-radius: 3px;
    padding: 0.5rem 0.65rem;
    font-family: var(--sans);
    font-size: 0.78rem;
    pointer-events: none;
    box-shadow: 0 2px 6px rgba(0,0,0,0.05);
    min-width: 200px;
  }
  .tt-year {
    font-weight: 700;
    color: var(--ink);
    margin-bottom: 0.3rem;
  }
  .tt-row {
    display: grid;
    grid-template-columns: 10px 1fr auto;
    gap: 6px;
    align-items: center;
    color: var(--ink-soft);
    font-size: 0.75rem;
    line-height: 1.5;
  }
  .dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
  }
  .dot.all { background: #a8a299; }
  .dot.gen { background: #d65a3a; }
  .val {
    font-weight: 600;
    color: var(--ink);
    font-variant-numeric: tabular-nums;
  }
</style>
