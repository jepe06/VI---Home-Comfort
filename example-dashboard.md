---
theme: dashboard
title: Home comfort
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
//variacaoes
const avgVarT = d3.mean(montlyAverages, (d) => +d.tempVariation)
const avgVarH = d3.mean(montlyAverages, (d) => +d.humidityVariation)

```
Variação média de temperatura é ${avgVarT.toFixed(1)}°C e de Humidade é ${avgVarH.toFixed(1)}%

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
        y: (d) => d.avgTemperature3 + (d.tempVariation > 0 ? 1.5 : -1.5),
        text: (d) => `${d.tempVariation > 0 ? "+" : ""}${d.tempVariation.toFixed(1)}°C`, 
        fontSize: 12,
        fill: "steelblue",
        dx: 5, 
      }),

      Plot.text(montlyAverages, {
        x: "month",
        y: (d) => d.avgHumidity3 + (d.humidityVariation > 0 ? 2 : -2), 
        text: (d) => `${d.humidityVariation > 0 ? "+" : ""}${d.humidityVariation.toFixed(1)}%`, 
        fontSize: 12,
        fill: "green",
        dx: 5, 
      })
    
    ],
    x: {
      label: "Mês",
      tickFormat: d3.utcFormat("%b %Y") 
    },
    y: {
      label: "Value",
      grid: true, 
      tickCount: 20
    },
    width: width, 
    height: 700,
   color: {
      domain: ["Temperatura Média", "Humidade Média"], 
      range: ["steelblue", "green"], 
      legend: true,
    },
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
 
  const groupedData = d3.group(data, (d) => d.date);

  
  return Array.from(groupedData, ([date, values]) => ({
    date, 
    avgTemperature2: d3.mean(values, (d) => +d.temperature),
    avgHumidity2: d3.mean(values, (d) => +d.humidity),

  }));
}

function calculateDailyAverages2(data, referenceValueTemp2 = referenceValueTemp, referenceValueHum2 = referenceValueHum) {

  const groupedData = d3.group(data, (d) => d.date);

  
  return Array.from(groupedData, ([date, values]) => {

    const avgTemperature = d3.mean(values, (d) => +d.temperature) || 0; 
    const avgHumidity = d3.mean(values, (d) => +d.humidity) || 0; 

   
    const tempVariation = avgTemperature - referenceValueTemp2;
    const humidityVariation = avgHumidity - referenceValueHum2;

    
    return {
      date, 
      avgTemperature, 
      avgHumidity, 
      tempVariation,
      humidityVariation 
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
     
    },
    y: {
      label: "Value",
      grid: true 
    },
    width: width, 
    height: 500,
    color: {
      legend: true, 
    }
  });
}



function todosDias2(dailyAverages2, {width} = {}) {
  return Plot.plot({
    title: "Temperatura e Humidade Média",
    marks: [
    
      Plot.line(dailyAverages2, {
        x: "date",
        y: "avgTemperature",
        stroke: "steelblue",
        strokeWidth: 2,
        title: (d) => `Temperature: ${d.avgTemperature.toFixed(1)}°C`,
      }),
  
      Plot.line(dailyAverages2, {
        x: "date",
        y: "avgHumidity",
        stroke: "green",
        strokeWidth: 2,
        title: (d) => `Humidity: ${d.avgHumidity.toFixed(1)}%`,
      }),
     
      Plot.text(dailyAverages2, {
        x: "date",
        y: (d) => d.avgTemperature + (d.tempVariation > 0 ? 1.5 : -1.5),
        text: (d) => `${d.tempVariation > 0 ? "+" : ""}${d.tempVariation.toFixed(1)}°C`,
        fontSize: 12,
        fill: "steelblue",
        dx: 5,
      }),
   
      Plot.text(dailyAverages2, {
        x: "date",
        y: (d) => d.avgHumidity + (d.humidityVariation > 0 ? 2 : -2), 
        text: (d) => `${d.humidityVariation > 0 ? "+" : ""}${d.humidityVariation.toFixed(1)}%`, 
        fontSize: 12,
        fill: "green",
        dx: 5, 
      })
    ],
    x: {
      label: "Dia",
     
    },
    y: {
      label: "Value",
      grid: true, 
    },
    width: width, 
    height: 500,
    color: {
      legend: true, 
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
//tentativas de grafiocs interativos com o d3
function todosDiass(dailyAverages) {


  const margin = { top: 20, right: 30, bottom: 110, left: 40 },
        margin2 = { top: 430, right: 30, bottom: 30, left: 40 },
        width = 800 - margin.left - margin.right, 
        height = 500 - margin.top - margin.bottom,
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


  const svg = d3
    .create("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .style("background", "#ffffff");


  svg
    .append("defs")
    .append("clipPath")
    .attr("id", "clip")
    .append("rect")
    .attr("width", width)
    .attr("height", height);

  const focus = svg
    .append("g")
    .attr("class", "focus")
    .attr("transform", `translate(${margin.left},${margin.top})`);


  const context = svg
    .append("g")
    .attr("class", "context")
    .attr("transform", `translate(${margin2.left},${margin2.top})`);


  const data = dailyAverages;

  // Update domains based on avgTemperature3 and avgHumidity3
  x.domain(d3.extent(data, (d) => d.month));
  y.domain([0, d3.max(data, (d) => d.avgTemperature3)]); // Temperature on left y-axis
  y2.domain([0, d3.max(data, (d) => d.avgHumidity3)]); // Humidity on right y-axis
  x2.domain(x.domain());


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

 
  context.append("g").attr("class", "brush").call(brush);

  svg
    .append("rect")
    .attr("class", "zoom")
    .attr("width", width)
    .attr("height", height)
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .style("fill", "none")
    .style("pointer-events", "all") 
    .call(zoom);

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


  function zoomed(event) {
    const transform = event.transform;
    const newX = transform.rescaleX(x2);
    x.domain(newX.domain());
    updateFocus();
    context
      .select(".brush")
      .call(brush.move, x.range().map(transform.invertX, transform));
  }


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


  function brushed(event) {
    if (event.selection) {
      const [x0, x1] = event.selection.map(x2.invert);
      x.domain([x0, x1]);
      updateFocus();
    }
  }


  return svg.node();
}

```


