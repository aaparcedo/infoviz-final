<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';
  import * as topojson from 'topojson-client';

  let { activeStep = 0 } = $props();

  let svgEl = $state(null);
  let width = $state(900);
  const margin = { top: 36, right: 28, bottom: 28, left: 28 };

  let metros = $state([]);
  let usMap = $state(null);
  let hoveredMetro = $state(null);

  $effect(() => {
    if (metros.length && usMap && svgEl) {
      void activeStep;
      render();
    }
  });

  onMount(async () => {
    [metros, usMap] = await Promise.all([
      d3.csv('/data/02-geography.csv', (d) => ({
        metro: d.metro,
        state: d.state_or_region,
        lat: +d.lat,
        lon: +d.lon,
        mw: +d.total_mw_2025,
      })),
      d3.json('https://cdn.jsdelivr.net/npm/us-atlas@3/states-10m.json'),
    ]);

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
    if (!svgEl || metros.length === 0 || !usMap) return;

    const height = Math.max(440, Math.min(560, width * 0.6));
    const svg = d3.select(svgEl)
      .attr('viewBox', `0 0 ${width} ${height}`)
      .attr('width', width)
      .attr('height', height);

    let root = svg.select('g.root');
    if (root.empty()) {
      svg.selectAll('*').remove();
      root = svg.append('g').attr('class', 'root')
        .attr('transform', `translate(${margin.left}, ${margin.top})`);
      root.append('path').attr('class', 'states');
      root.append('path').attr('class', 'nation');
      root.append('g').attr('class', 'bubbles');
      root.append('g').attr('class', 'centers');
      root.append('g').attr('class', 'labels');
    }

    const states = topojson.feature(usMap, usMap.objects.states);
    const nation = topojson.feature(usMap, usMap.objects.nation);

    const projection = d3.geoAlbersUsa()
      .fitSize([width - margin.left - margin.right, height - margin.top - margin.bottom], nation);
    const path = d3.geoPath(projection);

    root.select('path.states')
      .datum(states)
      .attr('fill', 'none')
      .attr('stroke', '#cdc9c0')
      .attr('stroke-width', 0.6)
      .attr('d', path);

    root.select('path.nation')
      .datum(nation)
      .attr('fill', 'none')
      .attr('stroke', '#8b857d')
      .attr('stroke-width', 1)
      .attr('d', path);

    const r = d3.scaleSqrt().domain([0, d3.max(metros, m => m.mw)]).range([0, 60]);
    const positioned = metros
      .map(m => ({ ...m, xy: projection([m.lon, m.lat]) }))
      .filter(m => m.xy);

    // Step-driven emphasis
    // 0: all visible normal
    // 1: highlight NoVA only
    // 2: highlight top 3 (NoVA, Atlanta, DFW)
    // 3: all visible (back to normal, ready for any further detail)
    const top3 = ['Northern Virginia', 'Atlanta', 'Dallas-Fort Worth'];
    const opacityFor = (m) => {
      if (activeStep === 1) return m.metro === 'Northern Virginia' ? 1 : 0.18;
      if (activeStep === 2) return top3.includes(m.metro) ? 1 : 0.2;
      return 1;
    };

    // Bubbles
    root.select('g.bubbles')
      .selectAll('circle')
      .data(positioned, d => d.metro)
      .join(
        enter => enter.append('circle')
          .attr('cx', d => d.xy[0])
          .attr('cy', d => d.xy[1])
          .attr('r', 0)
          .attr('fill', '#d65a3a')
          .attr('fill-opacity', 0.42)
          .attr('stroke', '#d65a3a')
          .attr('stroke-width', 1.4)
          .style('cursor', 'pointer')
          .on('mouseenter', (e, d) => hoveredMetro = d)
          .on('mouseleave', () => hoveredMetro = null),
        update => update,
      )
      .transition().duration(700)
      .attr('cx', d => d.xy[0])
      .attr('cy', d => d.xy[1])
      .attr('r', d => r(d.mw))
      .attr('opacity', d => opacityFor(d));

    // Center dots
    root.select('g.centers')
      .selectAll('circle')
      .data(positioned, d => d.metro)
      .join('circle')
      .attr('cx', d => d.xy[0])
      .attr('cy', d => d.xy[1])
      .attr('r', 1.5)
      .attr('fill', '#9c2e16')
      .transition().duration(700)
      .attr('opacity', d => opacityFor(d));

    // Labels — positioned clearly outside each bubble's radius
    const labelOffsets = {
      'Northern Virginia': { dx: 0,  dy: 76,  anchor: 'middle' },
      'Atlanta':           { dx: 0,  dy: 50,  anchor: 'middle' },
      'Dallas-Fort Worth': { dx: 0,  dy: 46,  anchor: 'middle' },
      'Phoenix':           { dx: 0,  dy: 38,  anchor: 'middle' },
      'Chicago':           { dx: 0,  dy: -34, anchor: 'middle' },
      'Silicon Valley':    { dx: 32, dy: 4,   anchor: 'start'  },
      'Hillsboro':         { dx: 0,  dy: -34, anchor: 'middle' },
    };

    const labelGroups = root.select('g.labels')
      .selectAll('g.lbl')
      .data(positioned, d => d.metro)
      .join(
        enter => {
          const g = enter.append('g').attr('class', 'lbl');
          g.append('text').attr('class', 'metro-name annotation')
            .attr('font-size', 11).attr('font-weight', 700)
            .attr('fill', '#14110d')
            .style('paint-order', 'stroke')
            .attr('stroke', '#fbfaf7')
            .attr('stroke-width', 3.5)
            .attr('stroke-linejoin', 'round')
            .text(d => d.metro);
          g.append('text').attr('class', 'metro-mw annotation')
            .attr('y', 13)
            .attr('font-size', 11).attr('font-weight', 600)
            .attr('fill', '#d65a3a')
            .style('paint-order', 'stroke')
            .attr('stroke', '#fbfaf7')
            .attr('stroke-width', 3.5)
            .attr('stroke-linejoin', 'round')
            .text(d => `${d.mw.toLocaleString()} MW`);
          return g;
        },
        update => update,
      );

    labelGroups
      .attr('transform', d => {
        const off = labelOffsets[d.metro] ?? { dx: 10, dy: -6 };
        return `translate(${d.xy[0] + off.dx}, ${d.xy[1] + off.dy})`;
      });
    labelGroups.selectAll('text')
      .attr('text-anchor', function(_d) {
        const m = d3.select(this.parentNode).datum();
        return labelOffsets[m.metro]?.anchor ?? 'start';
      });

    labelGroups.transition().duration(700)
      .attr('opacity', d => opacityFor(d));

    // Title at top
    root.selectAll('text.chart-title').data([null]).join(
      enter => enter.append('text').attr('class', 'chart-title label')
        .attr('font-size', 11).attr('fill', '#8b857d')
        .text('U.S. data center inventory by metro · CBRE H2 2025'),
    )
    .attr('x', 0).attr('y', -10);
  }
</script>

<div class="chart">
  <svg bind:this={svgEl}></svg>
  {#if hoveredMetro}
    <div class="tooltip">
      <div class="tt-metro">{hoveredMetro.metro}, {hoveredMetro.state}</div>
      <div class="tt-mw">{hoveredMetro.mw.toLocaleString()} MW</div>
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
    min-width: 160px;
  }
  .tt-metro {
    font-weight: 700;
    color: var(--ink);
    margin-bottom: 0.2rem;
  }
  .tt-mw {
    color: var(--accent);
    font-weight: 600;
    font-variant-numeric: tabular-nums;
  }
</style>
