<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'

  import { onMount } from "svelte";
  import * as d3 from "d3";

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


  onMount(() => {
    drawDiamond();
  });

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

    // Add a group element for the hover text box
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

    // Draw bases (white diamonds with black outlines)
    const bases = [
      { x: fieldWidth / 2, y: fieldHeight / 4, label: "Double %: " },
      { x: 3 * fieldWidth / 4, y: fieldHeight / 2, label: "Single %: " },
      { x: fieldWidth / 2, y: 3 * fieldHeight / 4, label: "Home Run %: " },
      { x: fieldWidth / 4, y: fieldHeight / 2, label: "Triple %: " },
    ];

    bases.forEach((base) => {
      svg
        .append("rect")
        .attr("x", base.x - halfBaseSize)
        .attr("y", base.y - halfBaseSize)
        .attr("width", baseSize)
        .attr("height", baseSize)
        .attr("fill", "white")
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
  <h1>Introduction</h1>
  <p class="body-text">
    Baseball is one of the leading sports in the collection and usage of data. Two of the newer metrics which have
    gained popularity are Exit Velocity (EV) and Launch Angle (LA). Exit velocity
    is the speed (in mph) that the ball is travelling directly after contact. Launch angle is the angle
    (in degrees) that the ball is travelling after the initial contact. There is some additional vocabulary
    that will be used throughout this project. The first of which is BBE or batted ball event is any
    time a ball is hit that occurs in an out, a hit, or an error. wOBA or weighted on-base average 
    measures the value of each method of reaching base based on how much it changes projected runs scored.
  </p>
  <h2>Objectives</h2>
  <p class = "body-text">
    The goal of this project is to visualize the relationship between EV, LA, and the outcomes that they 
    produce. In the graphic below, use the sliders to pick a desired EV and LA combination to inspect. If there
    were any recorded instances of that combination during the 2023 season, the box to the side will have a full breakdown
    of the statistics. Additionally you can see the percentage of BBEs that result in each of the bases by hovering over them.
  </p>

  <svg id="diamond"></svg>
  <div class="container">
    <div class="slider-container">
      <label for="exitVelocity">Exit Velocity</label>
      <input type="range" id="exitVelocity" min="12" max="122" bind:value={exitVelocity}>
      <span>{exitVelocity}</span>
      
      <label for="launchAngle">Launch Angle</label>
      <input type="range" id="launchAngle" min="-89" max="89" bind:value={launchAngle}>
      <span>{launchAngle}</span>
    </div>
    <div class="text-box">
      <h>Stats</h>
      <p>Hits: {Hits}</p> 
      <p>BBE: {BBE}</p>
      <p>Avg. Distance (ft): {AvgDist}</p>
      <p>Batting Avg.: {avg}</p>
      <p>wOBA: {wOBA}</p>
      <p>1B: {singles}</p>
      <p>2B: {doubles}</p>
      <p>3B: {triples}</p>
      <p>HR: {homeruns}</p>
    </div>
  </div>

  <section id="conclusion">
  <h1 class="body-header">Final Project Prototype -- Write Up</h1>
    <h2 class="centered" style="margin-bottom: 15px">What we have done so far:</h2>
      <p class="body-text" style="margin-bottom: 30px">
        So far, we have created the webpage, implemented the scrollyteller, and established
        some structure to the webpage. We have cleaned the data so that we can retrieve statistics for each combination of
        exit velocity and launch angle. We created the baseball field, and added sliders below the baseball diamond so that you can
        change the exit velocity and launch angle. We included a brief introduction to the project, along
        with our overall objectives.
      </p>
   
    <h2 class="centered" style="margin-bottom: 15px">What will be the most challenging:</h2>
      <p class='body-text' style="margin-bottom: 30px">
        The most challenging part of our design will probably be connecting the data from
        our Jupyter Notebook to the interactive visualization. Our goal is to create text boxes when you hover over a base which
        show the number of singles, doubles, triples, or home runs for the combination of exit velocity and launch angle. We also
        want to color encode the bases so that it represents the likelihood of a hit getting on that particular base. This will be difficult as we will need to calculate coordinates, and figure out how to change the statistics based on the hovered base.
    </p>
</section>
  
</main>

<style>

  .body-text {
    text-align: justify;
    word-wrap: break-word;
    font-size: 20px;
    line-height: 1.5; 
    margin-top: 20px;
    margin-bottom: 20px;
  }


  h1, h2 {
      text-align: center;
      font-size: 24px;
      font-weight: bold;
      color: #333;
      margin-top: 20px;
      margin-bottom: 20px;
  }


    .container {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    margin-top: 50px;
  }

    .text-box {
    margin-left: 20px;
    background-color: white;
    border: 1px solid black;
    padding: 10px;
    border-radius: 10px;
  }

  svg {
    display: block;
    margin: auto;
    margin-top: 50px;
  }
  
  .slider-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-top: 20px;
  }
  
  .slider-container label {
    margin-top: 10px;
  }
  
  .slider-container input {
    margin: 5px 0;
  }
  
  .slider-container span {
    margin-bottom: 20px;
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


  #conclusion {
    padding: 2rem 1rem;
  }
</style>
