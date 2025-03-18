---
theme: dashboard
title: Level 2
toc: false
---

# Homework 1

<!-- Load and transform the data -->

```js
const temperature = FileAttachment("data/temperature_daily.csv").csv({typed: true});
```

```js
const filteredData = temperature.filter(d => {
  const year = new Date(d.date).getFullYear();
  return year >= 2017 - 5;
});
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

filteredData.forEach(d => {
  const date = new Date(d.date);
  d.year = date.getFullYear();
  d.month = monthNames[date.getMonth() + 1];
  d.day = date.getDate();
});
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
// Define the gridData and add it to Plot as before
function part1(data, showMaxTemp, { width } = {}) {
  const cellWidth = width / data.length; // Dynamically set cell width based on grid size

  const plot = Plot.plot({
    width,
    height: 750,
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
      // Heatmap rects for temperature
      Plot.rect(gridData, {
        x: "year", y: "month",
        fill: d => colorScale(showMaxTemp ? d3.max(d.max_temperatures, t => t.temp) : d3.min(d.min_temperatures, t => t.temp), showMaxTemp),
        title: d => `${d.month} ${d.year}\nMax Temp: ${d3.max(d.max_temperatures, t => t.temp)}°C\nMin Temp: ${d3.min(d.min_temperatures, t => t.temp)}°C`,
      }),
      Plot.ruleY([0])
    ]
  });

  // After creating the plot, define cells and append the line chart for daily temperature variation
  const cells = d3.select(plot).selectAll(".mark-rect")
    .data(gridData)
    .enter()
    .append("g") // Grouping each cell to append the line chart
    .attr("class", "cell")
    .attr("transform", d => {
      const x = d3.scaleBand().domain(d3.sort([...new Set(data.map(d => d.year))])).range([0, width]);
      const y = d3.scaleBand().domain(["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]).range([0, 700]);

      return `translate(${x(d.year)}, ${y(d.month)})`;
    });

  // Append the path (line) chart to each cell for max temp
  cells.append("path")
    .attr("class", "temp-line max-line")
    .attr("d", d => {
      const dailyData = d.max_temperatures;
      const validData = dailyData.filter(point => !isNaN(point.temp) && !isNaN(point.day));
      return validData.length > 0 ? createLineChart(validData, cellWidth) : null;
    })
    .attr("fill", "none")
    .attr("stroke", "green")  // Green color for max temp line
    .attr("stroke-width", 2.5)
    .style("opacity", 1);

  // Append the path (line) chart to each cell for min temp
  cells.append("path")
    .attr("class", "temp-line min-line")
    .attr("d", d => {
      const dailyData = d.min_temperatures;
      const validData = dailyData.filter(point => !isNaN(point.temp) && !isNaN(point.day));
      return validData.length > 0 ? createLineChart(validData, cellWidth) : null;
    })
    .attr("fill", "none")
    .attr("stroke", "blue")  // Blue color for min temp line
    .attr("stroke-width", 2.5)
    .style("opacity", 1);

  // Create a legend for max and min temperature lines
  const legend = d3.select(plot).append("g")
    .attr("class", "legend")
    .attr("transform", "translate(20, 20)"); 

  // Add a rectangle for the Max Temp line
  legend.append("rect")
    .attr("width", 30)
    .attr("height", 30)
    .attr("fill", "green")
    .attr("x", 1050)
    .attr("y", 650);

  // Add text next to the Max Temp legend
  legend.append("text")
    .attr("x", 1150)
    .attr("y", 665)
    .attr("dy", ".35em")
    .text("Max Temp")
    .style("font-size", "20px");

  // Add a rectangle for the Min Temp line
  legend.append("rect")
    .attr("width", 30)
    .attr("height", 30)
    .attr("fill", "blue")
    .attr("x", 1050)
    .attr("y", 600);

  // Add text next to the Min Temp legend
  legend.append("text")
    .attr("x", 1150)
    .attr("y", 615)
    .attr("dy", ".35em")
    .text("Min Temp")
    .style("font-size", "20px");
  return plot;
}

// Function to create the line chart path for daily temperature variation
function createLineChart(dailyData) {
  const leftMargin = 10;
  // Define scales for X and Y axis based on the cell's width and height
  const xScale = d3.scaleBand()
    .domain(d3.range(1, 32))
    .range([0, 100]); 

  const yScale = d3.scaleLinear()
    .domain([0, 40]) 
    .range([100, 0]); 

  const lineGenerator = d3.line()
    .x(d => xScale(d.day))  // Position each data point based on the day
    .y(d => yScale(d.temp));  // Position based on the temperature

  return lineGenerator(dailyData);
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
      Level 2
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