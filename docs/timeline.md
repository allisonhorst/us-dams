# Timeline

```js
const dams = FileAttachment("data/dam-simple.csv").csv({typed: true});
```

```js
const newDamsByYear = d3.flatRollup(dams, d => d.length, v => v.yearCompleted).map(([year, newDams]) => ({year, newDams})).filter(d => d.year != "NA");
```

```js
const damsUnknownYearCount = dams.filter(d => d.yearCompleted == "NA").length
```

```js
const newDams = d3.sort(newDamsByYear, d => d.year);
```

```js
const cumulativeDams = d3.cumsum(newDams.map(d => d.newDams));
```

```js
// This adds a baseline that is the number of dams w/ NA completion year (but that should still be included in the totals)

const totalCumulativeDams = newDams.map((d,i) => ({year: d.year, cumulativeDams: cumulativeDams[i] + damsUnknownYearCount}));
```

```js
display(totalCumulativeDams);
```

```js
function newDamsChart(width) {
return pickTimeline == "New dams" ? 
  Plot.plot({
    width,
    title: "Dams completed in U.S.",
    caption: `The ${d3.format(",")(damsUnknownYearCount)} dams with an unknown or unreported completion year are not represented in this chart.`,
    marginLeft: 50,
    y: {tickFormat: ",f", label: pickTimeline == "New dams" ? "New dams completed" : "Cumulative dams", grid: true},
    x: {tickFormat: "y", label: "Year"},
    marks: [
       Plot.lineY(newDams, {x: "year", y: d => d.newDams, tip: true, curve: "step", fill: "#ccc", fillOpacity: 0.4, stroke: "#ccc"})]
   }) :
   Plot.plot({
    width,
    title: "Dams completed in U.S.",
    caption: `The baseline value of ${d3.format(",")(damsUnknownYearCount)} dams accounts for dams with an unknown or unreported completion year.`,
    y: {domain: [0, 93000], grid: true, label: "Total cumulative dams"},
    x: {tickFormat: "y", label: "Year"},
    marginLeft: 50,
    marks: [
      Plot.areaY(totalCumulativeDams, {x: "year", y: "cumulativeDams", fill: "#ccc", opacity: 0.4}),
      Plot.lineY(totalCumulativeDams, {x: "year", y: "cumulativeDams", stroke: "#ccc", tip: true}),
      Plot.ruleY([d3.max(totalCumulativeDams, d => d.cumulativeDams)], {strokeDasharray: [4,4]}),
      Plot.ruleY([damsUnknownYearCount], {strokeDasharray: [4,4]})
    ]});
}
```

```js
const pickTimeline = view(Inputs.radio(["New dams", "Cumulative dams"], {label: "Choose timeline:", value: "Cumulative dams"}));
```

<div class="grid grid-cols-2 grid-rows-2">
<div class="card grid-colspan-1 grid-rowspan-2">${resize(width => newDamsChart(width))}</div>
<div class="card grid-colspan-1 grid-rowspan-1">Something</div>
<div class="card grid-colspan-1 grid-rowspan-1">Something</div>
</div>

