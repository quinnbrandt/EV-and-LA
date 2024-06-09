<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'

  import { onMount } from "svelte";
  import * as d3 from "d3";
  import { csv } from "d3-fetch";

  let exitVelocity = 67;
  let launchAngle = 0;
  let hoverText = "";
  let hoverX = 0;
  let hoverY = 0;

  let Hits = 0;
  let BBE = 0;
  let AvgDist = 0.0;
  let avg = 0.0;
  let wOBA = 0.0;
  let singles = 0;
  let doubles = 0;
  let triples = 0;
  let homeruns = 0;
  let single_per = 0;
  let double_per = 0;
  let triple_per = 0;
  let homer_per = 0;

  let data = [];

  onMount(async () => {
    console.log('onMount called')
    drawDiamond()
    try {
      data = await d3.csv('EV_LA_Data.csv');
      console.log("Data loaded:", data);
      updateStatistics();
    } catch (error) {
      console.error("Error in onMount:", error);
    }
  });

  function updateStatistics() {
    const entry = data.find(d => +d['Exit Velocity'] === exitVelocity && +d['Launch Angle'] === launchAngle);
    if (entry) {
      Hits = +entry['Hits'];
      BBE = +entry['BBE'];
      AvgDist = +entry['Average\nDistance (ft)'];
      avg = +entry['AVG'];
      wOBA = +entry['wOBA'];
      singles = +entry['1B'];
      doubles = +entry['2B'];
      triples = +entry['3B'];
      homeruns = +entry['HR'];
      single_per = +entry['1B %'];
      double_per = +entry['2B %'];
      triple_per = +entry['3B %'];
      homer_per = +entry['HR %'];
      console.log(homer_per)
      console.log(single_per);
      drawDiamond();
    } else {
      Hits = BBE = AvgDist = avg = wOBA = singles = doubles = triples = homeruns = single_per = double_per = triple_per = homer_per = 0;
    }
  }

  function drawDiamond() {
    const width = 500;
    const height = 500;
    const margin = { top: 20, right: 20, bottom: 20, left: 20 };

    const svg = d3
      .select("#diamond")
      .attr("width", width)
      .attr("height", height)
      .append("g")
      .attr("transform", `translate(${margin.left},${margin.top})`);

    const fieldWidth = width - margin.left - margin.right;
    const fieldHeight = height - margin.top - margin.bottom;

    // Draw the green diamond
    svg
      .append("rect")
      .attr("x", fieldWidth / 8)
      .attr("y", fieldHeight / 8)
      .attr("width", (3 * fieldWidth) / 3)
      .attr("height", (3 * fieldHeight) / 3)
      .attr("fill", "green")
      .attr("rx", fieldWidth / 10)
      .attr("ry", fieldHeight / 10)
      .attr("transform", `rotate(225, ${fieldWidth / 2}, ${fieldHeight / 2})`);

    // Draw the tan diamond
    svg
      .append("rect")
      .attr("x", fieldWidth / 4)
      .attr("y", fieldHeight / 4)
      .attr("width", fieldWidth / 2)
      .attr("height", fieldHeight / 2)
      .attr("fill", "tan")
      .attr("transform", `rotate(45, ${fieldWidth / 2}, ${fieldHeight / 2})`);

    const baseSize = fieldWidth / 20;
    const halfBaseSize = baseSize / 2;

    const hoverBox = svg.append("g").style("visibility", "hidden");

    hoverBox
      .append("rect")
      .attr("id", "hoverBox")
      .attr("fill", "white")
      .attr("stroke", "black")
      .attr("rx", 5)
      .attr("ry", 5);

    hoverBox
      .append("text")
      .attr("id", "hoverText")
      .attr("text-anchor", "middle")
      .attr("dx", -10)
      .attr("dy", 12);

    // Draw bases
    const bases = [
      { x: fieldWidth / 2, y: fieldHeight / 4, label: "Double %: " + double_per, percentage: double_per},
      { x: 3 * fieldWidth / 4, y: fieldHeight / 2, label: "Single %: " + single_per, percentage: single_per},
      { x: fieldWidth / 2, y: 3 * fieldHeight / 4, label: "Home Run %: " + homer_per, percentage: homer_per},
      { x: fieldWidth / 4, y: fieldHeight / 2, label: "Triple %: " + triple_per, percentage: triple_per},
    ];

    const logScale = d3.scaleLog()
      .domain([1, 100])  // Adjust the domain based on your data range
      .range([0, 1]);

    bases.forEach((base) => {

      const fillColor = base.percentage === 0 ? "white" : d3.interpolateBlues(logScale(base.percentage));

      svg
        .append("rect")
        .attr("x", base.x - halfBaseSize)
        .attr("y", base.y - halfBaseSize)
        .attr("width", baseSize)
        .attr("height", baseSize)
        .attr("fill", fillColor)
        .attr("stroke", "black")
        .attr("transform", `rotate(45, ${base.x}, ${base.y})`)
        .on("mouseover", (event) => {
          const textWidth = base.label.length * 10; // Approximate text width
          const textHeight = 30; // Box height

          hoverBox
            .attr("transform", `translate(${base.x},${base.y - halfBaseSize - textHeight - 10})`)
            .style("visibility", "visible");

          hoverBox.select("rect")
            .attr("width", textWidth + 20)
            .attr("height", textHeight)
            .attr("x", -(textWidth / 2) - 10)
            .attr("y", -textHeight / 2);

          hoverBox.select("text")
            .attr("x", 0)
            .attr("y", -textHeight / 4)
            .text(base.label);
        })
        .on("mouseout", () => {
          hoverBox.style("visibility", "hidden");
        });
    });
  }
