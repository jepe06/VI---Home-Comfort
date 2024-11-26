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
<b> A casa selecionada é ${selectedDataset.label} <b>

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


```js
const selectedEstacao = view(Inputs.select(
  [
    { label: "All", value: "" },
    { label: "Verao", value: "verao" },
    { label: "Inverno", value: "inverno" }
  ],
  {
    label: "Select Season:",
    format: (x) => x.label,
    value: (x) => x.value
  }
))
```


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

const  estacaoMeses = {
  verao: [ 4, 5,6,7,8,9,10], 
  inverno: [11, 12, 1,2,3] 
};
```

```js

let temperatureR, humidityR,referenceValueTemp,referenceValueHum;

if (estacaoText === "Verao") {
  temperatureR = [21, 25];
  humidityR = [45, 55];
  referenceValueTemp = 24;
   referenceValueHum = 50;
} else if (estacaoText === "Inverno") {
  temperatureR = [19, 21]; //zonas de conforto
  humidityR = [50, 60];
  referenceValueTemp = 20;
  referenceValueHum = 55;
} else {
  temperatureR = [19, 25];
  humidityR = [45, 60];
  referenceValueTemp = 21;
  referenceValueHum = 50;
}



document.getElementById('referenceValueTemp').textContent = `${referenceValueTemp}°C`;
document.getElementById('referenceValueHum').textContent = `${referenceValueHum}%`;

```
<div class="grid grid-cols-2">
  <div class="card">
    <h2>Temperatura e Humidade de Referência</h2>
    <span class="big" id="referenceValueTemp"></span>
    <span class="big" id="referenceValueHum" style="margin-left: 30px;"></span>
  </div>
</div>

```js
// Filter the dataset based on the selected season
const filteredDatasetE2 = selectedDataset.value.filter((d) => {

  if (selectedYear.value && d.year !== +selectedYear.value) {
    return false;
  }

  // Check if the season matches the selected season (or "All")
  if (selectedEstacao.value) {
    const meses = estacaoMeses[selectedEstacao.value]; // Get months for the selected season
    if (!meses.includes(d.month)) {
      return false;
    }
  }

  // Check temperature and humidity thresholds
  return +d.temperature > 10 && +d.humidity > 30;
});


//esta tá a ser usado nos graficos
const filteredDataset = selectedDataset.value.filter((d) => +d.temperature > 10 && +d.humidity > 30 && (d.year== "" ||+d.year == selectedYear.value));
```


```js
//DASHBOARD STUFF
const yearExists = !selectedYear.value || filteredDatasetE2.some((d) => d.year == selectedYear.value);
  const yearText = yearExists ? (selectedYear.value || "Todos os anos") : "N/A";
  document.getElementById("selectedYear").textContent = yearText;

  
const avgTemp = d3.mean(filteredDatasetE2, (d) => +d.temperature)|| 0;;

// Calculate average humidity using d3.mean
const avgHumidity = d3.mean(filteredDatasetE2, (d) => +d.humidity)|| 0;;


 document.getElementById("avgTemp").textContent = avgTemp > 0 ? avgTemp.toFixed(1) + "°C" : "N/A";
document.getElementById("avgHumidity").textContent = avgHumidity > 0 ? avgHumidity.toFixed(1) + "%" : "N/A";

const avgTempElement = document.getElementById("avgTemp");


if (avgTemp< 2){
  avgTempElement.style.color = 'blue';
}
else{
avgTempElement.style.color = 'black';
}
  // Update selected season
const estacaoText = selectedEstacao.value
    ? selectedEstacao.label // Assuming `selectedEstacao` contains a label
    : "Todas as estações";
     document.getElementById("selectedSeason").textContent = estacaoText;





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
  <div class="card">
    <h2>Ano Selecionado</h2>
    <span class="big" id="selectedYear"></span>
  </div>
  <div class="card">
    <h2>Estação Selecionada</h2>
    <span class="big" id="selectedSeason"></span>
  </div>
</div>

```js
filteredDataset.forEach((d) => {
  d.date = `${d.year}-${String(d.month).padStart(2, "0")}-${String(d.day).padStart(2, "0")}`;
});

```