```js

//mais uma tentativa 

function todosDiasss(dailyAverages, { width }) {

  const margin = { top: 20, right: 30, bottom: 110, left: 40 },
        margin2 = { top: 430, right: 30, bottom: 30, left: 40 },
        height = 500 - margin.top - margin.bottom,
        height2 = 500 - margin2.top - margin2.bottom;


  const svg = d3
    .create("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .style("background", "#ffffff");

  svg
    .append("defs")
    .append("clipPath")
    .attr("id", "clip")
    .append("rect")
    .attr("width", width)
    .attr("height", height);


  const focus = svg
    .append("g")
    .attr("class", "focus")
    .attr("transform", translate(${margin.left},${margin.top}));

 
  const context = svg
    .append("g")
    .attr("class", "context")
    .attr("transform", translate(${margin2.left},${margin2.top}));

  const x = d3.scaleTime().range([0, width]);
  const x2 = d3.scaleTime().range([0, width]);
  const y = d3.scaleLinear().range([height, 0]);
  const y2 = d3.scaleLinear().range([height2, 0]);


  const xAxis = d3.axisBottom(x);
  const xAxis2 = d3.axisBottom(x2);
  const yAxis = d3.axisLeft(y);

  const data = montlyAverages;


  x.domain(d3.extent(data, (d) => d.month));
  y.domain([0, d3.max(data, (d) => d.avgTemperature3)]);
  x2.domain(x.domain());
  y2.domain(y.domain());


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


  context.append("g").attr("class", "brush").call(brush);

  svg
    .append("rect")
    .attr("class", "zoom")
    .attr("width", width)
    .attr("height", height)
    .attr("transform", translate(${margin.left},${margin.top}))
    .style("fill", "none")
    .style("pointer-events", "all") 
    .call(zoom);

 
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
        .defined((d) => d.avgTemperature3) 
        .x((d) => x(d.month))
        .y((d) => y(d.avgTemperature3))
    );

  
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

    .attr("stroke", "red") 
    .attr("stroke-width", 1) 
    .attr("stroke-dasharray", "4 2"); 

  context
    .append("g")
    .attr("class", "axis axis--x")
    .attr("transform", translate(0,${height2}))
    .call(xAxis2);

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

  function brushed(event) {
    if (event.selection) {
      const [x0, x1] = event.selection.map(x2.invert);
      x.domain([x0, x1]);
      updateFocus();
    }
  }


  function zoomed(event) {
    const transform = event.transform;
    const newX = transform.rescaleX(x2);
    x.domain(newX.domain());
    updateFocus();
    context
      .select(".brush")
      .call(brush.move, x.range().map(transform.invertX, transform));
  }


  return svg.node();
}
```



