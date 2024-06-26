<svelte:head>
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css" integrity="sha384-AfEj0r4/OFrOo5t7NnNe46zW/tFgW6x/bCJG8FqQCEo3+Aro6EYUG4+cU+KJWu/X" crossorigin="anonymous">
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.4.1/dist/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
</svelte:head>

<script>
  import katexify from '../katexify';

  import { onMount } from 'svelte';
  import { onDestroy } from 'svelte';
  import * as d3 from 'd3';
  import { range } from "d3-array";
  import { line } from "d3-shape";

  let numDataPoints = 1000;
  let allData = [];
  let data = [];
  let bins = [];
  let height = 500;
  let width;
  let windowWidth;

  function gaussian(x, mu, sigma) {
    return (1 / (sigma * Math.sqrt(2 * Math.PI))) * Math.exp(-((x - mu) ** 2) / (2 * sigma ** 2));
  }

  const generateInitialData = () => {
    const randomNormal = d3.randomNormal(0.01, 0.8);
    allData = Array.from({ length: 10000 }, randomNormal);
    updateData();
  };

  const updateData = () => {
    data = allData.slice(0, numDataPoints);
    createHistogram();
  };

  const createHistogram = () => {
    const numBins = Math.max(20, Math.floor(numDataPoints / 100));
    const x = d3.scaleLinear().domain([-3, 3]).range([0, width - width / (bins.length + 1)]);

    const histogram = d3.histogram()
      .domain(x.domain())
      .thresholds(x.ticks(numBins));

    bins = histogram(data);
  };

  onMount(() => {
    generateInitialData();

    windowWidth = window.innerWidth;
    width = Math.min(0.8 * windowWidth, 800);

    const updateWidth = () => {
      windowWidth = window.innerWidth;
      width = Math.min(0.8 * windowWidth, 800);
    };

    window.addEventListener('resize', updateWidth);
  });
</script>

<main>
  <div id="npde" class="section">
    <h2 class="section-header">Histogram Estimators</h2>
    
    <div class="subsection">
      <h4 class="subsection-header">Intuition</h4>

      <p>Histogram estimators are a nonparametric density estimation technique.
      Data is split up into non-overlapping bins and the proportion of data points
      in each bin is calculated. These proportions are then normalized (scaled down)
      so that the resulting histogram is a valid density function.</p>

      <p>When estimating the density at a specific value, the height of the bin
      containing that value is used. The formula for the height is given by:</p>

      {@html katexify("f(x)=\\frac{\\text{points in bin containing x}}{\\text{total points}\\times\\text{bin size}}", true)}
    </div>

    <div class="subsection">
      <h4 class="subsection-header">Histogram Simulation</h4>

      <p>The following visualization depicts how histogram estimators can be incredibly
      close to the true distribution as the number of data points and bins increases.</p>

      <div class="graph-container">
        <svg width=width-10 height="501">
          {#each bins as bin}
            <g transform={`translate(${d3.scaleLinear().domain([-3, 3]).range([0, width - width / (bins.length + 1)])(bin.x0)},${d3.scaleLinear().domain([0, d3.max(bins, d => d.length / (numDataPoints * bin.length))]).range([500, 0])(bin.length / (numDataPoints * bin.length))})`} class="bar">
              <rect width={width / (bins.length) + 1} height={width - d3.scaleLinear().domain([0, d3.max(bins, d => d.length / (numDataPoints * bin.length))]).range([500, 0])(bin.length / (numDataPoints * bin.length))} />
            </g>
            <line x1="0" y1="500" x2="800" y2="500" stroke="#000000" stroke-opacity="1" stroke-width="1"/>
          {/each}
          <path d="{line().x(d => (d + 4) * 100).y(d => 500 - gaussian(d, 0, 1) * 1150)(range(-10, 10, 0.01))}" fill="none" stroke="black" stroke-width="3" />
          <text x="600" y="100" fill="black" font-size="18" font-family="Inter" text-anchor="middle">True Density</text>

          <defs>
            <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="0" refY="3.5" orient="auto">
              <polygon points="0 0, 3.5 3.5, 0 7" />
            </marker>
          </defs>
          <path d="M600,110 C580,165 560,190 510,200" stroke="black" stroke-width="2" fill="none" marker-end="url(#arrowhead)" />
        </svg>
        <div class="input-container">
          <label>
            <div>Number of Data Points & Bins:</div>
            <input type="range" bind:value={numDataPoints} min="1000" max="10000" step="1000" on:input={updateData} />
          </label>
        </div>
      </div>
    </div>

    <div class="subsection">
      <h4 class="subsection-header">Pros & Cons of Histogram Estimators</h4>

      <p>The main advantages of using a histogram estimator is that they are
      very easy to implement and don't make any assumptions
      about the underlying distribution of the data.</p>

      <p>However, histogram estimators also have a few key disadvantages. Firstly, they
      produce a discontinuous density estimate which may be inaccurate. Additionally, as seen 
      in the simulation, a good estimate requires a lot of data. Ideally, in practice, every
      bin will have atleast one data point. In low dimensions, this is feasible;
      however, once you start working with high-dimensional data, the number of bins
      increases exponentially.</p>
    </div>
  </div>
</main>

<style>
  @import 'styles.css';

  .bar {
    fill: #2980b9;
  }

  .input-container {
    margin-top: 25px;
    margin-bottom: -15px;
  }

  input[type="range"] {
    width: 300px;
    accent-color: #2980b9;
  }

  input[type="range"]:focus {
    accent-color: #2980b9;
  }
</style>