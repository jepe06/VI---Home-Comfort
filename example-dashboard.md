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


const filteredDataset = selectedDataset.value.filter((d) => +d.temperature > 10 && +d.humidity > 30 && (d.year== "" ||+d.year == selectedYear.value));
```
Valor da condicao ${filteredDataset[1]} 

```js

//if (filteredDataset.lenght> 0){
const avgTemp = d3.mean(filteredDataset, (d) => +d.temperature).toFixed(1);

// Calculate average humidity using d3.mean
const avgHumidity = d3.mean(filteredDataset, (d) => +d.humidity).toFixed(1);

 

document.getElementById("avgTemp").textContent = `${avgTemp}°C`;
document.getElementById("avgHumidity").textContent = `${avgHumidity}%`;
//}
//else{
 //  const avgTemp =null
 // const avgHumidity =null
 // document.getElementById("avgTemp").textContent = `N/A`;
//document.getElementById("avgHumidity").textContent = `N/A`;
//}
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
filteredDataset.forEach((d) => {
  d.date = `${d.year}-${String(d.month).padStart(2, "0")}-${String(d.day).padStart(2, "0")}`;
});

```

// médias de cada dia

```js

function calculateDailyAverages(data) {
  // Group the data by the full date (year-month-day)
  const groupedData = d3.group(data, (d) => `${d.year}-${String(d.month).padStart(2, "0")}-${String(d.day).padStart(2, "0")}`);

  // Return an array with the date and the average temperature and humidity for each day
  return Array.from(groupedData, ([date, values]) => ({
    date, // Full date in 'YYYY-MM-DD' format
    avgTemperature: d3.mean(values, (d) => +d.temperature), // Average temperature
    avgHumidity: d3.mean(values, (d) => +d.humidity) // Average humidity
  }));
}
const dailyAverages = calculateDailyAverages(filteredDataset)
```
```js

function todosDias(dailyAverages , {width} = {}) {
  return Plot.plot({
    title: "Temperatura e Humidade Média",
    marks: [
      // Line for average temperature
      Plot.line(dailyAverages , {
        x: "date",
        y: "avgTemperature",
        stroke: "steelblue",
        strokeWidth: 2,
        title: (d) => `Temperature: ${d.avgTemperature.toFixed(1)}°C`
      }),
      // Line for average humidity
      Plot.line(dailyAverages , {
        x: "date",
        y: "avgHumidity",
        stroke: "green",
        strokeWidth: 2,
        title: (d) => `Humidity: ${d.avgHumidity.toFixed(1)}%`
      }),
    ],
    x: {
      label: "Mês",
      tickFormat: d3.utcFormat("%b %Y") // Format: "Jan 2021"
    },
    y: {
      label: "Value",
      grid: true // Add gridlines for better readability
    },
    width: width, // Dynamic width based on container size
    height: 500,
    color: {
      legend: true, // Add a legend for better visualization
    }
  });
}

```


<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Daily Averages Example</title>
</head>
<body>
  <h2>Daily Averages Table</h2>
  <table border="1" id="daily-averages-table">
    <thead>
      <tr>
        <th>Date</th>
        <th>Average Temperature (°C)</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <script>
 
    const dailyAverages = calculateDailyAverages(filteredDataset)
    const tbody = document.querySelector('#daily-averages-table tbody');

    // Create table rows for each entry in the dailyAverages array
    dailyAverages.forEach(entry => {
      const row = document.createElement('tr');
      const dateCell = document.createElement('td');
      dateCell.textContent = entry.date;
      const tempCell = document.createElement('td');
      tempCell.textContent = entry.avgTemperature;

      row.appendChild(dateCell);
      row.appendChild(tempCell);
      tbody.appendChild(row);
    });
  </script>
</body>
</html>

```js
//calcular médias de cada mês
function calculateMonthlyAverages(data) {

  const groupedData = d3.group(data, (d) => `${d.year}-${String(d.month).padStart(2, "0")}`);

  return Array.from(groupedData, ([yearmonth, values]) => ({
    month: new Date(`${yearmonth}-01`),
    avgTemperature: d3.mean(values, (d) => +d.temperature),
    avgHumidity: d3.mean(values, (d) => +d.humidity)
  }));
}
```

```js
const montlyAverages = calculateMonthlyAverages(filteredDataset)

```
```js
function todosMeses(montlyAverages, {width} = {}) {
  return Plot.plot({
    title: "Temperatura e Humidade Média",
    marks: [
      // Line for average temperature
      Plot.line(montlyAverages, {
        x: "month",
        y: "avgTemperature",
        stroke: "steelblue",
        strokeWidth: 2,
        title: (d) => `Temperature: ${d.avgTemperature.toFixed(1)}°C`
      }),
      // Line for average humidity
      Plot.line(montlyAverages, {
        x: "month",
        y: "avgHumidity",
        stroke: "green",
        strokeWidth: 2,
        title: (d) => `Humidity: ${d.avgHumidity.toFixed(1)}%`
      }),
    ],
    x: {
      label: "Mês",
      tickFormat: d3.utcFormat("%b %Y") // Format: "Jan 2021"
    },
    y: {
      label: "Value",
      grid: true // Add gridlines for better readability
    },
    width: width, // Dynamic width based on container size
    height: 500,
    color: {
      legend: true, // Add a legend for better visualization
    }
  });
}