```js

//outra tentativa 

function createChart() {

  
  


  const marginTop = 20, marginRight = 20, marginBottom = 30, marginLeft = 30;
  const width = 928, height = 500;


  const x = d3.scaleUtc()
    .domain(d3.extent(dailyAverages, (d) => d.date))
    .range([marginLeft, width - marginRight]);

  const y = d3.scaleLinear()
    .domain([0, d3.max(dailyAverages, (d) => Math.max(d.avgTemperature2, d.avgHumidity2))])
    .nice()
    .range([height - marginBottom, marginTop]);


  const xAxis = (g, x) => g
    .call(d3.axisBottom(x).ticks(width / 80).tickSizeOuter(0));


  const area = (data, x) => d3.area()
    .curve(d3.curveStepAfter)
    .x((d) => x(d.date))
    .y0(y(0))
    .y1((d) => y(d.avgTemperature2))  
    (data);


  const zoom = d3.zoom()
    .scaleExtent([1, 32])
    .extent([[marginLeft, 0], [width - marginRight, height]])
    .translateExtent([[marginLeft, -Infinity], [width - marginRight, Infinity]])
    .on("zoom", zoomed);


  const svg = d3.create("svg")
    .attr("viewBox", [0, 0, width, height])
    .attr("width", width)
    .attr("height", height)
    .attr("style", "max-width: 100%; height: auto;");

  const clip = svg.append("defs")
    .append("clipPath")
    .attr("id", "clip")
    .append("rect")
    .attr("x", marginLeft)
    .attr("y", marginTop)
    .attr("width", width - marginLeft - marginRight)
    .attr("height", height - marginTop - marginBottom);


  const path = svg.append("path")
    .attr("clip-path", "url(#clip)") 
    .attr("fill", "steelblue")
    .attr("d", area(dailyAverages, x));


  const gx = svg.append("g")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(xAxis, x);


  svg.append("g")
    .attr("transform", `translate(${marginLeft},0)`)
    .call(d3.axisLeft(y).ticks(null, "s"))
    .call((g) => g.select(".domain").remove())
    .call((g) => g.select(".tick:last-of-type text").clone()
      .attr("x", 3)
      .attr("text-anchor", "start")
      .attr("font-weight", "bold")
      .text("Temperature (°C)")); 


  function zoomed(event) {
    const xz = event.transform.rescaleX(x);
    path.attr("d", area(dailyAverages, xz));
    gx.call(xAxis, xz);
  }


  svg.call(zoom)
    .transition()
    .duration(750)
    .call(zoom.scaleTo, 4, [x(Date.UTC(2024, 0, 1)), 0]); // Zoom into a specific range (January 2024)


  document.body.appendChild(svg.node());
}

```













