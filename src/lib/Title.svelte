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
  
  <main style="transform: scale({scaleX}, {scaleY}); transform-origin: top left; {style}">
    <div class="desktop">
      <div class="boston-serial-evictors">
        <span class="title-text">BOSTON SERIAL EVICTORS</span>
      </div>
    </div>
  </main>
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Coda+Caption:wght@800&display=swap');
    
    main {
      position: relative;
      width: 1600px; /* Base width */
      height: 800px; /* Base height */
      padding: 0;
      overflow: hidden;
      pointer-events: all;
    }
    
    .title-text {
      color: black;
      font-size: 90px;
      font-family: 'Coda Caption', sans-serif;
      font-weight: 800;
      word-wrap: break-word;
    }
    
    .boston-serial-evictors {
      left: 176px;
      top: 420px;
      position: absolute;
    }
    
    .desktop {
      width: 100%;
      height: 100%;
      position: center;
      overflow: hidden;
    }
  </style>