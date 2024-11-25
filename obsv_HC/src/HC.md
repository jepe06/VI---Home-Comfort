# HC report

## Introduction

This report is about the Home Comfort project. The project is about the development of a system that can monitor the comfort of a home. The system will be able to monitor the temperature, humidity, and light levels in a home. The system will also be able to control the heating and cooling systems in the home. The system will be able to provide feedback to the user about the comfort level in the home.

### Import csv data with js
<!-- Create a for loop to import and name the files in the folder data -->

```js
import {FileAttachment} from "observablehq:stdlib";

import * as d3 from "npm:d3"; // import everything as a namespace
const frames = [
    FileAttachment("data/02A-sgh02015d5c61cc.csv"),
    FileAttachment("data/03A-sgh0201a8c87da4.csv"),
    FileAttachment("data/05L-sgh020125bce03a.csv"),
    FileAttachment("data/08L-sgh02018fe9be2c.csv"),
    FileAttachment("data/10A-sgh0201f6cb55ed.csv"),
];

```

```js
const selectedDataset = view(Inputs.select(
  [
    { label: "Dataset A2", value: frames[0].name },
    { label: "Dataset A3", value: frames[1].name },
    { label: "Dataset L5", value: frames[2].name },
    { label: "Dataset L8", value: frames[3].name },
    { label: "Dataset A10", value: frames[4].name },
  ],
  {
    label: "Select House:", // Label for the dropdown
    format: (x) => x.label, // Display the house name
    value: (x) => x.value // Use the array cell name as the value
  }
));

const selectedYear = view(Inputs.select(
  [
    { label: "Whole period", value: "" },
    { label: "2019", value: 2019 },
    { label: "2020", value: 2020 },
    { label: "2021", value: 2021 }
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
const fileData = selectedDataset.value;
```

```js
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
});