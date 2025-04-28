<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  // Calculate scaling based on provided dimensions
  let scaleX = 1;
  let scaleY = 1;
  
  $: {
    // Base dimensions for a 2:1 rectangle (width is twice the height)
    const baseWidth = 1600;
    const baseHeight = 800;
    scaleX = width / baseWidth;
    scaleY = height / baseHeight;
  }
 </script>
 
 <main style="transform: scale({scaleX}, {scaleY}); transform-origin: top left; {style}">
   <div class="header-section">
     <div class="number">#1</div>
     <div class="title-box">NOTICE TO QUIT</div>
   </div>
   
   <div class="layout-container">
     <!-- Notice text section -->
     <div class="notice-container">
       <div class="notice-text">
         You've received a notice to quit from your landlord.<br/>
         This tells you the reason they're seeking to evict you and how long you have to respond<br/>
         <br/>
         Not all eviction filings that happen in Boston are for failure to pay rent, in fact, many filings are what's called <span class="highlight">"No Fault"</span> evictions.<br/>
         <br/>
         <span class="highlight">"No Fault"</span> means the landlord is asking the tenant to move out without claiming that the lease was violated. This could be because the owner is selling the property, they want to renovate it, or something else.<span class="highlight">"Cause"</span> means the landlord claims there was a violation of the lease.
       </div>
     </div>
     
     <!-- Rectangle positioned to the right of the text -->
     <div class="rectangle"></div>
   </div>
 </main>
 
 <style>
   @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
   
   main {
     position: relative;
     width: 1600px; /* Base width for 2:1 ratio */
     height: 800px; /* Base height for 2:1 ratio */
     padding: 0;
     overflow: hidden;
     pointer-events: all;
   }
   
   /* Layout container with more width */
   .layout-container {
     position: absolute;
     top: 50%;
     left: 70px;
     width: 1460px; /* Increased width to fill the wider container */
     transform: translateY(-50%);
     display: flex;
     flex-direction: row;
     justify-content: space-between;
     align-items: center;
     gap: 100px; /* Increased space between text and rectangle */
   }
   
   .header-section {
     position: absolute;
     top: 80px;
     left: 70px;
     display: flex;
     align-items: center;
     z-index: 20;
   }
   
   .number {
     font-family: 'Bebas Neue', sans-serif;
     font-size: 60px;
     font-weight: bold;
     margin-right: 15px;
     line-height: 1;
     color: #14110F;
   }
   
   .title-box {
     background-color: #000000;
     color: white;
     border-radius: 12px;
     padding: 8px 20px;
     font-family: 'Bebas Neue', sans-serif;
     font-size: 48px;
     letter-spacing: 1px;
     display: inline-block;
     height: 60px;
     line-height: 60px;
   }
   
   .notice-container {
     position: relative;
     width: 700px; /* Increased width for the wider layout */
     height: 147px;
     background-color: transparent;
     display: flex;
     align-items: center;
   }
   
   .notice-text {
     width: 100%;
     flex-shrink: 0;
     color: #14110F;
     font-size: 20px;
     font-family: 'Geist Mono', monospace;
     font-weight: 400;
     word-wrap: break-word;
     line-height: 1.5;
   }
   
   .highlight {
     background-color: #06D6A0;
     padding: 0 4px;
     border-radius: 3px;
   }
   
   .rectangle {
     position: relative;
     width: 600px; /* Increased width for larger rectangle */
     height: 400px;
     background-color: #000000;
     border-radius: 12px;
   }
 </style>
