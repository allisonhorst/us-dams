# By state

<!--
TODOs: 

- Consistent color schemes for primaryPurpose, and extend to > 10 colors
- Chart that shows hazard risk *and* status information? Not sure what this looks like. Maybe a sunburst?
- Clean up table
- Allow user to scroll up beeswarm
- Toggle to show either condition or primary purpose?
- Check beeswarm: those with missing inspection date don't show up, severely messing with the percentages and how it aligns with the beeswarm. Check Montana for an example
-->

```js
const dams = FileAttachment("data/dam-simple.csv").csv({typed: true});

const fips = FileAttachment("data/county_fips_master.csv").csv({typed: true});

const capitals = FileAttachment("data/us-state-capitals.csv").csv({typed: true});

const purposes = [
  "Water Supply",
  "Irrigation",
  "Recreation",
  "Hydroelectric",
  "Flood Risk Reduction",
  "Fish and Wildlife Pond",
  "Fire Protection, Stock, Or Small Fish Pond",
  "Debris Control",
  "Tailings",
  "Grade Stabilization",
  "Navigation",
  "Other",
  "NA"];

const purposeColors = [
  "#4269d0", // blue
  "#97bbf5", // light blue
  "#efb118", // orange
  "#ff725c", // red
  "#9498a0",  // gray
  "#6cc5b0", // cyan
  "#3ca951", // green
  "#9c6b4e", // brown
  "#ff8ab7", // pink
  "#a463f2", // purple
  "gray",
  "black",
  "orchid"
];
```

```js
const fipsSelectedState = fips.filter(d => d.state_name == pickState).map(d => d.fips);

const capitalSelectedState = capitals.filter(d => d.name == pickState);
```

```js
const usCounties = await FileAttachment("./data/us-counties-10m.json").json();
```

```js
const states = topojson.feature(usCounties, usCounties.objects.states).features;
```

```js
const counties = topojson.feature(usCounties, usCounties.objects.counties).features.map(d => ({...d, fips: +d.id}));
```

```js
const selectedState = states.filter(d => d.properties.name === pickState);

const selectedStateCounties = counties.filter(d => fipsSelectedState.includes(d.fips));
```

```js
const colors = ["#9498a0", "#ff725c", "#efb118", "#97bbf5", "#4269d0"];
const conditions = ["Not available", "Poor", "Unsatisfactory", "Satisfactory", "Fair"];
```

```js
const damsSelectedState = dams.filter(d => d.state == pickState);
```

```js
function stateMap(width, height) {
    return Plot.plot({
    height: height - 30,
    marginTop: 0,
    marginBottom: 0,
    color: {legend: true, domain: conditions, range: colors},
    projection: {type: "albers-usa", domain: selectedState[0].geometry},
    r: {range: [2, 15]},
    marks: [
        Plot.geo(selectedState, {fill: "#ccc", opacity: 0.3}),
        Plot.geo(selectedStateCounties, {stroke: "white", strokeWidth: 1, opacity: 0.3}),
        Plot.geo(selectedState, {stroke: "#ccc", strokeWidth: 1}),
        Plot.dot(damsSelectedState, {x: "longitude", y: "latitude", r: "maxStorageAcreFt", fill: "conditionAssessment", tip: true, title: d => `Dam name: ${d.name}\nYear completed: ${d.yearCompleted}\nMax storage: ${d.maxStorageAcreFt} acre-ft\nPrimary purpose: ${d.primaryPurpose}\nCondition: ${d.conditionAssessment}`}),
        Plot.dot(capitalSelectedState, {x: "longitude", y: "latitude", fill: "#ff2272", r: 6, stroke: "white", strokeWidth: 1, symbol: "star"}),
        Plot.text(capitalSelectedState, {x: "longitude", y: "latitude", text: "description", fill: "black", dy: -18, fontWeight: 400, fontSize: 14, stroke: "white", strokeWidth: 2})
    ]
});
}
```

```js
const pickState = view(Inputs.select(dams.filter(d => d.state != "Guam" & d.state != "Puerto Rico").map(d => d.state), {multiple: false, label: "Pick a state:", unique: true, sort: true, value: "Massachusetts"}));
```