```js
//calcular médias de cada mês
function calculateMonthlyAverages(data,referenceValueTemp2 = referenceValueTemp, referenceValueHum2 = referenceValueHum) {

  const groupedData = d3.group(data, (d) => `${d.year}-${String(d.month).padStart(2, "0")}`);

  return Array.from(groupedData, ([yearmonth, values]) => {

    const avgTemperature3 = d3.mean(values, (d) => +d.temperature) || 0; 
    const avgHumidity3 = d3.mean(values, (d) => +d.humidity) || 0;
    const tempVariation = avgTemperature3 - referenceValueTemp2;
    const humidityVariation = avgHumidity3 - referenceValueHum2;
     return {
      month: new Date(`${yearmonth}-01`),
      avgTemperature3, 
      avgHumidity3, 
      tempVariation, 
      humidityVariation 
 
  }});
}
```

```js
const montlyAverages = calculateMonthlyAverages(filteredDataset)
```




```js
function todosMeses(montlyAverages, { width } = {}) {

  return Plot.plot({
    title: "Temperatura e Humidade Média",
    marks: [
      // Linha para temperatura média
      Plot.line(montlyAverages, {
        x: "month",
        y: "avgTemperature3",
        stroke: "steelblue",
        strokeWidth: 2,
        title: (d) => `Temperature: ${d.avgTemperature3.toFixed(1)}°C`
      }),
      // Linha para humidade média
      Plot.line(montlyAverages, {
        x: "month",
        y: "avgHumidity3",
        stroke: "green",
        strokeWidth: 2,
        title: (d) => `Humidity: ${d.avgHumidity3.toFixed(1)}%`
      }),
      Plot.text(montlyAverages, {
        x: "month",
        y: (d) => d.avgTemperature3 + (d.tempVariation > 0 ? 1.5 : -1.5), // Offset for visibility
        text: (d) => `${d.tempVariation > 0 ? "+" : ""}${d.tempVariation.toFixed(1)}°C`, // Variation in temperature
        fontSize: 12,
        fill: "steelblue",
        dx: 5, // Adjusts text position to the right of the point
      }),
      // Adding the humidity variation as text labels
      Plot.text(montlyAverages, {
        x: "month",
        y: (d) => d.avgHumidity3 + (d.humidityVariation > 0 ? 2 : -2), // Offset for visibility
        text: (d) => `${d.humidityVariation > 0 ? "+" : ""}${d.humidityVariation.toFixed(1)}%`, // Variation in humidity
        fontSize: 12,
        fill: "green",
        dx: 5, // Adjusts text position to the right of the point
      })
    
    ],
    x: {
      label: "Mês",
      tickFormat: d3.utcFormat("%b %Y") // Formato: "Jan 2021"
    },
    y: {
      label: "Value",
      grid: true, // Adicionar grelha para legibilidade
      tickCount: 20
    },
    width: width, 
    height: 700,
    color: {
      legend: true, 
    }
  });
}


```

<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => todosMeses (montlyAverages, {width}))}

  </div>
</div>

// médias de cada dia

