<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  
  import { onMount } from 'svelte';
  
  let imageLoaded = false;
  let imgElement;
  let imgWidth, imgHeight;
  
  onMount(() => {
    if (imgElement) {
      imgElement.onload = () => {
        // Get natural dimensions of the image
        imgWidth = imgElement.naturalWidth;
        imgHeight = imgElement.naturalHeight;
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
        <div class="number">#2</div>
        <div> <span class="title-highlight">FILLING IN HOUSING COURT</span></div>
      </div>
      
      <div class="text-section">
        <div class="body-text">
          <p>
            It’s no wonder that <span class="highlight">"Non-Payment"</span> is the most common cause 
            cited in filings. A 2024 analysis found that rapidly rising rents mean that the average Boston 
            renter spends <span class="highlight">47% of their income on housing costs,</span> yet wages in 
            Boston have not increased at the same rate.
          </p>
          <p>
            It’s important to keep in mind that even when an eviction filing claims a tenant is being asked 
            to leave for non-payment or a lease violation, <span class="highlight"> a landlord doesn’t need to prove their claim until 
            the case is heard in court.</span>
          </p>
        </div>
      </div>
      </div>
    </div>

    <div class="chart-right">
      <div class="chart">
        <p>Chart of Eviction Filings by Case Type</p>
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
    display: grid;
    grid-template-rows: subgrid;
    grid-row: span 2;
    grid-column: 1;
    width: 100%;
    height: 100%;
  }

  .chart-right {
    display: grid;
    grid-template-rows: subgrid;
    grid-column: 2;
    grid-row: 1 / span 2;
    align-items: center;
    justify-content: center;
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
