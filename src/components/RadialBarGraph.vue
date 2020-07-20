<template>
    
    <svg :width="svgWidth + 'px'" :height="svgHeight + 'px'" :id="id"></svg>
</template>

<script>
import * as d3 from "d3"

export default {

    name:"RadialBarGraph",
    props:{
        width: {
            type: Number,
            default: 300
        },
        values: {
            type: Array,
            required: true
        },
        labels: {
            type: Array,
            required: true
        },
        colorScale: {
            type: Array,
            default: () => ["#4FC3F7","#0277BD"]
        }
    },
    data: function(){
        return {
            svgWidth: this.width,
            svgHeight: this.width,
            id: "radialbargraph" + parseInt(Math.random()*100),
            cX: this.width/2,
            cY: this.width/2,
            graphValues: this.values,
            graphLabels: this.labels,
            graphColorScale: this.colorScale
        }
    },
    mounted(){
        this.drawChart();
        d3.select("body")
            .append("div")
            .attr("id",this.id+"tooltip")
            .style("opacity", 0)
            .attr("class", "tooltip")
            .style("background-color", "white")
            .style("font-family","sans-serif")
            .style("border", "1px solid gray")
            .style("padding", "5px")
            .style("position","absolute")
            .style("z-index","10")
            .text("this is a test");
    },

    methods: {
        drawChart(){

            const radialScale = d3.scaleLinear().domain([0,Math.max(...this.graphValues)*1.1]).range([0,(Math.PI * 1.5)]);
            const yScale = d3.scaleBand().paddingInner(0.2).domain(this.graphLabels).range([this.cY*0.2,(this.cY)*0.95]);
            const colorScale = d3.scaleLinear().domain([0,Math.max(...this.graphValues)]).range(this.graphColorScale);

            const svg = d3.select("#" + this.id); 
            svg.selectAll("*").remove();

            const group = svg.append("g").attr("transform",`translate(${this.cX},${this.cY})`);

            let sum = d3.sum(this.graphValues);

            this.graphValues.forEach((val,i) => {
                    group.append("path").attr(
                        "d",
                        d3.arc().innerRadius(yScale(this.graphLabels[i])).outerRadius(yScale(this.graphLabels[i]) + yScale.bandwidth()).startAngle(0).endAngle(1.5*Math.PI)
                    ).attr("fill","#f1f1f1");

                    let valGroup = group.append('g').attr("class","entry");
                    valGroup.append("text")
                            .attr("dx",-10)
                            .attr("dy",yScale(this.graphLabels[this.graphLabels.length - i - 1]) - this.cY - yScale.bandwidth())
                            .attr("text-anchor","end")
                            .attr("font-family","sans-serif")
                            .attr("fill",colorScale(val))
                            .attr("font-size",yScale.bandwidth())
                            .text(this.graphLabels[i] + ": " + val);
                    
                    valGroup.append("path").attr(
                        "d",
                        d3.arc().innerRadius(yScale(this.graphLabels[i])).outerRadius(yScale(this.graphLabels[i]) + yScale.bandwidth()).startAngle(0).endAngle(radialScale(val))
                    ).attr("fill", colorScale(val));  

                    valGroup.on("mouseover",() => {
                        let tooltipText = this.graphLabels[i] + ": " + val + " (" + (val/sum * 100).toFixed(2) + "%)";
                        d3.select("#"+this.id+"tooltip").style("opacity",1).style("top",(d3.event.pageY + 10) + "px").style("left",(d3.event.pageX + 10) + "px").text(tooltipText);
                    });
                    valGroup.on("mouseleave",() => {
                        d3.select("#"+this.id+"tooltip").style("opacity",0);
                    });
                    valGroup.on("mousemove",() => {
                        let tooltipText = this.graphLabels[i] + ": " + val + " (" + (val/sum * 100).toFixed(2) + "%)";
                        d3.select("#"+this.id+"tooltip").style("top",(d3.event.pageY + 10) + "px").style("left",(d3.event.pageX + 10) + "px").text(tooltipText);
                    });
                }
            );

        }
    },

    watch:{
        labels(newVal){
            this.graphLabels = newVal;
            this.drawChart();
        },
        values(newVal){
            this.graphValues = newVal;
            this.drawChart();
        }
    }
}
</script>