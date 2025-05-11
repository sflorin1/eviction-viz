<!-- TenantResources.svelte -->
<script>
    // Props for customization
    export let title = "TENANT RESOURCES & SUPPORT";
    export let cards = [
      {
        type: 1,
        title: "ADVOCACY ORGANIZATIONS",
        description: "Organizations that provide community support, policy advocacy, and organizing assistance for tenants facing housing insecurity.",
        organizations: [
          {
            name: "City Life/Vida Urbana (CLVU)",
            description: "A bilingual, community-led housing justice organization fighting evictions and displacement.",
            url: "https://www.clvu.org"
          },
          {
            name: "Massachusetts Law Reform Institute (MLRI)",
            description: "Statewide nonprofit focused on systemic advocacy, policy reform, and support for legal services providers.",
            url: "https://www.mlri.org"
          },
          {
            name: "Equal Justice Coalition",
            description: "Advocates for expanded access to civil legal aid for Massachusetts residents with low incomes.",
            url: "https://equaljusticecoalition.org"
          }
        ]
      },
      {
        type: 2,
        title: "LEGAL AID ORGANIZATIONS",
        description: "Free or low-cost legal representation and advice for tenants facing eviction or housing discrimination.",
        organizations: [
          {
            name: "Greater Boston Legal Services (GBLS)",
            description: "Provides free legal assistance to low-income residents, including representation in eviction cases.",
            url: "https://www.gbls.org"
          },
          {
            name: "Harvard Legal Aid Bureau",
            description: "A student-run organization at Harvard Law School that offers free legal services to low-income residents of Greater Boston, especially in housing court.",
            url: "https://www.harvardlegalaid.org"
          },
          {
            name: "Volunteer Lawyers Project (VLP)",
            description: "Offers free legal assistance to low-income tenants, especially those in Boston Housing Court.",
            url: "https://vlpnet.org"
          }
        ]
      },
      {
        type: 3,
        title: "GOVERNMENT PROGRAMS",
        description: "Official resources and programs that provide assistance, housing stability services, and funding for tenants.",
        organizations: [
          {
            name: "Office of Housing Stability",
            description: "Provides direct support, housing search assistance, and resources for tenants facing eviction or homelessness.",
            url: "https://www.boston.gov/departments/housing/office-housing-stability"
          },
          {
            name: "Massachusetts Legal Assistance Corporation",
            description: "Funds civil legal aid programs statewide, including eviction defense.",
            url: "https://www.mlac.org"
          }
        ]
      }
    ];
  </script>
  
  <div class="container">
    <div class="title-container">
      <h1>{title}</h1>
    </div>
    
    <div class="cards-container">
      {#each cards as card}
        <div class="resource-card card-{card.type}">
          <div class="card-content">
            <div class="header-section">
              <div class="title-highlight title-{card.type}">{card.title}</div>
            </div>
            
            <div class="card-description">
              {card.description}
            </div>
            
            <div class="organizations-grid">
              {#each card.organizations as org}
                <div class="organization-item">
                  <div class="organization-name" style="text-decoration: underline; text-decoration-color: {card.type === 1 ? '#FE4530' : card.type === 2 ? '#06D6A0' : '#636EF9'}; text-decoration-thickness: 2px;">{org.name}</div>
                  <div class="organization-description">
                    {org.description}
                  </div>
                  <a href={org.url} class="visit-button button-{card.type}" target="_blank">
                    Visit Website
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
                      <path d="M7 17L17 7M17 7H7M17 7V17" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </a>
                </div>
              {/each}
            </div>
          </div>
        </div>
      {/each}
    </div>
  </div>
  
  <style>
    /* Global styles */
    :global(body) {
      font-family: 'Inter', sans-serif;
      background-color: #FCE9E0;
      margin: 0;
      padding: 20px;
      color: #14110F;
    }
    
    .container {
      max-width: 1200px;
      margin: 0 auto;
      padding: 20px;
    }
    
    h1 {
      font-family: 'Bebas Neue', sans-serif;
      font-size: 48px;
      text-align: center;
      letter-spacing: 1.5px;
      color: white;
      background-color: #000000;
      padding: 10px 25px;
      border-radius: 10px;
      display: inline-block;
      transform: rotate(-1deg);
      box-shadow: 3px 3px 0 rgba(0,0,0,0.2);
      transition: transform 0.3s ease;
      margin: 30px auto 40px;
    }
    
    h1:hover {
      transform: rotate(0deg) scale(1.02);
    }
    
    .title-container {
      text-align: center;
      margin-bottom: 30px;
    }
    
    .cards-container {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
      gap: 30px;
      margin-bottom: 40px;
    }
    
    .resource-card {
      position: relative;
      background-color: white;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
      transition: transform 0.3s ease, box-shadow 0.3s ease;
      display: flex;
      flex-direction: column;
      height: 100%;
    }
    
    .resource-card:hover {
      transform: translateY(-5px) scale(1.01);
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.12);
    }
    
    .card-1 {
      border-top: 8px solid #FE4530;
    }
    
    .card-2 {
      border-top: 8px solid #06D6A0;
    }
    
    .card-3 {
      border-top: 8px solid #636EF9;
    }
    
    .card-content {
      padding: 30px;
      position: relative;
      z-index: 1;
    }
    
    .header-section {
      display: flex;
      flex-direction: column;
      align-items: center;
      text-align: center;
      gap: 10px;
      margin-bottom: 25px;
      padding-top: 10px;
    }
    
    .title-highlight {
      font-family: 'Bebas Neue', sans-serif;
      font-size: 32px;
      padding: 8px 20px;
      border-radius: 10px;
      letter-spacing: 1.3px;
      width: 90%;
      text-align: center;
      box-shadow: 2px 2px 0 rgba(0,0,0,0.1);
      transition: transform 0.2s ease, box-shadow 0.2s ease;
    }
    
    .title-highlight:hover {
      transform: translateY(-2px);
      box-shadow: 3px 3px 0 rgba(0,0,0,0.15);
    }
    
    .title-1 {
      background-color: #FE4530;
      color: white;
    }
    
    .title-2 {
      background-color: #06D6A0;
      color: #14110F;
    }
    
    .title-3 {
      background-color: #636EF9;
      color: white;
    }
    
    .card-description {
      font-size: 17px;
      line-height: 1.6;
      margin-bottom: 30px;
      color: #555;
      text-align: center;
      padding: 0 15px;
    }
    
    .organizations-grid {
      display: flex;
      flex-direction: column;
      gap: 15px;
      padding: 0 15px;
      flex-grow: 1;
    }
    
    .organization-item {
      background-color: rgba(250, 250, 250, 0.8);
      padding: 20px;
      border-radius: 12px;
      transition: all 0.2s ease;
    }
    
    .organization-item:hover {
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
    }
    
    .organization-name {
      font-weight: 600;
      font-size: 20px;
      margin-bottom: 10px;
    }
    
    .organization-description {
      font-size: 16px;
      line-height: 1.5;
      color: #555;
      margin-bottom: 15px;
    }
    
    .visit-button {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      font-weight: 500;
      font-size: 15px;
      padding: 8px 16px;
      border-radius: 8px;
      text-decoration: none;
      transition: all 0.2s ease;
    }
    
    .button-1 {
      background-color: rgba(254, 69, 48, 0.1);
      color: #FE4530;
    }
    
    .button-1:hover {
      background-color: rgba(254, 69, 48, 0.2);
    }
    
    .button-2 {
      background-color: rgba(6, 214, 160, 0.1);
      color: #06D6A0;
    }
    
    .button-2:hover {
      background-color: rgba(6, 214, 160, 0.2);
    }
    
    .button-3 {
      background-color: rgba(99, 110, 249, 0.1);
      color: #636EF9;
    }
    
    .button-3:hover {
      background-color: rgba(99, 110, 249, 0.2);
    }
    
    .footer {
      text-align: center;
      margin-top: 30px;
      color: #777;
      font-size: 15px;
    }
    
    @media (max-width: 768px) {
      .cards-container {
        grid-template-columns: 1fr;
      }
      
      .title-highlight {
        font-size: 28px;
        width: 90%;
      }
      
      h1 {
        font-size: 36px;
        padding: 8px 20px;
      }
    }
  </style>