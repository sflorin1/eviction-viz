<script>
    export let width = 100; // default value if none passed
    export let height = 100;
    export let style = ""; // optional style parameter
    
    // Calculate scaling based on provided dimensions
    let scaleX = 1;
    let scaleY = 1;
    
    $: {
      // Base dimensions for the component
      const baseWidth = 1600;
      const baseHeight = 800;
      scaleX = width / baseWidth;
      scaleY = height / baseHeight;
    }
  </script>
  
  <div class="title-container">
    <main style="transform: scale({scaleX}, {scaleY}); transform-origin: top left; {style}">
      <div class="desktop">
        <div class="boston-serial-evictors">
          <span class="title-text">BOSTON'S RENTAL POWER DYNAMICS</span>
          <br>
          <br>
          <span class="team">Team: Displaced by Design</span>
          <br>
          <span class="team">Sam Florin, Sheyla Chevarria, Jazzlynn Derrick, and Alejandra Pardini</span>
        </div>
      </div>
    </main>
  </div>
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Coda+Caption:wght@800&display=swap');

    main {
      display: flex;
      max-width: 90%;
      position: relative;
      width: 1600px; /* Base width */
      height: 800px; /* Base height */
      padding: 0;
      overflow: hidden;
      pointer-events: all;
      text-align: center;
      top: -10%;
    }
    
    .title-text {
      color: black;
      text-align: center;
      font-size: 60px;
      font-family: 'Coda Caption', sans-serif;
      font-weight: 800;
      word-wrap: break-word;
    }

    .team {
      color: #14110F;
      font-size: 14px;
      font-family: 'Geist Mono', monospace;
      font-weight: 400;
      word-wrap: break-word;
      line-height: 1.6;
    }
    
    .boston-serial-evictors {
      left: 176px;
      top: 350px;
      position: absolute;
    }
    
    .desktop {
      width: 100%;
      height: 100%;
      position: center;
      overflow: hidden;
    }
  </style>