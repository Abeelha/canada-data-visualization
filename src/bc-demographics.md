---
title: BC & Vancouver Demographics - 2021 Census
toc: false
---

<div style="display: flex; justify-content: space-between; align-items: start; gap: 2rem; margin-bottom: 2rem;">
  <div>
    <h1 style="margin: 0 0 0.5rem 0;">British Columbia & Vancouver Demographics</h1>
    <p style="margin: 0; color: var(--theme-foreground-muted);">2021 Census of Population - Comparing BC Province and Vancouver City</p>
    <p style="margin: 0; color: var(--theme-foreground-muted);"><strong>Data source:</strong> <a href="https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/index.cfm" target="_blank">Statistics Canada 2021 Census of Population</a> | <strong>Last Updated:</strong> 2025-11-07</p>
  </div>

</div>

```js
const bcCensusRaw = FileAttachment("data/census-bc-cleaned.csv").csv({typed: true});
const vancouverCensusRaw = FileAttachment("data/census-vancouver-cleaned.csv").csv({typed: true});

const bcData = await bcCensusRaw;
const vancouverData = await vancouverCensusRaw;

// Helper function to get value by characteristic
function getValue(data, topic, characteristic) {
  const row = data.find(r =>
    r.Topic === topic && r.Characteristic === characteristic
  );
  if (!row) return 0;
  const val = String(row.Total || "0").replace(/,/g, '').trim();
  return parseFloat(val) || 0;
}

// Population data
const bcPopulation = getValue(bcData, "Population and dwellings", "Population, 2021");
const vancouverPopulation = getValue(vancouverData, "Population and dwellings", "Population, 2021");
const bcPrevPopulation = getValue(bcData, "Population and dwellings", "Population, 2016");
const vancouverPrevPopulation = getValue(vancouverData, "Population and dwellings", "Population, 2016");
const bcGrowth = ((bcPopulation - bcPrevPopulation) / bcPrevPopulation * 100);
const vancouverGrowth = ((vancouverPopulation - vancouverPrevPopulation) / vancouverPrevPopulation * 100);

// Demographics
const bcMedianAge = getValue(bcData, "Age characteristics", "Median age of the population");
const vancouverMedianAge = getValue(vancouverData, "Age characteristics", "Median age of the population");

// Housing
const bcDwellings = getValue(bcData, "Population and dwellings", "Private dwellings occupied by usual residents");
const vancouverDwellings = getValue(vancouverData, "Population and dwellings", "Private dwellings occupied by usual residents");
const bcAvgHouseholdSize = getValue(bcData, "Household and dwelling characteristics", "Average household size");
const vancouverAvgHouseholdSize = getValue(vancouverData, "Household and dwelling characteristics", "Average household size");

// Income
const bcMedianIncome = getValue(bcData, "Income of individuals in 2020", "    Median total income in 2020 among recipients ($)");
const vancouverMedianIncome = getValue(vancouverData, "Income of individuals in 2020", "    Median total income in 2020 among recipients ($)");

// Build age distribution data
const ageGroups = [
  { name: "0 to 14 years", spaces: 2 },
  { name: "15 to 19 years", spaces: 4 },
  { name: "20 to 24 years", spaces: 4 },
  { name: "25 to 29 years", spaces: 4 },
  { name: "30 to 34 years", spaces: 4 },
  { name: "35 to 39 years", spaces: 4 },
  { name: "40 to 44 years", spaces: 4 },
  { name: "45 to 49 years", spaces: 4 },
  { name: "50 to 54 years", spaces: 4 },
  { name: "55 to 59 years", spaces: 4 },
  { name: "60 to 64 years", spaces: 4 },
  { name: "65 years and over", spaces: 2 }
];

const ageDistribution = ageGroups.flatMap(groupObj => {
  const indent = " ".repeat(groupObj.spaces);
  const bcCount = getValue(bcData, "Age characteristics", `${indent}${groupObj.name}`);
  const vancouverCount = getValue(vancouverData, "Age characteristics", `${indent}${groupObj.name}`);

  const cleanName = groupObj.name.replace(" years", "");

  return [
    {
      ageGroup: cleanName,
      percent: (bcCount / bcPopulation) * 100,
      region: "British Columbia"
    },
    {
      ageGroup: cleanName,
      percent: (vancouverCount / vancouverPopulation) * 100,
      region: "Vancouver"
    }
  ];
});

// Dwelling types
const dwellingTypes = [
  "Single-detached house",
  "Semi-detached house",
  "Row house",
  "Apartment or flat in a duplex",
  "Apartment in a building that has fewer than five storeys",
  "Apartment in a building that has five or more storeys"
];

const dwellingData = dwellingTypes.flatMap(type => {
  const bcCount = getValue(bcData, "Household and dwelling characteristics", `  ${type}`);
  const vancouverCount = getValue(vancouverData, "Household and dwelling characteristics", `  ${type}`);

  return [
    {
      type: type.replace("Apartment in a building that has ", "").replace("Apartment or flat in a ", ""),
      count: bcCount,
      percent: (bcCount / bcDwellings) * 100,
      region: "British Columbia"
    },
    {
      type: type.replace("Apartment in a building that has ", "").replace("Apartment or flat in a ", ""),
      count: vancouverCount,
      percent: (vancouverCount / vancouverDwellings) * 100,
      region: "Vancouver"
    }
  ];
});
```

