---
title: Vancouver Island & Coast - Unemployment Rate
toc: false
---

<div style="display: flex; justify-content: space-between; align-items: start; gap: 2rem; margin-bottom: 2rem;">
  <div>
    <h1 style="margin: 0 0 0.5rem 0;">Vancouver Island & Coast - Unemployment Rate</h1>
    <p style="margin: 0; color: var(--theme-foreground-muted);">Three-month moving average unemployment rate (August - December 2024)</p>
  </div>
  <div class="note" style="text-align: right; font-size: 0.875rem; min-width: 300px;">
    <strong>Data source:</strong> <a href="https://www150.statcan.gc.ca/t1/tbl1/en/tv.action?pid=1410038701" target="_blank">Statistics Canada Table 14-10-0387-01</a> - Three-month moving average unemployment rates for Vancouver Island & Coast compared to BC and Canada (August - December 2024)
  </div>
</div>

```js
const labourRaw = FileAttachment("data/labour-force-cleaned.csv").csv({typed: true});
const rawData = await labourRaw;

const months = ["August 2024", "September 2024", "October 2024", "November 2024", "December 2024"];
const monthLabels = ["Aug 2024", "Sep 2024", "Oct 2024", "Nov 2024", "Dec 2024"];

// Find Vancouver Island row
const vancouverIslandRow = rawData.find(row =>
  row["Geography 4 5"] === "Vancouver Island and Coast, British Columbia"
);

const bcRow = rawData.find(row =>
  row["Geography 4 5"] === "British Columbia"
);

const canadaRow = rawData.find(row =>
  row["Geography 4 5"] === "Canada"
);

// Parse unemployment rates
const vancouverIslandData = months.map((month, idx) => ({
  month: monthLabels[idx],
  monthIndex: idx,
  rate: parseFloat(vancouverIslandRow[month]) || 0,
  region: "Vancouver Island & Coast"
}));

const bcData = months.map((month, idx) => ({
  month: monthLabels[idx],
  monthIndex: idx,
  rate: parseFloat(bcRow[month]) || 0,
  region: "British Columbia"
}));

const canadaData = months.map((month, idx) => ({
  month: monthLabels[idx],
  monthIndex: idx,
  rate: parseFloat(canadaRow[month]) || 0,
  region: "Canada"
}));

const allData = [...vancouverIslandData, ...bcData, ...canadaData];
const latestMonthIndex = months.length - 1;
```

```js
const selectedMonthIndex = view(Inputs.select(d3.range(months.length), {
  value: latestMonthIndex,
  format: i => monthLabels[i],
  label: "Select Month"
}));
```

```js
const currentMonth = monthLabels[selectedMonthIndex];
const currentVIRate = vancouverIslandData[selectedMonthIndex].rate;
const currentBCRate = bcData[selectedMonthIndex].rate;
const currentCanadaRate = canadaData[selectedMonthIndex].rate;
const prevVIRate = selectedMonthIndex > 0 ? vancouverIslandData[selectedMonthIndex - 1].rate : 0;
const viChange = currentVIRate - prevVIRate;
const viBCDiff = currentVIRate - currentBCRate;
const viCanadaDiff = currentVIRate - currentCanadaRate;
```

<div class="grid grid-cols-4">
  <div class="card">
    <h2>Vancouver Island</h2>
    <span class="big ${viChange > 0 ? 'red' : 'green'}">${currentVIRate.toFixed(1)}%</span>
    <span class="muted">${currentMonth}</span>
  </div>
  <div class="card">
    <h2>Month-over-Month</h2>
    <span class="big ${viChange > 0 ? 'red' : 'green'}">${viChange >= 0 ? '+' : ''}${viChange.toFixed(1)}pp</span>
    <span class="muted">Change from prev month</span>
  </div>
  <div class="card">
    <h2>vs BC Average</h2>
    <span class="big ${viBCDiff > 0 ? 'red' : 'green'}">${viBCDiff >= 0 ? '+' : ''}${viBCDiff.toFixed(1)}pp</span>
    <span class="muted">BC: ${currentBCRate.toFixed(1)}%</span>
  </div>
  <div class="card">
    <h2>vs Canada Average</h2>
    <span class="big ${viCanadaDiff > 0 ? 'red' : 'green'}">${viCanadaDiff >= 0 ? '+' : ''}${viCanadaDiff.toFixed(1)}pp</span>
    <span class="muted">Canada: ${currentCanadaRate.toFixed(1)}%</span>
  </div>
</div>

<div class="card">
  <h2>Unemployment Rate Comparison (Aug - ${currentMonth})</h2>
  ${resize((width) => createUnemploymentChart(width))}
</div>

```js
function createUnemploymentChart(width) {
  const isMobile = width < 640;
  const filteredData = allData
    .filter(d => d.monthIndex <= selectedMonthIndex)
    .sort((a, b) => a.monthIndex - b.monthIndex);

  return Plot.plot({
    width,
    height: isMobile ? 300 : 400,
    marginLeft: isMobile ? 50 : 60,
    marginRight: isMobile ? 20 : 100,
    marginBottom: isMobile ? 50 : 50,
    style: {
      fontSize: "16px",
      fontWeight: "600"
    },
    x: {
      label: "Month",
      grid: false,
      domain: monthLabels.slice(0, selectedMonthIndex + 1)
    },
    y: {
      label: "Unemployment Rate (%)",
      grid: true,
      domain: [0, 8]
    },
    color: {
      domain: ["Vancouver Island & Coast", "British Columbia", "Canada"],
      range: ["#0ea5e9", "#f59e0b", "#6366f1"],
      legend: true
    },
    marks: [
      Plot.line(filteredData, {
        x: "month",
        y: "rate",
        z: "region",
        stroke: "region",
        strokeWidth: 2.5,
        curve: "catmull-rom"
      }),
      Plot.dot(filteredData, {
        x: "month",
        y: "rate",
        stroke: "region",
        r: 4,
        fill: "white",
        strokeWidth: 2,
        tip: true,
        title: d => `${d.region}\n${d.month}: ${d.rate.toFixed(1)}%`
      })
    ]
  });
}
```
