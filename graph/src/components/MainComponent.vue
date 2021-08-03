<template>
  <div id="dat1"></div>
</template>
<script>
import * as d3 from "d3";
import axios from "axios";
export default {
  data() {
    return {
      chartData: null,
    };
  },
  mounted() {
    axios.get("http://localhost:8080/z.json").then((response) => {
      console.log("chartResponse is : ", response);
      if (response && response.data && response.data.length) {
        this.renderChart(response.data);
      }
    });
  },
  methods: {
    renderChart(data) {
      const margin = { top: 40, bottom: 20, left: 80, right: 30 },
        width = 1500 - margin.left - margin.right,
        height = 500 - margin.top - margin.bottom;

      let companies = [];
      let roles = [];
      let months = [];

      data.forEach((d) => {
        companies.push(d.company);

        d.roles.forEach((d1) => {
          roles.push(d1.role);

          d1.months.forEach((d3) => {
            months.push(d3.month.split("-")[0]);
          });
        });
      });

      const svgWidth = width + margin.left + margin.right;
      const svg = d3
        .select("#dat1")
        .append("svg")
        .attr("width", svgWidth + 100)
        .attr("height", height + margin.top + 10 + margin.bottom)
        .append("g")
        .attr(
          "transform",
          "translate(" + margin.left + "," + (margin.top + 10) + ")"
        );

      const x0 = d3
        .scaleBand()
        .domain(companies)
        .range([0, svgWidth - 100]);
      svg
        .append("g")
        .attr("transform", "translate(" + 0 + "," + 0 + ")")
        .call(d3.axisTop(x0).tickSizeOuter(0));

      svg
        .selectAll("gridlines")
        .data(companies)
        .enter()
        .append("line")
        .attr("x1", (d) => x0(d) + x0.bandwidth())
        .attr("y1", 0)
        .attr("x2", (d) => x0(d) + x0.bandwidth())
        .attr("y2", height)
        .attr("stroke-width", "1px")
        .attr("stroke", "black");

      const y1 = d3.scaleBand().domain(roles).range([0, height]);

      svg
        .append("g")
        .attr("transform", "translate(" + 0 + "," + 0 + ")")
        .call(d3.axisLeft(y1).tickSizeOuter(0));

      const x1 = d3.scaleBand().domain(months).range([0, x0.bandwidth()]);

      // function make_x_gridlines() {
      //   return d3.axisBottom(x0);
      // }

      function make_y_gridlines() {
        return d3.axisLeft(y1);
      }

      svg
        .append("g")
        .data(data)
        .enter()
        .append("text")
        .attr("x", (d) => x1(d.months))
        .attr("y", (d) => y1(d.roles))
        .style("fill", "blue");

      svg
        .append("text")
        .attr("text-anchor", "start")
        .attr("x", svgWidth / 2 - 80)
        .attr("y", -20)
        .attr("text-anchor", "middle")
        .attr("alignment-baseline", "middle")
        .attr("text", "left")
        .text("CHURN RISK GROUP");

      svg
        .append("text")
        .attr("text-anchor", "middle")
        .attr("transform", "rotate(-90)")
        .attr("x", -margin.top - height / 2 + 20)
        .attr("y", -margin.left + 10)
        .attr("alignment-baseline", "middle")
        .text("SENIORITY");

 

      svg
        .append("g")
        .attr("class", "grid")
        .call(
          make_y_gridlines()
            .tickSize(-svgWidth + 100)
            .tickFormat("")
        );
      svg
        .selectAll(".company")
        .data(data)
        .enter()
        .append("circle")
        .attr("class", "company")

        .attr("x", (d) => {
          console.log(d.company);
          return x0(d.company);
        })

        .attr("y", 80)
        .attr("transform", (d, i) => {
          const x = i * 40;
          return "translate(" + x + ",0)";
        })
        .attr("width", x0.bandwidth(5))
        .attr("height", 100)
        .attr("fill", "#0000ff");
      data.forEach((c) => {
        svg
          .append("g")
          .attr("transform", "translate(" + x0(c.company) + "," + height + ")")
          .call(d3.axisBottom(x1).tickSizeOuter(0));
        c.roles.forEach((r) => {
          r.months.forEach((m) => {
            if(m.value){
              svg
                .append("circle")
                .attr(
                  "cx",
                  x0(c.company) + x1(m.month.split("-")[0]) + x1.bandwidth() / 2
                )
  
                .attr("cy", y1(r.role) + y1.bandwidth() / 2)
                .attr("r", "20")
                .style("fill", "red");
              svg
                .append("text")
                .text(m.value)
                .attr(
                  "x",
                  x0(c.company) + x1(m.month.split("-")[0]) + x1.bandwidth() / 2
                )
                .attr("y", y1(r.role) + y1.bandwidth() / 2)
                .attr("text-anchor", "middle")
                .attr("alignment-baseline", "middle");
            }
          });
        });
      });
    },
  },
};
</script>
<style lang="scss" scoped>
#dat1 {
  position: relative;
  float: left;
  overflow-y: auto;
  overflow-x: auto;
  background-color: #dde0eb;
}
</style>