<div class="grid grid-cols-4">
  <div class="card">
    <h2>BC Population</h2>
    <span class="big">${(bcPopulation / 1000000).toFixed(2)}M</span>
    <span class="muted">2021 Population</span>
  </div>
  <div class="card">
    <h2>Vancouver Population</h2>
    <span class="big">${(vancouverPopulation / 1000).toFixed(0)}K</span>
    <span class="muted">2021 Population</span>
  </div>
  <div class="card">
    <h2>BC Growth (2016-2021)</h2>
    <span class="big green">${bcGrowth >= 0 ? '+' : ''}${bcGrowth.toFixed(1)}%</span>
    <span class="muted">Population change</span>
  </div>
  <div class="card">
    <h2>Vancouver Growth (2016-2021)</h2>
    <span class="big green">${vancouverGrowth >= 0 ? '+' : ''}${vancouverGrowth.toFixed(1)}%</span>
    <span class="muted">Population change</span>
  </div>
</div>

<div class="grid grid-cols-4">
  <div class="card">
    <h2>BC Median Age</h2>
    <span class="big">${bcMedianAge.toFixed(1)}</span>
    <span class="muted">years</span>
  </div>
  <div class="card">
    <h2>Vancouver Median Age</h2>
    <span class="big">${vancouverMedianAge.toFixed(1)}</span>
    <span class="muted">years</span>
  </div>
  <div class="card">
    <h2>BC Median Income</h2>
    <span class="big">$${(bcMedianIncome / 1000).toFixed(0)}K</span>
    <span class="muted">2020 income</span>
  </div>
  <div class="card">
    <h2>Vancouver Median Income</h2>
    <span class="big">$${(vancouverMedianIncome / 1000).toFixed(0)}K</span>
    <span class="muted">2020 income</span>
  </div>
</div>

<div class="grid grid-cols-2">
  <div class="card">
    <h2>Age Distribution Comparison</h2>
    ${resize((width) => createAgeDistributionChart(width))}
  </div>
  <div class="card">
    <h2>Dwelling Types by Region</h2>
    ${resize((width) => createDwellingChart(width))}
  </div>
</div>

```js
function createAgeDistributionChart(width) {
  const isMobile = width < 640;

  return Plot.plot({
    width,
    height: isMobile ? 300 : 350,
    marginLeft: isMobile ? 60 : 70,
    marginRight: isMobile ? 20 : 30,
    marginBottom: isMobile ? 90 : 80,
    style: {
      fontSize: "16px",
      fontWeight: "600"
    },
    x: {
      label: null,
      tickRotate: -45
    },
    y: {
      label: "Population (%)",
      grid: true
    },
    color: {
      domain: ["British Columbia", "Vancouver"],
      range: ["#3b82f6", "#0ea5e9"],
      legend: true
    },
    marks: [
      Plot.barY(ageDistribution, {
        x: "ageGroup",
        y: "percent",
        fill: "region",
        tip: true,
        title: d => `${d.region}\n${d.ageGroup}: ${d.percent.toFixed(1)}%`
      }),
      Plot.ruleY([0])
    ]
  });
}
```

```js
function createDwellingChart(width) {
  const isMobile = width < 640;

  return Plot.plot({
    width,
    height: isMobile ? 300 : 350,
    marginLeft: isMobile ? 180 : 200,
    marginRight: isMobile ? 20 : 30,
    marginBottom: isMobile ? 50 : 45,
    style: {
      fontSize: "16px",
      fontWeight: "600"
    },
    x: {
      label: "Percentage (%)",
      grid: true
    },
    y: {
      label: null
    },
    color: {
      domain: ["British Columbia", "Vancouver"],
      range: ["#3b82f6", "#0ea5e9"],
      legend: true
    },
    marks: [
      Plot.barX(dwellingData, {
        y: "type",
        x: "percent",
        fill: "region",
        tip: true,
        title: d => `${d.region}\n${d.type}: ${d.percent.toFixed(1)}%`
      }),
      Plot.ruleX([0])
    ]
  });
}
```
