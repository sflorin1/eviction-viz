<script>
    export let width = 100; // default value if none passed
    export let height = 100;
    export let style = ""; // optional style parameter
    
    // Import the image to get its dimensions
    import { onMount } from 'svelte';
    
    let imageLoaded = false;
    let imgElement;
    let imgWidth, imgHeight;
    let containerWidth, containerHeight;
    
    onMount(() => {
      if (imgElement) {
        imgElement.onload = () => {
          // Get natural dimensions of the image
          imgWidth = imgElement.naturalWidth;
          imgHeight = imgElement.naturalHeight;
          
          // Calculate container dimensions based on proportional scaling
          const scaleRatio = Math.min(width / imgWidth, height / imgHeight);
          containerWidth = imgWidth * scaleRatio;
          containerHeight = imgHeight * scaleRatio;
          
          imageLoaded = true;
        };
      }
    });
  </script>
  
  <body>
    <div class="content-container">
      <div class="text-left">
            <div class="text-left-inner">
                <div class="header-section">
                    <div> <span class="title-highlight">HIDDEN IMPACT</span></div>
                </div>
                <div class="body-text">
                    <p>
                        This process has been pretty confusing. Many tenants understandably think that they must move 
                        out when they receive a Notice to Quit, even though they cannot be legally evicted without a court ruling. Because of this,
                        <span class="highlight">we will never truly know how many people are displaced from their homes without due process in court.</span> 
                    </p>
                </div>
            </div>
        </div>
  
      <div class="chart-right">
        <div class="header-section">
            <div style="opacity: 0;"> <span class="title-highlight">HIDDEN IMPACT</span></div>
        </div>
        <div class="body-text">
            <p>
                Even when tenants are able to successfully fight their evictions in court, many experience lifelong consequences. 
                Studies show that tenants who simply having an eviction <i>filing</i> on their record with no eviction experience longterm housing instability.
                That's because <span class="highlight">eviction filings are public records, which are often used to create "blacklists"</span> by landlords and tenant screening services.
            </p>
        </div>
      </div>
    </div>
  </body>
  
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
  
    .content-container {
      width: 90%;
      margin-inline: auto;
      display: grid;
      grid-template-columns: 1fr 1fr;
      grid-template-rows: auto;
      column-gap: 1rem;
      max-height: calc(100vh - 320px);
      overflow: hidden;
      padding-top: 15px;
      gap: 3rem;
    }
  
    .text-left {

      width: 100%;
      height: 100%;
    }
  
    .chart-right {
      width: 100%;
      height: 100%;
    }
  
    .chart {
      align-self: center;
      width: 100%;
      height: 100%;
    }
  
    .header-section {
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      gap: 0.5rem;
    }
  
    .number {
      font-family: 'Bebas Neue', sans-serif;
      font-size: 52px;
      font-weight: bold;
      margin-right: 15px;
      line-height: 1;
      color: #14110F;
      width: 1fr;
    }
  
    .body-text {
      color: #14110F;
      font-size: clamp(12px, 3.5vw, 15px);
      font-family: 'Geist Mono', monospace;
      font-weight: 400;
      word-wrap: break-word;
      line-height: 1.6;
      margin-top: 3.5rem;
    }
  
    .footnote {
      color: #14110F;
      font-family: 'Geist Mono', monospace;
      font-size: 11px;
      word-wrap: break-word;
      line-height: 1.7;
    }
  
    a {
      color: #3E3E3D;
    }
  
    .highlight {
      background-color: #06D6A0;
      padding: 0 4px;
      border-radius: 3px;
    }
  
    .title-highlight {
      background-color: #000000;
      color: white;
      padding: 7px 20px;
      border-radius: 10px;
      font-family: 'Bebas Neue', sans-serif;
      font-size: 40px;
      letter-spacing: 1.3px;
    }
  </style>