```js

function calculateDailyAverages(data) {
  // Group the data by the full date (year-month-day)
  const groupedData = d3.group(data, (d) => d.date);

  // Return an array with the date and the average temperature and humidity for each day
  return Array.from(groupedData, ([date, values]) => ({
    date, // Full date in 'YYYY-MM-DD' format
    avgTemperature2: d3.mean(values, (d) => +d.temperature), // Average temperature
    avgHumidity2: d3.mean(values, (d) => +d.humidity), // Average humidity

  }));
}

function calculateDailyAverages2(data, referenceValueTemp2 = referenceValueTemp, referenceValueHum2 = referenceValueHum) {
  // Group the data by the full date (year-month-day)
  const groupedData = d3.group(data, (d) => d.date);

  // Return an array with the date, average temperature, humidity, and their variations from reference values
  return Array.from(groupedData, ([date, values]) => {
    // Calculate the average temperature and humidity for the current day
    const avgTemperature = d3.mean(values, (d) => +d.temperature) || 0; // Average temperature for the day
    const avgHumidity = d3.mean(values, (d) => +d.humidity) || 0; // Average humidity for the day

    // Calculate the variation of temperature and humidity with respect to the reference values
    const tempVariation = avgTemperature - referenceValueTemp2;
    const humidityVariation = avgHumidity - referenceValueHum2;

    // Return the object with the calculated values
    return {
      date, // Full date in 'YYYY-MM-DD' format
      avgTemperature, // Average temperature
      avgHumidity, // Average humidity
      tempVariation, // Variation in temperature from reference value
      humidityVariation // Variation in humidity from reference value
    };
  });
}

const dailyAverages2 = calculateDailyAverages2(filteredDataset)



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
        y: "avgTemperature2",
        stroke: "steelblue",
        strokeWidth: 2,
        title: (d) => `Temperature: ${d.avgTemperature2.toFixed(1)}°C`
      }),
      // Line for average humidity
      Plot.line(dailyAverages , {
        x: "date",
        y: "avgHumidity2",
        stroke: "green",
        strokeWidth: 2,
        title: (d) => `Humidity: ${d.avgHumidity2.toFixed(1)}%`
      }),
    ],
    x: {
      label: "Dia",
      //tickFormat: d3.utcFormat("%b %Y") // Format: "Jan 2021"
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



function todosDias2(dailyAverages2, {width} = {}) {
  return Plot.plot({
    title: "Temperatura e Humidade Média",
    marks: [
      // Line for average temperature
      Plot.line(dailyAverages2, {
        x: "date",
        y: "avgTemperature",
        stroke: "steelblue",
        strokeWidth: 2,
        title: (d) => `Temperature: ${d.avgTemperature.toFixed(1)}°C`,
      }),
      // Line for average humidity
      Plot.line(dailyAverages2, {
        x: "date",
        y: "avgHumidity",
        stroke: "green",
        strokeWidth: 2,
        title: (d) => `Humidity: ${d.avgHumidity.toFixed(1)}%`,
      }),
      // Adding the temperature variation as text labels
      Plot.text(dailyAverages2, {
        x: "date",
        y: (d) => d.avgTemperature + (d.tempVariation > 0 ? 1.5 : -1.5), // Offset for visibility
        text: (d) => `${d.tempVariation > 0 ? "+" : ""}${d.tempVariation.toFixed(1)}°C`, // Variation in temperature
        fontSize: 12,
        fill: "steelblue",
        dx: 5, // Adjusts text position to the right of the point
      }),
      // Adding the humidity variation as text labels
      Plot.text(dailyAverages2, {
        x: "date",
        y: (d) => d.avgHumidity + (d.humidityVariation > 0 ? 2 : -2), // Offset for visibility
        text: (d) => `${d.humidityVariation > 0 ? "+" : ""}${d.humidityVariation.toFixed(1)}%`, // Variation in humidity
        fontSize: 12,
        fill: "green",
        dx: 5, // Adjusts text position to the right of the point
      })
    ],
    x: {
      label: "Dia",
      //tickFormat: d3.utcFormat("%b %Y") // Format: "Jan 2021"
    },
    y: {
      label: "Value",
      grid: true, // Add gridlines for better readability
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
    ${resize((width) => todosDias2 (dailyAverages2, {width}))}

  </div>
</div>



<div class="grid grid-cols-1">
  <div class="card">
    ${resize((width) => todosDias (dailyAverages, {width}))}

  </div>
</div>


```js

