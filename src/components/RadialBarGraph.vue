<template>
    <div :style="`width: ${width}px; height: ${width}px; text-align:center;`">
        <div class="title-text">{{title}}</div>
        <svg :width="svgWidth + 'px'" :height="svgHeight + 'px'" :id="id" style="margin: auto;"></svg>
    </div>
    
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
        },
        title:String
    },
    data: function(){
        return {
            svgWidth: this.width * 0.9,
            svgHeight: this.width * 0.9,
            id: "radialbargraph" + parseInt(Math.random()*100),
            cX: this.width * 0.45,
            cY: this.width * 0.45,
            graphValues: this.values,
            graphLabels: this.labels,
            graphColorScale: this.colorScale,
            yScale: null,
        }
    },
    computed:{
        headerFontSize(){
            return parseInt(this.yScale.bandwidth() * 1.2);
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
            this.yScale = d3.scaleBand().paddingInner(0.2).domain(this.graphLabels).range([this.cY*0.2,(this.cY)*0.95]);
            const colorScale = d3.scaleLinear().domain([0,Math.max(...this.graphValues)]).range(this.graphColorScale);

            const svg = d3.select("#" + this.id); 
            svg.selectAll("*").remove();

            const group = svg.append("g").attr("transform",`translate(${this.cX},${this.cY})`);

            let sum = d3.sum(this.graphValues);

            this.graphValues.forEach((val,i) => {
                    group.append("path").attr(
                        "d",
                        d3.arc().innerRadius(this.yScale(this.graphLabels[i])).outerRadius(this.yScale(this.graphLabels[i]) + this.yScale.bandwidth()).startAngle(0).endAngle(1.5*Math.PI)
                    ).attr("fill","#f1f1f1");

                    let valGroup = group.append('g').attr("class","entry");
                    valGroup.append("text")
                            .attr("dx",-10)
                            .attr("dy",this.yScale(this.graphLabels[this.graphLabels.length - i - 1]) - this.cY - this.yScale.bandwidth())
                            .attr("text-anchor","end")
                            .attr("font-family","'Helvetica Neue', Helvetica, Arial, sans-serif")
                            .attr("fill",colorScale(val))
                            .attr("font-size",this.yScale.bandwidth())
                            .text(this.graphLabels[i] + ": " + val);
                    
                    valGroup.append("path").attr(
                        "d",
                        d3.arc().innerRadius(this.yScale(this.graphLabels[i])).outerRadius(this.yScale(this.graphLabels[i]) + this.yScale.bandwidth()).startAngle(0).endAngle(radialScale(val))
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

<style scoped>
    .title-text{
        font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
        font-weight: 400;
        font-size: 1.2rem;
    }
</style>