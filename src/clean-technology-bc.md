---
title: Clean Technology Sector - British Columbia
toc: false
---

# Clean Technology Sector - British Columbia

Employment in environmental and clean technology products (2012-2023)

```js
const cleanTechRaw = FileAttachment("data/clean-tech-cleaned.csv").csv({typed: true});
const cleanTechData = (await cleanTechRaw).map(row => {
  const category = row["Goods and services (products)"];
  // Filter out rows with missing data (..)
  if (!category) return null;

  // Parse year values, treating ".." as 0
  const parseValue = (val) => {
    const str = String(val || "0").replace(/,/g, '').replace('p', '').trim();
    if (str === ".." || str === "") return 0;
    return parseFloat(str) || 0;
  };

  return {
    category: category,
    y2012: parseValue(row["2012"]),
    y2013: parseValue(row["2013"]),
    y2014: parseValue(row["2014"]),
    y2015: parseValue(row["2015"]),
    y2016: parseValue(row["2016"]),
    y2017: parseValue(row["2017"]),
    y2018: parseValue(row["2018"]),
    y2019: parseValue(row["2019"]),
    y2020: parseValue(row["2020"]),
    y2021: parseValue(row["2021"]),
    y2022: parseValue(row["2022"]),
    y2023: parseValue(row["2023"])
  };
}).filter(d => d !== null && (d.y2023 > 0 || d.category.includes("Total")));

const years = [2012, 2013, 2014, 2015, 2016, 2017, 2018, 2019, 2020, 2021, 2022, 2023];
const timeSeriesData = [];

years.forEach(year => {
  cleanTechData.forEach(d => {
    const value = d[`y${year}`];
    if (!isNaN(value) && value > 0) {
      timeSeriesData.push({
        year: year,
        category: d.category,
        employment: value
      });
    }
  });
});

const latestYear = d3.max(years);
```

```js
const selectedYear = view(Inputs.select(years, {
  value: latestYear,
  label: "Select Year"
}));
```

```js
const yearData = timeSeriesData.filter(d => d.year === selectedYear);

const totalEmployment = yearData
  .filter(d => d.category === "Total, environmental and clean technology products")
  .reduce((sum, d) => sum + d.employment, 0);

const envProducts = yearData
  .filter(d => d.category === "Total, environmental products")
  .reduce((sum, d) => sum + d.employment, 0);

const cleanTech = yearData
  .filter(d => d.category === "Total, clean technology products")
  .reduce((sum, d) => sum + d.employment, 0);

const prevYearData = timeSeriesData.filter(d => d.year === selectedYear - 1);
const prevTotal = prevYearData
  .filter(d => d.category === "Total, environmental and clean technology products")
  .reduce((sum, d) => sum + d.employment, 0);

const yoyGrowth = prevTotal > 0 ? ((totalEmployment - prevTotal) / prevTotal * 100) : 0;
```

<div class="grid grid-cols-4">
  <div class="card">
    <h2>Total Employment</h2>
    <span class="big">${(totalEmployment / 1000).toFixed(1)}K</span>
    <span class="muted">Jobs (${selectedYear})</span>
  </div>
  <div class="card">
    <h2>Environmental</h2>
    <span class="big">${(envProducts / 1000).toFixed(1)}K</span>
    <span class="muted">${((envProducts/totalEmployment)*100).toFixed(1)}% of total</span>
  </div>
  <div class="card">
    <h2>Clean Technology</h2>
    <span class="big">${(cleanTech / 1000).toFixed(1)}K</span>
    <span class="muted">${((cleanTech/totalEmployment)*100).toFixed(1)}% of total</span>
  </div>
  <div class="card">
    <h2>YoY Growth</h2>
    <span class="big ${yoyGrowth >= 0 ? 'green' : 'red'}">${yoyGrowth >= 0 ? '+' : ''}${yoyGrowth.toFixed(1)}%</span>
    <span class="muted">vs ${selectedYear - 1}</span>
  </div>
</div>

<div class="grid grid-cols-3">
  <div class="card">
    <h2>Employment Trends (2012-${selectedYear})</h2>
    ${resize((width) => createTrendChart(width))}
  </div>
  <div class="card">
    <h2>Environmental vs Clean Tech (${selectedYear})</h2>
    ${resize((width) => createCategoryChart(width))}
  </div>
  <div class="card">
    <h2>Year-over-Year Growth Rate</h2>
    ${resize((width) => createGrowthChart(width))}
  </div>
</div>

