<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let { activeStep = 0, scrollProgress = 0 } = $props();

  let svgEl = $state(null);
  let width = $state(900);
  // Two stacked panels: cumulative line on top, panel breakdown below
  const totalH = 620;
  const m = { top: 36, right: 64, bottom: 16, left: 130 };
  const topH = 280;
  const bottomH = 200;
  const gap = 72;

  let rawRows = $state([]); // every authorization, with year + panel
  let cumulativeRows = $state([]);

  // Brush window — slides through the timeline as the user scrolls.
  // Hidden during intro, then sweeps from minYear to (maxYear - windowSize).
  const minYear = 2015;
  const maxYear = 2025;
  const windowSize = 3;
  const introFrac = 0.25; // first quarter of the beat's scroll is the intro

  let showBrush = $derived(activeStep >= 1);
  let brushStart = $derived.by(() => {
    if (!showBrush) return null;
    const useful = Math.max(0, Math.min(1, (scrollProgress - introFrac) / (1 - introFrac)));
    return minYear + useful * (maxYear - minYear - windowSize);
  });
  let brushEnd = $derived(brushStart != null ? brushStart + windowSize : null);
  // Integer year endpoints for readout and panel mix (only changes when boundaries cross)
  let brushYearRange = $derived(
    brushStart != null ? [Math.round(brushStart), Math.round(brushEnd)] : null
  );

  $effect(() => {
    if (rawRows.length && cumulativeRows.length && svgEl) {
      void activeStep;
      void brushStart;
      void brushYearRange;
      render();
    }
  });

  onMount(async () => {
    const [raw, cum] = await Promise.all([
      d3.csv('/data/03-fda-raw.csv', (d) => {
        const dateStr = d['Date of Final Decision'];
        const parts = dateStr.split('/');
        const year = parts.length === 3 ? +parts[2] : NaN;
        return {
          year,
          panel: d['Panel (Lead)']?.trim() ?? 'Unknown',
        };
      }),
      d3.csv('/data/03-fda-cumulative.csv', (d) => ({
        year: +d.year,
        yearCount: +d.year_count,
        cumulative: +d.cumulative_count,
      })),
    ]);
    rawRows = raw.filter(r => !isNaN(r.year) && r.year >= 2015 && r.year <= 2025);
    cumulativeRows = cum.filter(r => r.year >= 2015 && r.year <= 2025).sort((a, b) => a.year - b.year);

    const ro = new ResizeObserver(() => {
      if (!svgEl) return;
      width = svgEl.parentElement.clientWidth;
      render();
    });
    if (svgEl) ro.observe(svgEl.parentElement);
    width = svgEl ? svgEl.parentElement.clientWidth : 900;
    render();
  });

  function getPanelMix(yearMin, yearMax) {
    const subset = rawRows.filter(r => r.year >= yearMin && r.year <= yearMax);
    const counts = d3.rollup(subset, v => v.length, d => {
      // Bucket smaller panels into "Other"
      if (d.panel === 'Radiology' || d.panel === 'Cardiovascular' || d.panel === 'Neurology') {
        return d.panel;
      }
      return 'Other';
    });
    const total = subset.length || 1;
    const order = ['Radiology', 'Cardiovascular', 'Neurology', 'Other'];
    return order.map(p => ({
      panel: p,
      count: counts.get(p) ?? 0,
      pct: ((counts.get(p) ?? 0) / total) * 100,
      total,
    }));
  }

  function render() {
    if (!svgEl || rawRows.length === 0) return;

    const svg = d3.select(svgEl)
      .attr('viewBox', `0 0 ${width} ${totalH}`)
      .attr('width', width)
      .attr('height', totalH);

    let root = svg.select('g.root');
    if (root.empty()) {
      svg.selectAll('*').remove();
      root = svg.append('g').attr('class', 'root');
      // Top panel — cumulative line
      const top = root.append('g').attr('class', 'top-panel').attr('transform', `translate(0, ${m.top})`);
      top.append('g').attr('class', 'grid');
      top.append('path').attr('class', 'area');
      top.append('path').attr('class', 'line');
      top.append('g').attr('class', 'dots');
      top.append('g').attr('class', 'axis-x');
      top.append('g').attr('class', 'axis-y');
      top.append('g').attr('class', 'annotations');
      top.append('g').attr('class', 'brush-layer');
      // Bottom panel — panel mix bars
      const bot = root.append('g').attr('class', 'bot-panel')
        .attr('transform', `translate(0, ${m.top + topH + gap})`);
      bot.append('g').attr('class', 'bot-bars');
      bot.append('g').attr('class', 'bot-axis-x');
      bot.append('g').attr('class', 'bot-axis-y');
      bot.append('text').attr('class', 'bot-title label');
      bot.append('text').attr('class', 'bot-subtitle label');
    }

    // ---------- Top panel (cumulative line) ----------
    const x = d3.scaleLinear().domain([2015, 2025]).range([m.left, width - m.right]);
    const y = d3.scaleLinear().domain([0, 1500]).range([topH, 0]);

    const top = root.select('g.top-panel');

    top.select('g.grid')
      .selectAll('line')
      .data(y.ticks(5))
      .join('line')
      .attr('x1', m.left).attr('x2', width - m.right)
      .attr('y1', d => y(d)).attr('y2', d => y(d))
      .attr('stroke', '#e3ddd2').attr('stroke-dasharray', '2 3');

    const area = d3.area()
      .x(d => x(d.year))
      .y0(y(0))
      .y1(d => y(d.cumulative))
      .curve(d3.curveMonotoneX);
    const line = d3.line()
      .x(d => x(d.year))
      .y(d => y(d.cumulative))
      .curve(d3.curveMonotoneX);

    top.select('path.area')
      .datum(cumulativeRows)
      .attr('fill', '#d65a3a')
      .attr('opacity', 0.18)
      .attr('d', area);
    top.select('path.line')
      .datum(cumulativeRows)
      .attr('fill', 'none')
      .attr('stroke', '#d65a3a')
      .attr('stroke-width', 2.6)
      .attr('d', line);

    top.select('g.dots')
      .selectAll('circle')
      .data(cumulativeRows, d => d.year)
      .join('circle')
      .attr('cx', d => x(d.year))
      .attr('cy', d => y(d.cumulative))
      .attr('r', d => d.year === 2025 ? 5 : 2.5)
      .attr('fill', '#d65a3a');

    // Annotations per step
    const ann = top.select('g.annotations');
    ann.selectAll('*').remove();
    const last = cumulativeRows[cumulativeRows.length - 1];
    const first = cumulativeRows[0];
    if (activeStep === 1) {
      ann.append('text')
        .attr('x', x(last.year) - 8).attr('y', y(last.cumulative) - 14)
        .attr('text-anchor', 'end')
        .attr('font-size', 16).attr('font-weight', 700)
        .attr('fill', '#14110d')
        .style('opacity', 0)
        .text('1,430 devices')
        .transition().duration(500).style('opacity', 1);
      ann.append('text')
        .attr('x', x(last.year) - 8).attr('y', y(last.cumulative) + 4)
        .attr('text-anchor', 'end')
        .attr('font-size', 11)
        .attr('fill', '#4a4641')
        .style('opacity', 0)
        .text('cumulative through 2025')
        .transition().duration(500).style('opacity', 1);
      ann.append('text')
        .attr('x', x(first.year) + 6).attr('y', y(first.cumulative) - 8)
        .attr('font-size', 11).attr('fill', '#8b857d')
        .text(`${first.cumulative} in ${first.year}`);
    }

    // Axes (top)
    const xAxisTop = d3.axisBottom(x).tickValues([2015, 2017, 2019, 2021, 2023, 2025]).tickFormat(d3.format('d'));
    const yAxisTop = d3.axisLeft(y).ticks(5).tickFormat(d3.format(','));
    top.select('g.axis-x')
      .attr('transform', `translate(0,${topH})`)
      .call(xAxisTop)
      .call(g => g.select('.domain').remove());
    top.select('g.axis-y')
      .attr('transform', `translate(${m.left},0)`)
      .call(yAxisTop)
      .call(g => g.select('.domain').remove())
      .call(g => g.selectAll('.tick line').remove());

    // Title above top
    top.selectAll('text.top-title').data([null]).join(
      enter => enter.append('text').attr('class', 'top-title label')
        .attr('font-size', 11).attr('fill', '#8b857d')
        .text('FDA AI medical device authorizations'),
    ).attr('x', m.left - 4).attr('y', -14);

    // ---------- Brush (scroll-driven, non-interactive) ----------
    const brushG = top.select('g.brush-layer');
    brushG.selectAll('*').remove();
    if (brushStart != null) {
      const x0 = x(brushStart);
      const x1 = x(brushEnd);
      brushG.append('rect')
        .attr('x', x0)
        .attr('y', 0)
        .attr('width', x1 - x0)
        .attr('height', topH)
        .attr('fill', '#d65a3a')
        .attr('fill-opacity', 0.12)
        .attr('stroke', '#d65a3a')
        .attr('stroke-width', 1)
        .attr('pointer-events', 'none');
    }

    // ---------- Bottom panel (panel mix) ----------
    const yearMin = brushYearRange ? brushYearRange[0] : 2015;
    const yearMax = brushYearRange ? brushYearRange[1] : 2025;
    const mix = getPanelMix(yearMin, yearMax);

    const bot = root.select('g.bot-panel');
    const xBot = d3.scaleLinear().domain([0, 100]).range([m.left, width - m.right]);
    const yBot = d3.scaleBand()
      .domain(mix.map(d => d.panel))
      .range([0, bottomH])
      .padding(0.18);

    const colors = {
      Radiology: '#d65a3a',
      Cardiovascular: '#a8a299',
      Neurology: '#cfa685',
      Other: '#e3ddd2',
    };

    const rowSel = bot.select('g.bot-bars')
      .selectAll('g.row')
      .data(mix, d => d.panel)
      .join(
        enter => {
          const g = enter.append('g').attr('class', 'row');
          g.append('rect').attr('class', 'panel-bar')
            .attr('x', xBot(0))
            .attr('width', 0)
            .attr('fill', d => colors[d.panel]);
          g.append('text').attr('class', 'panel-name annotation')
            .attr('x', m.left - 8)
            .attr('text-anchor', 'end')
            .attr('font-size', 12)
            .attr('font-weight', 600)
            .attr('fill', '#14110d')
            .text(d => d.panel);
          g.append('text').attr('class', 'panel-pct annotation')
            .attr('font-size', 12)
            .attr('font-weight', 700)
            .attr('fill', '#14110d');
          return g;
        },
        update => update,
      );

    // Update children — short transitions so bars track the scrolling brush smoothly
    const tDur = 220;
    rowSel.select('rect.panel-bar')
      .attr('y', d => yBot(d.panel))
      .attr('height', yBot.bandwidth())
      .transition().duration(tDur)
      .attr('width', d => xBot(d.pct) - xBot(0));

    rowSel.select('text.panel-name')
      .attr('y', d => yBot(d.panel) + yBot.bandwidth() / 2 + 4);

    rowSel.select('text.panel-pct')
      .attr('y', d => yBot(d.panel) + yBot.bandwidth() / 2 + 4)
      .transition().duration(tDur)
      .attr('x', d => xBot(d.pct) + 6)
      .tween('text', function(d) {
        const node = this;
        return () => {
          node.textContent = `${d.pct.toFixed(1)}% · ${d.count.toLocaleString()}`;
        };
      });

    // Bottom titles
    bot.select('text.bot-title')
      .attr('font-size', 12)
      .attr('font-weight', 700)
      .attr('fill', '#14110d')
      .attr('x', m.left - 4).attr('y', -22)
      .text(`Panel mix · ${yearMin}–${yearMax}`);

    bot.select('text.bot-subtitle')
      .attr('font-size', 10)
      .attr('fill', '#8b857d')
      .attr('x', m.left - 4).attr('y', -6)
      .text(brushStart != null ? '' : 'Showing all years.');
  }
</script>

<div class="chart">
  <svg bind:this={svgEl}></svg>
  {#if brushYearRange}
    <div class="brush-readout">
      <strong>{brushYearRange[0]}–{brushYearRange[1]}</strong>
      &nbsp;·&nbsp; {rawRows.filter(r => r.year >= brushYearRange[0] && r.year <= brushYearRange[1]).length.toLocaleString()} devices
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
  .brush-readout {
    position: absolute;
    top: 8px;
    right: 12px;
    background: var(--paper);
    border: 1px solid var(--rule);
    border-radius: 3px;
    padding: 0.4rem 0.6rem;
    font-family: var(--sans);
    font-size: 0.78rem;
    color: var(--ink-soft);
    box-shadow: 0 2px 6px rgba(0,0,0,0.05);
    pointer-events: none;
  }
  .brush-readout strong {
    color: var(--accent);
    font-weight: 700;
  }
</style>