function todosDiass(dailyAverages) {

  // Set up the chart dimensions and margin
  const margin = { top: 20, right: 30, bottom: 110, left: 40 },
        margin2 = { top: 430, right: 30, bottom: 30, left: 40 },
        width = 800 - margin.left - margin.right, // Width minus margin
        height = 500 - margin.top - margin.bottom, // Height minus margin
        height2 = 500 - margin2.top - margin2.bottom;

  // Create scales for focus and context
  const x = d3.scaleTime().range([0, width]);
  const x2 = d3.scaleTime().range([0, width]);
  const y = d3.scaleLinear().range([height, 0]);  // Left y-axis for temperature
  const y2 = d3.scaleLinear().range([height2, 0]);  // Right y-axis for humidity

  // Create axes
  const xAxis = d3.axisBottom(x);
  const xAxis2 = d3.axisBottom(x2);
  const yAxis = d3.axisLeft(y);
  const yAxis2 = d3.axisRight(y2);

  // Create SVG element
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

  // Use dailyAverages as the data source
  const data = dailyAverages;

  // Update domains based on avgTemperature3 and avgHumidity3
  x.domain(d3.extent(data, (d) => d.month));
  y.domain([0, d3.max(data, (d) => d.avgTemperature3)]); // Temperature on left y-axis
  y2.domain([0, d3.max(data, (d) => d.avgHumidity3)]); // Humidity on right y-axis
  x2.domain(x.domain());

  // Create brush and zoom behaviors
  const brush = d3
    .brushX()
    .extent([[0, 0], [width, height2]])
    .on("brush end", brushed);

  const zoom = d3
    .zoom()
    .scaleExtent([1, Infinity])
    .translateExtent([[0, 0], [width, height]])
    .extent([[0, 0], [width, height]])
    .on("zoom", zoomed);

  // Append context path (overview line chart) for both temperature and humidity
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
        .defined((d) => d.avgTemperature3 != null && !isNaN(d.avgTemperature3))
        .x((d) => x2(d.month))
        .y((d) => y2(d.avgTemperature3)) // Humidity line in context
    );

  context
    .append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "green")
    .attr("stroke-width", 1.0)
    .attr(
      "d",
      d3
        .line()
        .defined((d) => d.avgHumidity3 != null && !isNaN(d.avgHumidity3))
        .x((d) => x2(d.month))
        .y((d) => y2(d.avgHumidity3)) // Humidity line in context
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

  // Append focus path (main line chart) for both temperature and humidity
  const focusLineTemp = focus
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
        .defined((d) => d.avgTemperature3)
        .x((d) => x(d.month))
        .y((d) => y(d.avgTemperature3)) // Temperature line in focus
    );

  const focusLineHum = focus
    .append("path")
    .datum(data)
    .attr("class", "line")
    .attr("fill", "none")
    .attr("stroke", "green")
    .attr("stroke-width", 1.5)
    .attr(
      "d",
      d3
        .line()
        .defined((d) => d.avgHumidity3)
        .x((d) => x(d.month))
        .y((d) => y2(d.avgHumidity3)) // Humidity line in focus
    );

  // Add axes to focus and context areas
  focus
    .append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height})`)
    .call(xAxis);

  focus
    .append("g")
    .attr("class", "axis axis--y")
    .call(yAxis); // Left y-axis for temperature

  focus
    .append("g")
    .attr("class", "axis axis--y2")
    .attr("transform", `translate(${width}, 0)`)
    .call(yAxis2); // Right y-axis for humidity

  context
    .append("g")
    .attr("class", "axis axis--x")
    .attr("transform", `translate(0,${height2})`)
    .call(xAxis2);

  // Function for zoom interaction
  function zoomed(event) {
    const transform = event.transform;
    const newX = transform.rescaleX(x2);
    x.domain(newX.domain());
    updateFocus();
    context
      .select(".brush")
      .call(brush.move, x.range().map(transform.invertX, transform));
  }

  // Function to update focus area (line and axes)
  function updateFocus() {
    focusLineTemp.attr(
      "d",
      d3
        .line()
        .x((d) => x(d.month))
        .y((d) => y(d.avgTemperature3))
    );

    focusLineHum.attr(
      "d",
      d3
        .line()
        .x((d) => x(d.month))
        .y((d) => y2(d.avgHumidity3))
    );

    focus.select(".axis--x").call(xAxis);
    focus.select(".axis--y").call(yAxis);
    focus.select(".axis--y2").call(yAxis2);
  }

  // Function for brushing interaction
  function brushed(event) {
    if (event.selection) {
      const [x0, x1] = event.selection.map(x2.invert);
      x.domain([x0, x1]);
      updateFocus();
    }
  }

  // Return the SVG node (this is what will be inserted into the DOM)
  return svg.node();
}

```


