---
theme: dashboard
title: Level 1
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
  d.day = date.getDate();
});

const filteredData = temperature.filter(d => d.year !== 1996);
```

```js
const dataByYearMonth = d3.group(filteredData, d => d.year, d => d.month);
```

```js
const gridData = [];
dataByYearMonth.forEach((months, year) => {
    months.forEach((temps, month) => {
        gridData.push({ 
            year, 
            month, 
            max_temperatures: temps.map(d => ({ day: d.day, temp: d.max_temperature })),
            min_temperatures: temps.map(d => ({ day: d.day, temp: d.min_temperature }))
        });
    });
});
```

```js
function colorScale(temp, showMaxTemp) {
  if (showMaxTemp) {
    // Red-orange color scale for max temperature
    return d3.scaleSequential(d3.interpolateOrRd)
      .domain([0, 40])(temp);
  } else {
    // Blue color scale for min temperature
    return d3.scaleSequential(d3.interpolateBlues)
      .domain([0, 40])(temp); 
  }
}
```

```js
let showMaxTemp = true;
```

```js
function part1(data, showMaxTemp, { width } = {}) {
  return Plot.plot({
    width,
    height: 700,
    x: {
      label: "Year",
      domain: d3.sort([...new Set(data.map(d => d.year))]),
      nice: false,
      band: true
    },
    y: {
      label: "Month",
      domain: ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"],
      ticks: 12,
      ticksize: 0,
      band: true
    },
    marks: [
      Plot.rect(gridData, {
        x: "year", y: "month",
        fill: d => colorScale(showMaxTemp ? d3.max(d.max_temperatures, t => t.temp): d3.min(d.min_temperatures, t => t.temp), showMaxTemp), // Toggle between min and max temperature
        title: d => `${d.month} ${d.year}\nMax Temp: ${d3.max(d.max_temperatures, t => t.temp)}°C\nMin Temp: ${d3.min(d.min_temperatures, t => t.temp)}°C`
      }),
      Plot.ruleY([0])
    ]
  });
}

```

```js
let showMaxTemp = true;

document.getElementById("toggle-button-1").addEventListener("click", function() {
  showMaxTemp = !showMaxTemp;
  document.getElementById("plot-container-1").innerHTML = "";
  document.getElementById("plot-container-1").appendChild(part1(gridData, showMaxTemp, { width }));
  this.textContent = showMaxTemp ? "Show Min Temperature" : "Show Max Temperature";
});
```

<div class="grid grid-cols-1">
  <div class="card">
    <div class="card-header">
      Level 1
    </div>
    <button class="button" id="toggle-button-1">
      Show Min Temperature
    </button>
    <div id="plot-container-1">
      ${resize((width) => part1(gridData, true, {width}))}
    </div>
  </div>
</div>

<style>
.button {
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

.button:hover {
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