<script>
  export let width = 100; // default value if none passed
  export let height = 100;
  export let style = ""; // optional style parameter
  export let chartScale = 0.8; // Control chart size (0.7 = 70% of original size)
  
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
    
    // Handle Datawrapper iframe height adjustments
    window.addEventListener("message", (event) => {
      if (event.data["datawrapper-height"]) {
        const chartIframe = document.getElementById("datawrapper-chart-1E9iq");
        if (chartIframe && chartIframe.contentWindow === event.source) {
          for (let chartId in event.data["datawrapper-height"]) {
            const newHeight = event.data["datawrapper-height"][chartId] + "px";
            chartIframe.style.height = newHeight;
          }
        }
      }
    });
  });
</script>

<body>
  <div class="content-container">
    <div class="text-left">
      <div class="text-left-inner">
      <div class="header-section">
        <div class="number">#4</div>
        <div> <span class="title-highlight">APPEARING IN COURT</span></div>
      </div>
      
      <div class="text-section">
        <div class="body-text">
          <p>
          <span class="highlight">Without an attorney, it's much harder to win your case</span> and avoid eviction — 
          but it’s still crucial to show up for your court date. Otherwise, your landlord may be allowed to evict you automatically.
          </p>
          <p>
          Evictions often happen for reasons that have little to do with the tenant’s fault. This is why 
          <span class="highlight">groups in Boston work hard to offer free legal support</span> for people facing evictions.
          </p>
          <p class="footnote">
            *Check out the City of Boston’s 
            <a href="https://www.boston.gov/departments/housing/office-housing-stability/help-tenants-facing-eviction">Office of Housing Stability legal aid program</a>, 
            Harvard Law School’s <a href="https://legalservicescenter.org/get-legal-help/housing-law-unit/">Housing Law Clinic</a>, 
            <a href="https://www.masslegalhelp.org/housing-apartments-shelter/eviction">Mass Legal Help</a> and 
            <a href="https://www.gbls.org/MADE">Greater Boston Legal Services</a> also provide resources for tenants to navigate the process.
          </p>
        </div>
      </div>
      </div>
    </div>

    <div class="chart-right">
      <div class="chart">
        <p>Pie Chart: Tenant Attorneys</p>
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