```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => todosMeses (montlyAverages, {width}))}

  </div>
</div>

```js

function todosDiass(dailyAverages, { width }) {
  // Set the dimensions and margins of the graph
  const margin = { top: 20, right: 30, bottom: 110, left: 40 },
        margin2 = { top: 430, right: 30, bottom: 30, left: 40 },
        height = 500 - margin.top - margin.bottom,
        height2 = 500 - margin2.top - margin2.bottom;

  // Create an SVG element
  const svg = d3
    .create("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .style("background", "#ffffff");

  // Add clip path to prevent drawing outside the chart area
  svg
    .append("defs")
    .append("clipPath")
    .attr("id", "clip")
    .append("rect")
    .attr("width", width)
    .attr("height", height);

  // Add focus area for main chart
  const focus = svg
    .append("g")
    .attr("class", "focus")
    .attr("transform", `translate(${margin.left},${margin.top})`);

  // Add context area for zoom and brush
  const context = svg
    .append("g")
    .attr("class", "context")
    .attr("transform", `translate(${margin2.left},${margin2.top})`);

  // Create scales for focus and context
  const x = d3.scaleTime().range([0, width]);
  const x2 = d3.scaleTime().range([0, width]);
  const y = d3.scaleLinear().range([height, 0]);
  const y2 = d3.scaleLinear().range([height2, 0]);

  // Create axes
  const xAxis = d3.axisBottom(x);
  const xAxis2 = d3.axisBottom(x2);
  const yAxis = d3.axisLeft(y);

  // Use avgTemperaturePerDay as the data source
  const data = dailyAverages;

  // Update domains based on avgTemperaturePerDay
  x.domain(d3.extent(data, (d) => d.date));
  y.domain([0, d3.max(data, (d) => d.avgTemperature)]);
  x2.domain(x.domain());
  y2.domain(y.domain());

  // Create brush and zoom behaviors
  const brush = d3
    .brushX()
    .extent([
      [0, 0],
      [width, height2],
    ])
    .on("brush end", brushed);

  const zoom = d3
    .zoom()
    .scaleExtent([1, Infinity])
    .translateExtent([
      [0, 0],
      [width, height],
    ])
    .extent([
      [0, 0],
      [width, height],
    ])
    .on("zoom", zoomed);

  // Append context path (overview line chart)
  context
    .append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1.0)
    .attr(
      "d",
      d3
        .line()
        .defined((d) => d.avgTemperature != null && !isNaN(d.avgTemperature)) // Ignore missing data points
        .x((d) => x2(d.date))
        .y((d) => y2(d.avgTemperature))
    );

  // Append rectangles for brush and zoom behavior
  context.append("g").attr("class", "brush").call(brush);

  svg
    .append("rect")
    .attr("class", "zoom")
    .attr("width", width)
    .attr("height", height)
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .style("fill", "none")
    .style("pointer-events", "all") // Enable zoom interaction
    .call(zoom);

  // Append focus path (main line chart)
  const focusLine = focus
    .append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1.5)
    .attr(
      "d",
      d3
        .line()
        .defined((d) => d.avgTemperature != null && !isNaN(d.avgTemperature)) // Ignore missing data points
        .x((d) => x(d.date))
        .y((d) => y(d.avgTemperature))
    );

  // Add axes to focus and context areas
  focus
    .append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height})`)
    .call(xAxis);

  focus.append("g").attr("class", "axis axis--y").call(yAxis);

  focus
    .append("line")
    .attr("class", "y-threshold")
    .attr("x1", 0)
    .attr("x2", width)
    //.attr("y1", idealV) //
   // .attr("y2", idealV) //
    .attr("stroke", "red") // Line color
    .attr("stroke-width", 1) // Line thickness
    .attr("stroke-dasharray", "4 2"); // Dashed line

  context
    .append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height2})`)
    .call(xAxis2);

  // Function to update focus area (line and axes)
  function updateFocus() {
    focusLine.attr(
      "d",
      d3
        .line()
        .x((d) => x(d.date))
        .y((d) => y(d.avgTemperature))
    );

    focus.select(".axis--x").call(xAxis);
  }

  // Function for brushing interaction
  function brushed(event) {
    if (event.selection) {
      const [x0, x1] = event.selection.map(x2.invert);
      x.domain([x0, x1]);
      updateFocus();
    }
  }

  // Function for zooming interaction
  function zoomed(event) {
    const transform = event.transform;
    const newX = transform.rescaleX(x2);
    x.domain(newX.domain());
    updateFocus();
    context
      .select(".brush")
      .call(brush.move, x.range().map(transform.invertX, transform));
  }

  // Return the SVG node (this is what will be inserted into the DOM)
  return svg.node();
}


```
<div class="grid grid-cols-1">
  <div class="card">
    <script>
      
      ${resize((width) => todosDiass (dailyAverages, {width}))}

    </script>

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
