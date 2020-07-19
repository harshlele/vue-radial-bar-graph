<template>
    <svg :width="svgWidth + 'px'" :height="svgHeight + 'px'" :id="id"></svg>
</template>

<script>
import * as d3 from "d3"

export default {

    name:"RadialBarGraph",
    props:{
        width: Number,
        values: {
            type: Array,
            required: true
        },
        labels: {
            type: Array,
            required: true
        },
        colorScale: Array
    },
    data: function(){
        return {
            svgWidth: this.width ? this.width : 300,
            svgHeight: this.width ? this.width : 300,
            id: "radialbargraph" + parseInt(Math.random()*100),
            cX: this.width ? (this.width/2) : 150,
            cY: this.width ? (this.width/2) : 150,
            graphValues: this.values,
            graphLabels: this.labels,
            graphColorScale: this.colorScale ? this.colorScale : ["#4FC3F7","#0277BD"]
        }
    },
    mounted(){

        let radialScale = d3.scaleLinear().domain([0,Math.max(...this.graphValues)*1.1]).range([0,(Math.PI * 1.5)]);
        let yScale = d3.scaleBand().paddingInner(0.2).domain(this.graphLabels).range([this.cY*0.2,(this.cY)*0.95]);
        let colorScale = d3.scaleLinear().domain([0,Math.max(...this.graphValues)]).range(this.graphColorScale);

        const svg = d3.select("#" + this.id); 
        
        const group = svg.append("g").attr("transform",`translate(${this.cX},${this.cY})`);

        this.graphValues.forEach((val,i) => {
                group.append("path").attr(
                    "d",
                    d3.arc().innerRadius(yScale(this.graphLabels[i])).outerRadius(yScale(this.graphLabels[i]) + yScale.bandwidth()).startAngle(0).endAngle(1.5*Math.PI)
                ).attr("fill","#f1f1f1");

                group.append("path").attr(
                    "d",
                    d3.arc().innerRadius(yScale(this.graphLabels[i])).outerRadius(yScale(this.graphLabels[i]) + yScale.bandwidth()).startAngle(0).endAngle(radialScale(val))
                ).attr("fill", colorScale(val));  
            }
        );
        

    }   
}
</script>