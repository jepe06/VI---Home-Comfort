---
theme: dashboard
title: Example dashboard
toc: false
---

# Home confort🏠

<!-- Load and transform the data -->




```js
import { FileAttachment } from "@observablehq/stdlib";

```

```js
const file1 = await FileAttachment("data/02A-sgh02015d5c61cc@1.csv").csv({ typed: true });
const A3 = await FileAttachment("data/03A-sgh0201a8c87da4@1.csv").csv({ typed: true });
const L5 = await FileAttachment("data/05L-sgh020125bce03a@1.csv").csv({ typed: true }); 
const L8 =await  FileAttachment("data/08L-sgh02018fe9be2c@1.csv").csv({ typed: true }); 
const A10 =await  FileAttachment("data/10A-sgh0201f6cb55ed@1.csv").csv({ typed: true }); 
const launches= FileAttachment("data/launches.csv").csv({ typed: true }); 
```

```js
Inputs.table(A3)

```

```js

const selectedDataset = view(
  Inputs.select(
    [
      { label: "House Matos, Aveiro", value: file1 },
      { label: "House Barrancos, Aveiro", value: A3 },
      { label: "House Custódia, Lamego", value: L5 },
      { label: "House Ferradura, Lamego", value: L8 },
      { label:  "House Salitre, Aveiro", value: A10 }
    ],
    {
      label:"Select a Dataset:",
      format: (x) => x.label, // Display the dataset label
      value: (x) => x.value  // Use the dataset value
    }
  )
);
```
Casa selecionada é ${selectedDataset.label} 
```js
const selectedYear = view(Inputs.select(
  [
    { label: "All", value: "" },
    { label: "2019", value: 2019 },
    { label: "2020", value: 2020 },
    { label: "2021", value: 2021 }
  ],
  {
    label: "Select Year:",
    format: (x) => x.label,
    value: (x) => x.value
  }
))
```
Ano selecionado é ${selectedYear.label} 
<!-- A shared color scale for consistency, sorted by the number of launches -->

```js
const color = Plot.scale({
  color: {
    type: "categorical",
    domain: d3.groupSort(launches, (D) => -D.length, (d) => d.state).filter((d) => d !== "Other"),
    unknown: "var(--theme-foreground-muted)"
  }
});
```
```js
const avgTemp = d3.mean(selectedDataset.value, (d) => +d.temperature).toFixed(1);

// Calculate average humidity using d3.mean
const avgHumidity = d3.mean(selectedDataset.value, (d) => +d.humidity).toFixed(1);

document.getElementById("avgTemp").textContent = `${avgTemp}°C`;
document.getElementById("avgHumidity").textContent = `${avgHumidity}%`;
```
<!-- Cards with big numbers -->

<div class="grid grid-cols-4">
  <div class="card">
    <h2>Temperatura média</h2>
   <span class="big" id="avgTemp"></span>
  </div>
  <div class="card">
    <h2>Humidade média</h2>
   <span class="big" id="avgHumidity"></span>
  </div>
</div>


<!-- Plot of launch history -->

```js
function launchTimeline(data, {width} = {}) {
  return Plot.plot({
    title: "Launches over the years",
    width,
    height: 300,
    y: {grid: true, label: "Launches"},
    color: {...color, legend: true},
    marks: [
      Plot.rectY(data, Plot.binX({y: "count"}, {x: "date", fill: "state", interval: "year", tip: true})),
      Plot.ruleY([0])
    ]
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => launchTimeline(launches, {width}))}
  </div>
</div>

<!-- Plot of launch vehicles -->

```js
function vehicleChart(data, {width}) {
  return Plot.plot({
    title: "Popular launch vehicles",
    width,
    height: 300,
    marginTop: 0,
    marginLeft: 50,
    x: {grid: true, label: "Launches"},
    y: {label: null},
    color: {...color, legend: true},
    marks: [
      Plot.rectX(data, Plot.groupY({x: "count"}, {y: "family", fill: "state", tip: true, sort: {y: "-x"}})),
      Plot.ruleX([0])
    ]
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => vehicleChart(launches, {width}))}
  </div>
</div>

Data: Jonathan C. McDowell, [General Catalog of Artificial Space Objects](https://planet4589.org/space/gcat)
