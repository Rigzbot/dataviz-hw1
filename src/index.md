---
theme: dashboard
title: Homework 1
toc: false
---

# Homework 1

<!-- Load and transform the data -->

```js
const temperature = FileAttachment("data/temperature_daily.csv").csv({typed: true});
```

```js
const monthNames = {
  1: "Jan", 2: "Feb", 3: "Mar", 4: "Apr", 5: "May", 6: "Jun",
  7: "Jul", 8: "Aug", 9: "Sep", 10: "Oct", 11: "Nov", 12: "Dec"
};

const monthMap = {
  "Jan": 0, "Feb": 1, "Mar": 2, "Apr": 3, "May": 4, "Jun": 5,
  "Jul": 6, "Aug": 7, "Sep": 8, "Oct": 9, "Nov": 10, "Dec": 11
};

temperature.forEach(d => {
  const date = new Date(d.date);
  d.year = date.getFullYear();
  d.month = monthNames[date.getMonth() + 1];
});

const filteredData = temperature.filter(d => d.year !== 1996);
```

```js
const dataByYearMonth = d3.rollup(
  filteredData,
  v => ({
    max_temperature: d3.max(v, d => d.max_temperature),
    min_temperature: d3.min(v, d => d.min_temperature)
  }),
  d => d.year,
  d => d.month
);
```

```js
const gridData = [];
dataByYearMonth.forEach((months, year) => {
    months.forEach((temps, month) => {
        gridData.push({ 
            year, 
            month, 
            max_temperature: temps.max_temperature,
            min_temperature: temps.min_temperature // Now added
        });
    });
});
```

```js
function colorScale(temp, showMaxTemp) {
  if (showMaxTemp) {
    // Red-orange color scale for max temperature
    return d3.scaleSequential(d3.interpolateOrRd)
      .domain([0, 40])(temp); // Adjust the domain according to your temperature range
  } else {
    // Blue color scale for min temperature
    return d3.scaleSequential(d3.interpolateBlues)
      .domain([0, 40])(temp); // Adjust the domain according to your min temperature range
  }
}
```

```js
let showMaxTemp = true;
```

```js
function part1(data, showMaxTemp, {width} = {}) {
  return Plot.plot({
    // title: showMaxTemp ? "Year/Month Max Temperature Grid" : "Year/Month Min Temperature Grid",
    width,
    height: 700,
    x: {
      label: "Year",
      domain: d3.sort([...new Set(data.map(d => d.year))]), // Ensure all years are displayed
      nice: false, // Don't auto-nice to avoid skipping years
      band: true
    },
    y: {
      label: "Month",
      domain: ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"], // Month range 0-11 for January to December
      ticks: 12,
      ticksize:0,
      band: true
    },
    marks: [
      Plot.rect(gridData, {
        x: "year", y: "month",
        fill: (d) => colorScale(showMaxTemp ? d.max_temperature : d.min_temperature, showMaxTemp), // Toggle between max & min temp
        title: (d) => `${d.month} ${d.year}\nMax Temp: ${d.max_temperature}°C\nMin Temp: ${d.min_temperature}°C`
      }),
      Plot.ruleY([0])  // A rule to create a baseline for Y axis
    ]
  });
}
```

```js
let showMaxTemp = true;

document.getElementById("toggle-button").addEventListener("click", function() {
  showMaxTemp = !showMaxTemp; // Toggle flag
  document.getElementById("plot-container").innerHTML = ""; // Clear previous plot
  document.getElementById("plot-container").appendChild(part1(filteredData, showMaxTemp, {width})); // Re-render
  this.textContent = showMaxTemp ? "Show Min Temperature" : "Show Max Temperature"; // Update button text
});
```

<div class="grid grid-cols-1">
  <div class="card">
    <div class="card-header">
      Level 1
    </div>
    <button id="toggle-button">
      Show Min Temperature
    </button>
    <div id="plot-container">
      ${resize((width) => part1(filteredData, true, {width}))}
    </div>
  </div>
</div>

<style>
#toggle-button {
  padding: 10px 20px;
  background-color:rgb(39, 82, 85); /* Green background */
  color: white; /* White text */
  font-size: 16px; /* Font size */
  border: none; /* No border */
  border-radius: 5px; /* Rounded corners */
  cursor: pointer; /* Pointer cursor on hover */
  transition: background-color 0.3s ease; /* Smooth background color transition */
  margin: 0 auto; /* Center the button horizontally */
  display: block; /* Make the button a block element so it takes up full width */
}

#toggle-button:hover {
  background-color:rgb(7, 66, 63); /* Darker green when hovered */
}

.card-header {
  position: absolute;
  top: 10px; 
  left: 10px; 
  font-size: 36px;
  font-weight: bold;
  padding: 5px 10px;
}

.card {
  position: relative; 
  padding: 20px;
  border-radius: 8px;
  text-align: center;
}
</style>