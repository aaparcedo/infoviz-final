<script>
  import { onMount } from 'svelte';
  import * as d3 from 'd3';

  let { activeStep = 0 } = $props();

  let svgEl = $state(null);
  let width = $state(900);
  const m = { top: 36, right: 28, bottom: 28, left: 28 };

  // 5-bucket aggregation. Sums to 99; one cell remains unfilled.
  const buckets = [
    { key: 'daily', label: 'Daily', count: 13, color: '#FF2A2A',
      sub: 'Several times a day or once a day' },
    { key: 'weekly', label: 'Weekly', count: 30, color: '#FFAB2E',
      sub: 'A few times a week or once a week' },
    { key: 'monthly', label: 'Monthly', count: 9, color: '#FFD27A',
      sub: 'A few times a month' },
    { key: 'rare', label: 'Tried but rarely', count: 19, color: 'rgba(255,171,46,0.35)',
      sub: 'Tried but rarely use' },
    { key: 'never', label: 'Never used', count: 28, color: 'rgba(255,171,46,0.18)',
      sub: 'Have not used an AI companion' },
  ];

  $effect(() => {
    if (svgEl) {
      void activeStep;
      void width;
      render();
    }
  });

  onMount(() => {
    const ro = new ResizeObserver(() => {
      if (!svgEl) return;
      width = svgEl.parentElement.clientWidth;
    });
    if (svgEl) ro.observe(svgEl.parentElement);
    width = svgEl ? svgEl.parentElement.clientWidth : 900;
    render();
  });

  function getCellsForStep(step) {
    // Returns 100 cells with their { bucket, color } based on step
    const cells = [];
    // Default arrangement (matches buckets order)
    const fullCells = [];
    for (const b of buckets) {
      for (let i = 0; i < b.count; i++) fullCells.push(b);
    }
    while (fullCells.length < 100) fullCells.push(null);

    if (step === 0 || step === 4) return fullCells;

    if (step === 1) {
      // Show "have used" (72) coral, "never" gray, "tried-rarely" muted
      // Daily/Weekly/Monthly = 52 → coral; Rare = 19 → muted; Never = 28 → very pale
      return fullCells.map(c => {
        if (c === null) return null;
        if (c.key === 'never') return { ...c, color: 'rgba(255,171,46,0.18)' };
        if (c.key === 'rare') return { ...c, color: 'rgba(255,171,46,0.35)' };
        return { ...c, color: '#FFAB2E' };  // collapse daily/weekly/monthly to one coral
      });
    }

    if (step === 2) {
      // Show full 5-bucket coloring
      return fullCells;
    }

    if (step === 3) {
      // Highlight 24 cells: "1 in 3 of users chose AI over person" = ~24% of all teens
      // We'll color the first 24 cells of "ever used" subset (daily+weekly+monthly+rare = 71 cells)
      // as a deeper coral, others gray
      let highlightedCount = 0;
      return fullCells.map(c => {
        if (c === null) return null;
        if (c.key === 'never') return { ...c, color: 'rgba(201,168,76,0.30)' };
        if (highlightedCount < 24) {
          highlightedCount++;
          return { ...c, color: '#FF2A2A' };
        }
        return { ...c, color: 'rgba(201,168,76,0.4)' };
      });
    }

    return fullCells;
  }

  function render() {
    if (!svgEl) return;
    const cells = getCellsForStep(activeStep);

    const cols = 10, rows = 10;
    const cellGap = 3;
    const usableWidth = width - m.left - m.right;
    const gridWidth = Math.min(usableWidth * 0.5, 360);
    const cell = (gridWidth - cellGap * (cols - 1)) / cols;
    const gridHeight = cell * rows + cellGap * (rows - 1);
    const height = m.top + gridHeight + m.bottom + 30;

    const svg = d3.select(svgEl)
      .attr('viewBox', `0 0 ${width} ${height}`)
      .attr('width', width)
      .attr('height', height);

    let root = svg.select('g.root');
    if (root.empty()) {
      svg.selectAll('*').remove();
      root = svg.append('g').attr('class', 'root');
      root.append('g').attr('class', 'grid').attr('transform', `translate(${m.left}, ${m.top})`);
      root.append('g').attr('class', 'legend');
      root.append('text').attr('class', 'chart-title label');
      root.append('g').attr('class', 'callout');
    }

    // Cells
    root.select('g.grid')
      .selectAll('rect')
      .data(cells)
      .join(
        enter => enter.append('rect')
          .attr('x', (_, i) => (i % cols) * (cell + cellGap))
          .attr('y', (_, i) => Math.floor(i / cols) * (cell + cellGap))
          .attr('width', cell)
          .attr('height', cell)
          .attr('fill', '#0A0A0F')
          .attr('stroke', 'rgba(201,168,76,0.30)')
          .attr('stroke-width', 0.6),
        update => update,
      )
      .transition().duration(600)
      .attr('x', (_, i) => (i % cols) * (cell + cellGap))
      .attr('y', (_, i) => Math.floor(i / cols) * (cell + cellGap))
      .attr('width', cell)
      .attr('height', cell)
      .attr('fill', d => d ? d.color : '#ffffff')
      .attr('stroke', d => d ? 'none' : 'rgba(201,168,76,0.30)');

    // Legend on right (only in steps 0, 2)
    const legendX = m.left + gridWidth + 28;
    const legendRight = width - m.right;
    const legendW = legendRight - legendX;

    const legend = root.select('g.legend')
      .attr('transform', `translate(0, ${m.top})`);

    legend.selectAll('*').remove();

    if (legendW > 140 && (activeStep === 0 || activeStep === 2 || activeStep === 4)) {
      const legY0 = 4;
      const lineH = 32;
      buckets.forEach((b, i) => {
        const ly = legY0 + i * lineH;
        legend.append('rect')
          .attr('x', legendX).attr('y', ly).attr('width', 14).attr('height', 14)
          .attr('fill', b.color);
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX + 22).attr('y', ly + 11)
          .attr('font-size', 12).attr('font-weight', 600)
          .attr('fill', '#FFD27A')
          .text(b.label);
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendRight).attr('y', ly + 11)
          .attr('text-anchor', 'end')
          .attr('font-size', 13).attr('font-weight', 700)
          .attr('fill', '#FFAB2E')
          .text(`${b.count}%`);
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX + 22).attr('y', ly + 24)
          .attr('font-size', 10).attr('fill', '#C9A84C')
          .text(b.sub);
      });
    }

    // Step-specific callout (replaces legend in steps 1, 3)
    if (legendW > 140 && (activeStep === 1 || activeStep === 3)) {
      const callY = m.top + 4;
      if (activeStep === 1) {
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY)
          .attr('font-size', 36).attr('font-weight', 700).attr('fill', '#FFAB2E')
          .text('72%');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 26)
          .attr('font-size', 14).attr('font-weight', 600).attr('fill', '#FFD27A')
          .text('have ever used');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 44)
          .attr('font-size', 14).attr('fill', '#FFAB2E')
          .text('an AI companion');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 70)
          .attr('font-size', 12).attr('fill', '#C9A84C')
          .text('Coral cells = ever used.');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 86)
          .attr('font-size', 12).attr('fill', '#C9A84C')
          .text('Gray = never used.');
      }
      if (activeStep === 3) {
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY)
          .attr('font-size', 36).attr('font-weight', 700).attr('fill', '#FF2A2A')
          .text('1 in 3');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 26)
          .attr('font-size', 14).attr('font-weight', 600).attr('fill', '#FFD27A')
          .text('teen users have chosen AI');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 44)
          .attr('font-size', 14).attr('fill', '#FFAB2E')
          .text('over a person for serious talks');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 70)
          .attr('font-size', 12).attr('fill', '#C9A84C')
          .text('Highlighted: 24% of all teens');
        legend.append('text')
          .attr('class', 'annotation')
          .attr('x', legendX).attr('y', callY + 86)
          .attr('font-size', 12).attr('fill', '#C9A84C')
          .text('(33% × 72% = ~24%).');
      }
    }

    // Title above grid
    root.select('text.chart-title')
      .attr('x', m.left).attr('y', m.top - 14)
      .attr('font-size', 11).attr('fill', '#C9A84C')
      .text('Each square = 1 percent of U.S. teens 13–17. n = 1,060.');
  }
</script>

<div class="chart">
  <svg bind:this={svgEl}></svg>
</div>

<style>
  .chart { width: 100%; }
  svg { width: 100%; height: auto; }
</style>
