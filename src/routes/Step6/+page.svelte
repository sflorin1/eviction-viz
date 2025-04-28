<script>
    // Svelte component script section
  </script>
  
  <main>
    <div class="notice-container">
      <div class="notice-text">
        You've received a notice to quit from your landlord.<br/>
        This tells you the reason they're seeking to evict you and how long you have to respond
      </div>
    </div>
  </main>
  
  <style>
    main {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background-color: #FAFAFA;
      padding: 1rem;
    }
    
    .notice-container {
      max-width: 800px;
      width: 100%;
      background-color: white;
      border-radius: 8px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      padding: 2rem;
    }
    
    .notice-text {
      width: 100%;
      height: 100%;
      color: #14110F;
      font-size: 20px;
      font-family: 'Geist Mono', monospace;
      font-weight: 400;
      word-wrap: break-word;
      line-height: 1.5;
    }
  </style>