```js


function todosDiasss(dailyAverages, { width }) {
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
    .attr("transform", translate(${margin.left},${margin.top}));

  // Add context area for zoom and brush
  const context = svg
    .append("g")
    .attr("class", "context")
    .attr("transform", translate(${margin2.left},${margin2.top}));

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
  const data = montlyAverages;

  // Update domains based on avgTemperaturePerDay
  x.domain(d3.extent(data, (d) => d.month));
  y.domain([0, d3.max(data, (d) => d.avgTemperature3)]);
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
        .defined((d) => d.avgTemperature3 != null && !isNaN(d.avgTemperature3)) // Ignore missing data points
        .x((d) => x2(d.month))
        .y((d) => y2(d.avgTemperature3))
    );

  // Append rectangles for brush and zoom behavior
  context.append("g").attr("class", "brush").call(brush);

  svg
    .append("rect")
    .attr("class", "zoom")
    .attr("width", width)
    .attr("height", height)
    .attr("transform", translate(${margin.left},${margin.top}))
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
        .defined((d) => d.avgTemperature3) //!= null && !isNaN(d.avgTemperature2)) // Ignore missing data points
        .x((d) => x(d.month))
        .y((d) => y(d.avgTemperature3))
    );

  // Add axes to focus and context areas
  focus
    .append("g")
    .attr("class", "axis axis--x")
    .attr("transform", translate(0,${height}))
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
    .attr("transform", translate(0,${height2}))
    .call(xAxis2);

  // Function to update focus area (line and axes)
  function updateFocus() {
    focusLine.attr(
      "d",
      d3
        .line()
        .x((d) => x(d.month))
        .y((d) => y(d.avgTemperature3))
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



```js
function createChart() {
  // Calculate daily averages for temperature and humidity
  
  // Assuming filteredDataset contains the dataset you want to process


  // Chart dimensions and margins
  const marginTop = 20, marginRight = 20, marginBottom = 30, marginLeft = 30;
  const width = 928, height = 500;

  // Create the horizontal and vertical scales for the area chart
  const x = d3.scaleUtc()
    .domain(d3.extent(dailyAverages, (d) => d.date))
    .range([marginLeft, width - marginRight]);

  const y = d3.scaleLinear()
    .domain([0, d3.max(dailyAverages, (d) => Math.max(d.avgTemperature2, d.avgHumidity2))])
    .nice()
    .range([height - marginBottom, marginTop]);

  // Create the horizontal axis generator
  const xAxis = (g, x) => g
    .call(d3.axisBottom(x).ticks(width / 80).tickSizeOuter(0));

  // Area generator for drawing the area chart
  const area = (data, x) => d3.area()
    .curve(d3.curveStepAfter)
    .x((d) => x(d.date))
    .y0(y(0))
    .y1((d) => y(d.avgTemperature2))  // Here we are plotting temperature, change to humidity if needed
    (data);

  // Create the zoom behavior
  const zoom = d3.zoom()
    .scaleExtent([1, 32])
    .extent([[marginLeft, 0], [width - marginRight, height]])
    .translateExtent([[marginLeft, -Infinity], [width - marginRight, Infinity]])
    .on("zoom", zoomed);

  // Create the SVG container for the chart
  const svg = d3.create("svg")
    .attr("viewBox", [0, 0, width, height])
    .attr("width", width)
    .attr("height", height)
    .attr("style", "max-width: 100%; height: auto;");

  // Create a clip-path for the area chart
  const clip = svg.append("defs")
    .append("clipPath")
    .attr("id", "clip")
    .append("rect")
    .attr("x", marginLeft)
    .attr("y", marginTop)
    .attr("width", width - marginLeft - marginRight)
    .attr("height", height - marginTop - marginBottom);

  // Create the path for the area
  const path = svg.append("path")
    .attr("clip-path", "url(#clip)") // Reference the clip-path defined above
    .attr("fill", "steelblue")
    .attr("d", area(dailyAverages, x));

  // Append the horizontal axis
  const gx = svg.append("g")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(xAxis, x);

  // Append the vertical axis
  svg.append("g")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(y).ticks(null, "s"))
    .call((g) => g.select(".domain").remove())
    .call((g) => g.select(".tick:last-of-type text").clone()
      .attr("x", 3)
      .attr("text-anchor", "start")
      .attr("font-weight", "bold")
      .text("Temperature (°C)")); // Adjust label accordingly

  // When zooming, redraw the area and the x-axis
  function zoomed(event) {
    const xz = event.transform.rescaleX(x);
    path.attr("d", area(dailyAverages, xz));
    gx.call(xAxis, xz);
  }

  // Initial zoom on a specific date range
  svg.call(zoom)
    .transition()
    .duration(750)
    .call(zoom.scaleTo, 4, [x(Date.UTC(2024, 0, 1)), 0]); // Zoom into a specific range (January 2024)

  // Append the SVG to the document
  document.body.appendChild(svg.node());
}

```













