---
toc: false
---

<div class="hero">
  <h1>🇨🇦 Canada Economic Data <span style="color: var(--theme-foreground)">Visualization</span></h1>
  <h2>British Columbia regional economic indicators and employment trends</h2>
</div>

<div class="grid grid-cols-3" style="margin: 2rem 0;">
  <div class="card" style="padding: 1.5rem;">
    <h3 style="margin-top: 0;">Clean Technology Sector - BC</h3>
    <p style="font-size: 0.9rem; color: var(--theme-foreground-muted);">Employment and compensation trends in environmental and clean technology products across British Columbia (2012-2023)</p>
    <a href="./clean-technology-bc" style="display: inline-block; margin-top: 0.5rem;">View Dashboard →</a>
  </div>
  <div class="card" style="padding: 1.5rem;">
    <h3 style="margin-top: 0;">Vancouver Island & Coast</h3>
    <p style="font-size: 0.9rem; color: var(--theme-foreground-muted);">Regional unemployment rate trends compared to BC and Canada (Aug-Dec 2024)</p>
    <a href="./vancouver-island-economy" style="display: inline-block; margin-top: 0.5rem;">View Dashboard →</a>
  </div>
  <div class="card" style="padding: 1.5rem;">
    <h3 style="margin-top: 0;">BC & Vancouver Demographics</h3>
    <p style="font-size: 0.9rem; color: var(--theme-foreground-muted);">2021 Census data comparing population, age distribution, housing types, and income between BC and Vancouver</p>
    <a href="./bc-demographics" style="display: inline-block; margin-top: 0.5rem;">View Dashboard →</a>
  </div>
</div>

---

## About This Project

This visualization project provides dashboards for analyzing economic and demographic data from British Columbia, Canada. The data is sourced from Statistics Canada and focuses on employment trends, clean technology sectors, regional unemployment rates, and census demographics.

### Data Sources

- **Statistics Canada Table 36-10-0681-01**: Environmental and clean technology products economic account - Employment and compensation per products category
- **Statistics Canada Table 14-10-0387-01**: Labour force characteristics by economic region, three-month moving average
- **Statistics Canada 2021 Census of Population**: Demographic, housing, and income characteristics for British Columbia and Vancouver

### Key Features

- **Comparative Analysis**: View regional data compared to provincial and national averages
- **Demographic Insights**: Compare population, age distribution, housing, and income between BC and Vancouver

<style>

.hero {
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: var(--sans-serif);
  margin: 4rem 0 8rem;
  text-wrap: balance;
  text-align: center;
}

.hero h1 {
  margin: 1rem 0;
  padding: 1rem 0;
  max-width: none;
  font-size: 14vw;
  font-weight: 900;
  line-height: 1;
  color: var(--theme-foreground-focus);
}

.hero h2 {
  margin: 0;
  max-width: 34em;
  font-size: 20px;
  font-style: initial;
  font-weight: 500;
  line-height: 1.5;
  color: var(--theme-foreground-muted);
}

@media (min-width: 640px) {
  .hero h1 {
    font-size: 56px;
  }
}

</style>