<div class="note">
**Data source:** <a href="https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=3610068101" target="_blank">Statistics Canada Table 36-10-0681-01</a> - British Columbia employment data for environmental and clean technology products (2012-2023)
</div>

```js
function createTrendChart(width) {
  const isMobile = width < 640;

  const envProductsByYear = timeSeriesData
    .filter(d => d.category === "Total, environmental products" && d.year <= selectedYear)
    .map(d => ({...d, series: "Environmental Products"}));

  const cleanTechByYear = timeSeriesData
    .filter(d => d.category === "Total, clean technology products" && d.year <= selectedYear)
    .map(d => ({...d, series: "Clean Technology"}));

  const combinedData = [...envProductsByYear, ...cleanTechByYear];

  return Plot.plot({
    width,
    height: isMobile ? 280 : 320,
    marginLeft: isMobile ? 50 : 60,
    marginRight: isMobile ? 20 : 30,
    marginBottom: isMobile ? 45 : 40,
    x: {
      label: "Year",
      grid: true
    },
    y: {
      label: "Employment (jobs)",
      grid: true,
      tickFormat: d => (d/1000).toFixed(0) + "K"
    },
    color: {
      domain: ["Environmental Products", "Clean Technology"],
      range: ["#0ea5e9", "#059669"],
      legend: true
    },
    marks: [
      Plot.line(combinedData, {
        x: "year",
        y: "employment",
        stroke: "series",
        strokeWidth: 2.5,
        curve: "catmull-rom"
      }),
      Plot.dot(combinedData, {
        x: "year",
        y: "employment",
        stroke: "series",
        r: 3,
        fill: "transparent",
        tip: true,
        title: d => `${d.series}\n${d.year}: ${d.employment.toLocaleString()} jobs`
      })
    ]
  });
}
```

```js
function createCategoryChart(width) {
  const isMobile = width < 640;

  const mainCategories = yearData.filter(d =>
    d.category === "Total, environmental products" ||
    d.category === "Total, clean technology products"
  );

  return Plot.plot({
    width,
    height: isMobile ? 240 : 280,
    marginLeft: isMobile ? 140 : 160,
    marginRight: isMobile ? 20 : 30,
    marginBottom: isMobile ? 40 : 35,
    x: {
      label: "Employment (jobs)",
      grid: true,
      tickFormat: d => (d/1000).toFixed(0) + "K"
    },
    y: {
      label: null
    },
    marks: [
      Plot.barX(mainCategories, {
        y: d => d.category.replace("Total, ", "").charAt(0).toUpperCase() + d.category.replace("Total, ", "").slice(1),
        x: "employment",
        fill: "#3b82f6",
        tip: true,
        title: d => `${d.employment.toLocaleString()} jobs`
      }),
      Plot.ruleX([0])
    ]
  });
}
```

```js
function createGrowthChart(width) {
  const isMobile = width < 640;

  const growthData = [];
  for (let i = 1; i < years.length; i++) {
    if (years[i] <= selectedYear) {
      const currentYear = years[i];
      const previousYear = years[i - 1];

      const currentTotal = timeSeriesData
        .filter(d => d.year === currentYear && d.category === "Total, environmental and clean technology products")
        .reduce((sum, d) => sum + d.employment, 0);

      const previousTotal = timeSeriesData
        .filter(d => d.year === previousYear && d.category === "Total, environmental and clean technology products")
        .reduce((sum, d) => sum + d.employment, 0);

      if (previousTotal > 0) {
        const growth = ((currentTotal - previousTotal) / previousTotal) * 100;
        growthData.push({
          year: currentYear,
          growth: growth
        });
      }
    }
  }

  return Plot.plot({
    width,
    height: isMobile ? 240 : 280,
    marginLeft: isMobile ? 50 : 60,
    marginRight: isMobile ? 20 : 30,
    marginBottom: isMobile ? 45 : 40,
    x: {
      label: "Year",
      grid: false
    },
    y: {
      label: "Growth Rate (%)",
      grid: true,
      tickFormat: d => d.toFixed(0) + "%"
    },
    marks: [
      Plot.ruleY([0], {stroke: "#ccc"}),
      Plot.barY(growthData, {
        x: "year",
        y: "growth",
        fill: d => d.growth >= 0 ? "#059669" : "#dc2626",
        tip: true,
        title: d => `${d.year}: ${d.growth >= 0 ? '+' : ''}${d.growth.toFixed(1)}%`
      })
    ]
  });
}
```
