<!-- HiddenImpact.svelte -->
<script>
    // Props for customization
    export let title = "HIDDEN IMPACT";
    export let leftStat = "NOTICE TO QUIT ≠ LEGAL EVICTION";
    export let rightStat = "LONG-TERM CONSEQUENCES";
    </script>
    
    <div class="hidden-impact-container">
      <div class="hidden-impact-text-left">
        <div class="hidden-impact-header-section">
          <div><span class="hidden-impact-title-highlight">{title}</span></div>
        </div>
        <div class="hidden-impact-body-text">
          <span class="hidden-impact-key-stat">{leftStat}</span>
          <slot name="left-content">
            <p>
              This process has been pretty confusing. Many tenants understandably think that they must move
              out when they receive a Notice to Quit, even though they cannot be legally evicted without a court ruling.
            </p>
            <p>
              Because of this, <span class="hidden-impact-highlight">we will never truly know how many people are displaced from their homes without due process in court.</span>
            </p>
          </slot>
        </div>
      </div>
      <div class="hidden-impact-chart-right">
        <div class="hidden-impact-header-section" style="opacity: 0;">
          <div><span class="hidden-impact-title-highlight">{title}</span></div>
        </div>
        <div class="hidden-impact-body-text">
          <span class="hidden-impact-key-stat">{rightStat}</span>
          <slot name="right-content">
            <p>
              Even when tenants are able to successfully fight their evictions in court, many experience lifelong consequences.
            </p>
            <p>
              Studies show that tenants who simply have an eviction <i>filing</i> on their record with no eviction experience longterm housing instability.
            </p>
            <p>
              That's because <span class="hidden-impact-highlight">eviction filings are public records, which are often used to create "blacklists"</span> by landlords and tenant screening services.
            </p>
          </slot>
        </div>
      </div>
    </div>
    
    <style>
      @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
      @import url('https://fonts.googleapis.com/css2?family=Geist+Mono:wght@400;600&display=swap');
      
      /* Remove global body styling to prevent affecting other components */
      /* Font imports above still apply */
      
      .hidden-impact-container {
        width: 90%;
        margin: 0 auto;
        display: grid;
        grid-template-columns: 1fr 1fr;
        column-gap: 3rem;
        padding: 30px 20px;
        background-color: transparent; /* Changed from white to transparent */
        border-radius: 15px;
        box-shadow: 0 5px 15px rgba(0,0,0,0.05);
        font-family: 'Geist Mono', monospace; /* Move font here from global */
      }
      
      .hidden-impact-header-section {
        display: flex;
        align-items: center;
        margin-bottom: 10px;
      }
      
      .hidden-impact-title-highlight {
        background-color: #000000;
        color: white;
        padding: 10px 25px;
        border-radius: 10px;
        font-family: 'Bebas Neue', sans-serif;
        font-size: 48px;
        letter-spacing: 1.5px;
        display: inline-block;
        transform: rotate(-1deg);
        box-shadow: 3px 3px 0 rgba(0,0,0,0.2);
        transition: transform 0.3s ease;
      }
      
      .hidden-impact-title-highlight:hover {
        transform: rotate(0deg) scale(1.02);
      }
      
      .hidden-impact-body-text {
        color: #14110F;
        font-size: 16px;
        font-family: 'Geist Mono', monospace;
        font-weight: 400;
        line-height: 1.8;
        margin-top: 3rem;
      }
      
      /* Changed from global to component-specific */
      .hidden-impact-highlight {
        background-color: #06D6A0;
        padding: 2px 6px;
        border-radius: 3px;
        font-weight: 600;
        font-size: 18px;
        box-shadow: 1px 1px 0 rgba(0,0,0,0.1);
      }
      
      .hidden-impact-key-stat {
        font-family: 'Bebas Neue', sans-serif;
        font-size: 28px;
        display: block;
        margin: 25px 0 15px;
        color: #000;
        padding-left: 12px;
        border-left: 4px solid #06D6A0;
      }
      
      .hidden-impact-section-divider {
        height: 3px;
        background: linear-gradient(90deg, #06D6A0, transparent);
        margin: 30px 0;
        width: 80%;
      }
      
      @media (max-width: 768px) {
        .hidden-impact-container {
          grid-template-columns: 1fr;
          gap: 2rem;
        }
      }
    </style>