<div class="grid grid-cols-3 grid-rows-3">
<div class="card grid-colspan-1 grid-rowspan-1">Of ${d3.format(",")(damsSelectedState.length)} NID recorded dams in ${pickState}, <b>${d3.format(",")(damsSelectedState.filter(d => d.conditionAssessment == "Poor").length)} ${damsSelectedState.filter(d => d.conditionAssessment == "Poor").length == 1 ? "is" : "are"} in poor condition</b>. Of those in poor condition, <b>${d3.format(",")(damsSelectedState.filter(d => d.conditionAssessment == "Poor" && d.hazardPotential == "High").length)} ${damsSelectedState.filter(d => d.conditionAssessment == "Poor" && d.hazardPotential == "High").length == 1 ? "has" : "have"} high hazard potential</b>, where "downstream flooding would likely result in loss of human life."</div>
<div class="card grid-colspan-2 grid-rowspan-1">${resize((width, height) => stackedBarChart(width, height))}</div>
<div class="card grid-colspan-2 grid-rowspan-3">${resize((width, height) => stateMap(width, height))}</div>
<div class="card grid-colspan-1 grid-rowspan-1">Apples!</div>
<div class="card grid-colspan-1 grid-rowspan-1">Apples!</div>
<div class="card grid-colspan-1 grid-rowspan-1">Apples!</div>
</div>

<!-- County FIPS codes from: https://github.com/kjhealy/fips-codes/blob/master/county_fips_master.csv -->

```js
const toggleBeeswarm = view(Inputs.radio(["primary purpose", "hazard potential"], {label: "Color by dam: ", value: "primary purpose"}));
```

```js
const lastInspectionBeeswarm = Plot.plot({
  width,
  height: 300,
  color: {legend: true}, //, domain: purposes, range: purposeColors},
  title: "Years since last inspection",
  subtitle: `${d3.format(",")(damsSelectedState.filter(d => d.yearsSinceInspection != "NA").length)} out of ${d3.format(",")(damsSelectedState.length)} dams in ${pickState} (${d3.format(".3s")(100 * damsSelectedState.filter(d => d.yearsSinceInspection != "NA").length / damsSelectedState.length)}%) are represented in this chart; those missing an inspection date are not shown. Based on the dams with a reported last inspection date, at least ${d3.format(".1f")(100 * damsSelectedState.filter(d => d.yearsSinceInspection <=10).length / damsSelectedState.length)}% of all ${pickState} dams have been inspected within past 10 years, and at least ${d3.format(".1f")(100 * damsSelectedState.filter(d => d.yearsSinceInspection <=10 && d.hazardPotential == "High").length / damsSelectedState.filter(d => d.hazardPotential == "High").length)}% of all ${pickState} dams considered to have High hazard potential have been inspected within past 10 years.`,
  caption: `Years since last inspection for individual dams in ${pickState}. Each dot represents a single dam; size is representative of maximum storage capacity. Dams toward the right on the chart have been inspected more recently.`,
  r: {range: [2, 20]},
  x: {tickFormat: "0f", reverse: true, label: "Years since last inspection"},
  marks: [
    Plot.frame({insetPadding: 5}),
    Plot.dot(damsSelectedState,
    Plot.dodgeY({x: d => +d.yearsSinceInspection, r: "maxStorageAcreFt", fill: toggleBeeswarm == "primary purpose" ? "primaryPurpose" : "hazardPotential", tip: true, stroke: "white", strokeWidth: 0.5}))
    ]
});
```

```js
const xScale = Plot.scale({ x: { domain: d3.extent(damsSelectedState, d => +d.yearsSinceInspection), label: "↑ y" }, stroke: "red"});
```

<div class="card">
${display(lastInspectionBeeswarm)}
</div>

```js
display(Inputs.table(damsSelectedState));
```

