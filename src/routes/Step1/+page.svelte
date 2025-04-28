<script>
    // Svelte component script section
   </script>
   <main>
   <div class="header-section">
   <div class="number">#1</div>
   <div class="title-box">NOTICE TO QUIT</div>
   </div>
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
   <!-- Moved rectangle outside the notice container for more independent positioning -->
   <div class="rectangle"></div>
   </main>
   <style>
   @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap');
   main {
   position: relative;
   height: 100vh;
   background-color: #FCE9E0;
   padding: 0;
    }
   .header-section {
   position: absolute;
   top: 80px;
   left: 70px;
   display: flex;
   align-items: center;
    }
<<<<<<< Updated upstream
    
    .number {
      font-family: 'Bebas Neue', sans-serif;
      font-size: 60px;
      font-weight: bold;
      margin-right: 20px;
      line-height: 1;
      color: #14110F;
=======
   .number {
   font-family: 'Bebas Neue', sans-serif;
   font-size: 60px;
   font-weight: bold;
   margin-right: 15px;
   line-height: 1;
   color: #14110F;
>>>>>>> Stashed changes
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
   position: absolute;
   top: 50%;
   left: 70px;
   width: 700px;
   height: 147px;
   background-color: transparent;
   display: flex;
   align-items: center;
   transform: translateY(-50%);
    }
   .notice-text {
   width: 700px;
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
   position: absolute;
   top: 50%;
   right: 70px; /* Position from right side instead of left */
   width: 750px; /* Reduced width from 700px */
   height: 500px;
   background-color: #000000; /* Black rectangle */
   border-radius: 12px;
   transform: translateY(-50%); /* Center vertically */
    }
   </style>