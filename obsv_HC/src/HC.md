# HC report

## Introduction

This report is about the Home Comfort project. The project is about the development of a system that can monitor the comfort of a home. The system will be able to monitor the temperature, humidity, and light levels in a home. The system will also be able to control the heating and cooling systems in the home. The system will be able to provide feedback to the user about the comfort level in the home.
<script src="https://d3js.org/d3.v7.min.js"></script>

```js
import {FileAttachment} from "observablehq:stdlib";
import * as Plot from "@observablehq/plot";
import * as d3 from "npm:d3"; // import everything as a namespace
import * as Inputs from "npm:@observablehq/inputs";
import { scaleSequential, scaleBand, select, axisLeft, axisTop, interpolateYlGnBu } from "d3";

const frames = await Promise.all([
    FileAttachment("data/02A-sgh02015d5c61cc.csv").csv({typed: true}),
    FileAttachment("data/03A-sgh0201a8c87da4.csv").csv({typed: true}),
    FileAttachment("data/05L-sgh020125bce03a.csv").csv({typed: true}),
    FileAttachment("data/08L-sgh02018fe9be2c.csv").csv({typed: true}),
    FileAttachment("data/10A-sgh0201f6cb55ed.csv").csv({typed: true}),
]);
```


```js
const selectedDataset = view(Inputs.select(
  [
    { label: "House Matos, Aveiro", value: frames[0] },
    { label: "House Barrancos, Aveiro", value: frames[1] },
    { label: "House Custódia, Lamego", value: frames[2] },
    { label: "House Ferradura, Lamego", value: frames[3] },
    { label: "House Salitre, Aveiro", value: frames[4] },
  ],
  {
    label: "Select House:", // Label for the dropdown
    format: (x) => x.label, // Display the house name
    value: (x) => x.value // Use the array cell name as the value
  }
));
```
<b>Casa selecionada é ${selectedDataset.label} <b>

```js
view(Inputs.table(selectedDataset.value, {
  layout: "auto",
}));
```

```js
const selectedYear = view(Inputs.select(
  [
    { label: "Whole period", value: "" },
    { label: "2019", value: 2019 },
    { label: "2020", value: 2020 }
  ],
  {
    label: "Select Year:",
    format: (x) => x.label,
    value: ""
  }
));
const selectedMonth = view(Inputs.select(
  [
    { label: "Whole year", value: "" },
    { label: "January", value: 1 },
    { label: "February", value: 2 },
    { label: "March", value: 3 },
    { label: "April", value: 4 },
    { label: "May", value: 5 },
    { label: "June", value: 6 },
    { label: "July", value: 7 },
    { label: "August", value: 8 },
    { label: "September", value: 9 },
    { label: "October", value: 10 },
    { label: "November", value: 11 },
    { label: "December", value: 12 }
  ],
  {
    label: "Select Month:",
    format: (x) => x.label,
    value: ""
  }
));

```


```js
const filteredData = selectedDataset.value.filter((d) => {
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
    return true; // Include all records
  }

  // If both year and month are selected, filter by both
  return d.year == selectedYear.value && d.month == selectedMonth.value;
});
```

```js
view(Inputs.table(filteredData, {
  layout: "auto",
}));
```

```js

const avgTemperaturePerDay = (() => {
  // Filter out records with missing or invalid temperature values
  const validData = filteredData.filter(
    (d) => d.temperature !== null && d.temperature !== undefined && !isNaN(+d.temperature)
  );

  // Group valid data by day
  const groupedByDay = d3.group(validData, (d) => {
    return new Date(d.year, d.month - 1, d.day); // Group by day
  });

  // Get the full date range
  const startDate = d3.min(filteredData, (d) => new Date(d.year, d.month - 1, d.day));
  const endDate = d3.max(filteredData, (d) => new Date(d.year, d.month - 1, d.day));

  // Generate all days within the range
  const allDays = [];
  for (let d = new Date(startDate); d <= endDate; d.setDate(d.getDate() + 1)) {
    allDays.push(new Date(d)); // Push each day as a Date object
  }

  // Calculate the average temperature for each day
  return allDays.map((day) => {
    const dayValues = groupedByDay.get(day.getTime()); // Get grouped data for the day
    return {
      date: new Date(day), // Ensure date is a Date object
      avgTemperature: dayValues ? d3.mean(dayValues, (d) => +d.temperature) : NaN, // NaN if no data
    };
  });
})();


// Step 7: Add Next-Day Data Check
// Enrich the array to include a flag indicating if the next day has data
const enrichedAvgTemperaturePerDay = avgTemperaturePerDay.map((d, i, arr) => {
  const nextDay = arr[i + 1]; // Look ahead to the next day
  return {
    ...d,
    hasNextDayData: !!nextDay && nextDay.avgTemperature !== null && nextDay.avgTemperature !== undefined,
    avgHumidity: d.humidity ? d3.mean(d.humidity) : NaN, // Add avgHumidity if not already present
  };
});

// Step 8: Plot the data
view(
  Plot.plot({
    marks: [
      Plot.line(enrichedAvgTemperaturePerDay, {
        x: "date",
        y: "avgTemperature",
        stroke: "steelblue",
        defined: (d) => {
          // Only plot if the current day and the next day both have data
          return d.avgTemperature !== null && d.hasNextDayData;
        },
      }),
    ],
    x: {
      label: "Day of the Month",
      tickFormat: d3.utcFormat("%b-%d"), // Format the x-axis labels for month and day
    },
    y: {
      label: "Average Temperature (°C)",
    },
    width: 800,
    height: 400,
  })
);

```

```js
view(Plot.plot({
  inset: 8,
  grid: true,
  y: {
    domain: [0, 40],
    label: "Temperature (°C)"
  },
  x: {
    // Dynamically set the x domain based on the actual data range
    domain: d3.extent(filteredData, (d) => +d.day), // Ensure day is treated as a number
    label: "Day of the Month",
    tickFormat: d3.format("d") // Format the x-axis as an integer
  },
  color: {
    domain: [0, 10, 15, 20, 25, 30],
    legend: true,
    type: "linear",
    range: ["darkblue", "blue", "lightblue", "yellow", "orange", "red"]
  },
  marks: [
    Plot.dot(filteredData, {
      x: "day",
      y: "temperature",
      fill: "temperature"
    })
  ]
}));

```
```js
view(Plot.plot({
  inset: 8,
  grid: true,
  y: {
    domain: [0, 100],
    label: "Humidity (%)"
  },
  x: {
    // Dynamically set the x domain based on the actual data range
    domain: d3.extent(filteredData, (d) => +d.day), // Ensure day is treated as a number
    label: "Day of the Month",
    tickFormat: d3.format("d") // Format the x-axis as an integer
  },
  color: {
    domain: [20, 40, 60, 80, 100],
    legend: true,
    type: "linear",
    range: ["red", "orange", "yellow", "lightblue", "blue","darkblue"]
  },
  marks: [
    Plot.dot(filteredData, {
      x: "day",
      y: "humidity",
      fill: "humidity"
    })
  ]
}));

```