```js
// Copyright 2021-2023 Observable, Inc.
// Released under the ISC license.
// https://observablehq.com/@d3/sunburst
function Sunburst(data, {
  path, 
  id = Array.isArray(data) ? d => d.id : null, 
  parentId = Array.isArray(data) ? d => d.parentId : null, 
  children, 
  value,
  sort = (a, b) => d3.descending(a.value, b.value), 
  label, 
  title, 
  link, 
  linkTarget = "_blank", 
  width = 640, 
  height = 400, 
  margin = 1, 
  marginTop = margin, 
  marginRight = margin, 
  marginBottom = margin, 
  marginLeft = margin, 
  padding = 1, 
  startAngle = 0, 
  endAngle = 2 * Math.PI, 
  radius = Math.min(width - marginLeft - marginRight, height - marginTop - marginBottom) / 2, 
  color = d3.interpolateRainbow, 
  fill = "#ccc", 
  fillOpacity = 0.6,
} = {}) {

  const root = path != null ? d3.stratify().path(path)(data)
      : id != null || parentId != null ? d3.stratify().id(id).parentId(parentId)(data)
      : d3.hierarchy(data, children);

  value == null ? root.count() : root.sum(d => Math.max(0, value(d)));

  if (sort != null) root.sort(sort);

  d3.partition().size([endAngle - startAngle, radius])(root);

  // Construct a color scale.
  if (color != null) {
    color = d3.scaleSequential([0, root.children.length], color).unknown(fill);
    root.children.forEach((child, i) => child.index = i);
  }

  // Construct an arc generator.
  const arc = d3.arc()
      .startAngle(d => d.x0 + startAngle)
      .endAngle(d => d.x1 + startAngle)
      .padAngle(d => Math.min((d.x1 - d.x0) / 2, 2 * padding / radius))
      .padRadius(radius / 2)
      .innerRadius(d => d.y0)
      .outerRadius(d => d.y1 - padding);

  const svg = d3.create("svg")
      .attr("viewBox", [
        marginRight - marginLeft - width / 2,
        marginBottom - marginTop - height / 2,
        width,
        height
      ])
      .attr("width", width)
      .attr("height", height)
      .attr("style", "max-width: 100%; height: auto; height: intrinsic;")
      .attr("font-family", "sans-serif")
      .attr("font-size", 10)
      .attr("text-anchor", "middle");

  const cell = svg
    .selectAll("a")
    .data(root.descendants())
    .join("a")
      .attr("xlink:href", link == null ? null : d => link(d.data, d))
      .attr("target", link == null ? null : linkTarget);

  cell.append("path")
      .attr("d", arc)
      .attr("fill", color ? d => color(d.ancestors().reverse()[1]?.index) : fill)
      .attr("fill-opacity", fillOpacity);

  if (label != null) cell
    .filter(d => (d.y0 + d.y1) / 2 * (d.x1 - d.x0) > 10)
    .append("text")
      .attr("transform", d => {
        if (!d.depth) return;
        const x = ((d.x0 + d.x1) / 2 + startAngle) * 180 / Math.PI;
        const y = (d.y0 + d.y1) / 2;
        return `rotate(${x - 90}) translate(${y},0) rotate(${x < 180 ? 0 : 180})`;
      })
      .attr("dy", "0.32em")
      .text(d => label(d.data, d));

  if (title != null) cell.append("title")
      .text(d => title(d.data, d));

  return svg.node();
}
```

```js
display(Sunburst);
```

```js
const hazardConditionCounts = {
  group = (data, [name, ...path]) => d3.groups(data, d => d[name]).map(([value, rows]) => {
      returnObject = {};
      returnObject["name"] = value; 
      if(path.length) {
        returnObject["children"] = group(rows, path);
      } else {
        returnObject["value"] = rows.length;
      }
      return returnObject; 
    });
  
  return {name: "damRisk", children: group(damsSelectedState, ["conditionAssessment", "hazardPotential"]) };
};
```

```js
display(hazardConditionCounts);
```

```js
const sunburstRisk = Sunburst(conditionHazardCounts, {value: d => d.damCount,
  label: "hello",
  title: "blah"
  width: 1152,
  height: 1152})
  ```

  ```js
  display(sunburstRisk);
  ```

  ```js
  display(d3.rollup(dams, d => d.length, v => v.primaryPurpose));
  ```

```js
const conditionCounts = d3.flatRollup(damsSelectedState, d => d.length, v => v.conditionAssessment).map(([condition, count]) => ({condition, count}));
```

```js
display(conditionCounts);
```

  ```js
  function stackedBarChart(width, height) {
    return Plot.plot({
    width,
    title: `${pickState} dam conditions`,
    height: height - 60,
    //marginTop: 40,
    //marginBottom: 30,
    color: {domain: conditions, range: colors, legend: true},
    x: {label: "Number of dams"},
    marks: [
        Plot.barX(conditionCounts, Plot.stackX({x: "count", fill: "condition", order: conditions, tip: true})),
        //Plot.textX(conditionCounts, Plot.stackX({x: "count", text: "condition", z: "condition", order: conditions, inset: 0.5, dy: -20, rotate: -30, textAnchor: "start"})),
    ]
  });
  }
  ```

  ```js
  display(stackedBarChart);
  ```