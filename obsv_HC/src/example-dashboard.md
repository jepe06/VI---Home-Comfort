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
const file1 = await FileAttachment("/Users/joaop.cardoso/MestradoCD/VI/Projecto VI/VI---Home-Comfort/obsv_HC/src/data/02A-sgh02015d5c61cc@1.csv").csv({ typed: true });
const A3 = await FileAttachment("/Users/joaop.cardoso/MestradoCD/VI/Projecto VI/VI---Home-Comfort/obsv_HC/src/data/03A-sgh0201a8c87da4@1.csv").csv({ typed: true });
const L5 = await FileAttachment("/Users/joaop.cardoso/MestradoCD/VI/Projecto VI/VI---Home-Comfort/obsv_HC/src/data/05L-sgh020125bce03a@1.csv").csv({ typed: true }); 
const L8 =await  FileAttachment("/Users/joaop.cardoso/MestradoCD/VI/Projecto VI/VI---Home-Comfort/obsv_HC/src/data/08L-sgh02018fe9be2c@1.csv").csv({ typed: true }); 
const A10 =await  FileAttachment("/Users/joaop.cardoso/MestradoCD/VI/Projecto VI/VI---Home-Comfort/obsv_HC/src/data/10A-sgh0201f6cb55ed@1.csv").csv({ typed: true }); 
const launches= FileAttachment("/Users/joaop.cardoso/MestradoCD/VI/Projecto VI/VI---Home-Comfort/obsv_HC/src/data/launches.csv").csv({ typed: true }); 
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
    { label: "2020", value: 2020 }
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
//Dados filtrados para valores realistas
const anoSelecionado = selectedYear.label
const filteredDataset = selectedDataset.value.filter((d) => +d.temperature > 10 && +d.humidity > 30 && (selectedYear.value == "" || +d.year == selectedYear.value));
```


```js

if (filteredDataset.lenght!== 0){
const avgTemp = d3.mean(filteredDataset, (d) => +d.temperature).toFixed(1);

// Calculate average humidity using d3.mean
const avgHumidity = d3.mean(filteredDataset, (d) => +d.humidity).toFixed(1);

 

document.getElementById("avgTemp").textContent = `${avgTemp}°C`;
document.getElementById("avgHumidity").textContent = `${avgHumidity}%`;
}
else{
   const avgTemp =null
  const avgHumidity =null
  document.getElementById("avgTemp").textContent = `N/A`;
document.getElementById("avgHumidity").textContent = `N/A`;
}
```

```js
filteredDataset.forEach((d) => {
  d.date = `${d.year}-${String(d.month).padStart(2, "0")}-${String(
    d.day
  ).padStart(2, "0")}`;
})
```
// médias de cada dia

```js

function calculateDailyAverages(data) {
  const groupedData = d3.group(data, (d) => d.date);

  return Array.from(groupedData, ([date, values]) => ({
    date,
    avgTemperature: d3.mean(values, (d) => +d.temperature),
    avgHumidity: d3.mean(values, (d) => +d.humidity)
  }));
}

const dailyAverages = calculateDailyAverages(filteredDataset)

```

```js
//calcular médias de cada mês
function calculateMonthlyAverages(data) {
  // Group by year and month (e.g., "2024-11" for November 2024)
  const groupedData = d3.group(data, (d) => `${d.year}-${String(d.month).padStart(2, "0")}`);

  return Array.from(groupedData, ([month, values]) => ({
    month,
    avgTemperature: d3.mean(values, (d) => +d.temperature),
    avgHumidity: d3.mean(values, (d) => +d.humidity)
  }));
}

const montlyAverages = calculateMonthlyAverages(filteredDataset)
```


<div class="grid grid-cols-1">
  <div class="card">
    ${resize(width => Plot.plot({
      marks: [
        Plot.line(montlyAverages, {
          x: "date",
          y: "avgTemperature",
          stroke: "steelblue"
        })
      ],
      x: {
        label: "Day of the Month",
        tickFormat: d3.utcFormat("%b") // Format the x-axis labels for month and day
      },
      y: {
        label: "Average Temperature (°C)"
      },
      width: width, // Dynamic width based on container size
      height: 400
    }))}

  </div>
</div>

```js
const fileData = selectedDataset.value;
const filteredData = fileData.filter((d) => {
  // If both year and month selectors are empty, return the whole dataset
  if (selectedYear.value === "" && selectedMonth.value === "") {
    return true; // Include all records
  }

  // If only year is selected, filter by year only
  if (selectedYear.value !== "" && selectedMonth.value === "") {
    return d.year == selectedYear.value;
  }

  // If only month is selected, filter by month across all years
  if (selectedYear.value === "" && selectedMonth.value !== "") {
    return d.month == selectedMonth.value;
  }

  // If both year and month are selected, filter by both
  return d.year == selectedYear.value && d.month == selectedMonth.value;
})

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

```js
Inputs.table(avgTemp)
Plot.plot({
  marks: [
    Plot.line(avgTemp, {
      x: "date",
      y: "Temperature",
      stroke: "steelblue"
    })
  ],
  x: {
    label: "Day of the Month",
    tickFormat: d3.utcFormat("%b-%d") // Format the x-axis labels for month and day
  },
  y: {
    label: "Average Temperature (°C)"
  },
  width: 800,
  height: 400
})
<!-- Plot of launch history -->
```


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