</script>

<main>
  <div class="header-box">
    <h2>Exit Velocity and Launch Angle<br>--  2023 MLB Season  --</h2>
  </div>
  <p class="body-text">
    Baseball is one of the leading sports in the collection and usage of data. Two of the newer metrics which have
    gained popularity are Exit Velocity (EV) and Launch Angle (LA). Exit velocity
    is the speed (in mph) that the ball is traveling directly after contact. Launch angle is the angle
    (in degrees) that the ball is traveling after the initial contact. There is some additional vocabulary
    that will be necessary to understand the project. The first of which is BBE or batted ball event is any
    time a ball is hit that occurs in an out, a hit, or an error. wOBA or weighted on-base average
    measures the value of each method of reaching base based on how much it changes projected runs scored.
  </p>
  <br>
  <hr>
  <div class="header-box">
    <h2>Relationship between<br>Exit Velocity, Launch Angle, and Hits?</h2>
  </div>
  <p class = "body-text">
    The relationship between EV, LA, is very complex because to truly understand the outcomes that they
    produce you must be able to interpret how far and what part of the field the ball would be likely to end up in.
    In the graphic below, use the sliders to pick a desired EV and LA combination to inspect. If there
    were any recorded instances of that combination during the 2023 season, the box to the side will have a full breakdown
    of the statistics. Additionally you can see the percentage of BBEs that result in each of the bases by hovering over them.
  </p>
  <br>
  <br>
  <hr>

  <div class="header-box">
    <h2>EV/LA Simulation</h2>
  </div>
  <svg id="diamond"></svg>
  <div class="container">
    <div class="slider-container">
      <label for="exitVelocity">Exit Velocity</label>
      <input type="range" id="exitVelocity" min="12" max="122" bind:value={exitVelocity} on:change={updateStatistics}>
      <span>{exitVelocity}</span>
      
      <label for="launchAngle">Launch Angle</label>
      <input type="range" id="launchAngle" min="-89" max="89" bind:value={launchAngle} on:change={updateStatistics}>
      <span>{launchAngle}</span>
    </div>
    <div class="text-box">
      <div class="text-line"><strong>Hits:</strong> <span>{Hits}</span></div>
      <div class="text-line"><strong>BBE:</strong> <span>{BBE}</span></div>
      <div class="text-line"><strong>Avg. Distance (ft):</strong> <span>{AvgDist}</span></div>
      <div class="text-line"><strong>Batting Avg.:</strong> <span>{avg}</span></div>
      <div class="text-line"><strong>wOBA:</strong> <span>{wOBA}</span></div>
      <div class="text-line"><strong>1B:</strong> <span>{singles}</span></div>
      <div class="text-line"><strong>2B:</strong> <span>{doubles}</span></div>
      <div class="text-line"><strong>3B:</strong> <span>{triples}</span></div>
      <div class="text-line"><strong>HR:</strong> <span>{homeruns}</span></div>
    </div>
  </div>
  <hr>
  <div class = "footnote">
    <p> This data was sourced from <a href="https://www.example.com">Baseball Savant.</a> </p>
  </div>

</main>

<style>

  .body-text {
    text-align: center;
    word-wrap: break-word;
    font-size: 12px;
    line-height: 1.5; 
    margin-top: 20px;
    max-width: 500px;
  }

  h1, h2 {
      text-align: center;
      font-size: 24px;
      font-weight: bold;
      color: #333;
      margin-top: 20px;
      margin-bottom: 20px;
      max-width: 500px;
  }


  .container {
    display: flex;
    justify-content: center;
    align-items: flex-start;
  }

  .text-box {
    margin-left: 20px;
    background-color: white;
    border: 1px solid black;
    padding: 10px;
    border-radius: 10px;
    font-size: 12px;
    transform: scale(0.8);
    margin-top: -113px;
    column-count: 2;
    column-gap: 20px;
    width: 300px;
    height: 100px;
    white-space: nowrap;
    overflow: hidden;
    transform: scale(0.7);
  }

  svg {
    display: block;
    margin: auto;
    transform: scale(0.55); 
    margin-top: -100px;
  } 
  
  .slider-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: -140px;
    transform: scale(0.7);
  }
  
  .slider-container label {
    margin-top: 10px;
  }
  
  .slider-container input {
    margin: 5px;
  }
  
  .slider-container span {
    margin-bottom: 2px;
  }
  
  .hover-box {
    position: absolute;
    background-color: white;
    border: 1px solid black;
    padding: 20px;
    border-radius: 3px;
    pointer-events: none;
    margin-top: 20px;
  }

  input[type="range"] {
    -webkit-appearance: none;
    appearance: none;
    width: 100%;
    height: 15px;
    background: #ddd;
    outline: none;
    opacity: 0.7;
    transition: opacity .2s;
  }

  input[type="range"]:hover {
    opacity: 1;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 25px;
    height: 25px;
    background: url('./assets/baseball.jpg') no-repeat;
    background-size: contain;
    cursor: pointer;
  }

  input[type="range"]::-moz-range-thumb {
    width: 25px;
    height: 25px;
    background: url('./assets/baseball.jpg') no-repeat;
    background-size: contain;
    cursor: pointer;
  }

  .footnote {
    margin-top: 20px;
    font-size: 8px;
  }

</style>
