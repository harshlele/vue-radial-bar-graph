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
        title:String,
        labelText:{
            type:String,
            default: 'label: value'
        },
        hoverText:{
            type:String,
            default: 'label: value (percent)'
        }
    },
    data: function(){
        return {
            svgWidth: parseInt(this.width * 0.9),
            svgHeight: parseInt(this.width * 0.9),
            id: "radialbargraph" + parseInt(Math.random()*100),
            cX: parseInt(this.width * 0.45),
            cY: parseInt(this.width * 0.45),
            graphValues: this.values,
            graphLabels: this.labels,
            graphColorScale: this.colorScale,
            yScale: null,
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
            .style("font-family","'Helvetica Neue', Helvetica, Arial, sans-serif")
            .style("border", "1px solid gray")
            .style("padding", "5px")
            .style("position","absolute")
            .style("color","#6c757d")
            .style("z-index","10");
    },

    methods: {
        drawChart(){

            const radialScale = d3.scaleLinear().domain([0,Math.max(...this.graphValues)*1.1]).range([0,(Math.PI * 1.5)]);
            this.yScale = d3.scaleBand().paddingInner(0.2).domain(this.graphLabels).range([this.cY*0.2,this.cY*0.95]);
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

                    let entryLabel = this.labelText;
                    entryLabel = entryLabel
                                .replace(new RegExp("label","g"),this.graphLabels[i])
                                .replace(new RegExp("percent","g"),(val/sum * 100).toFixed(2) + "%")
                                .replace(new RegExp("value","g"),val);

                    valGroup.append("text")
                            .attr("dx",-10)
                            .attr("dy",this.yScale(this.graphLabels[this.graphLabels.length - i - 1]) - (this.cY * 1.05))
                            .attr("text-anchor","end")
                            .attr("font-family","'Helvetica Neue', Helvetica, Arial, sans-serif")
                            .attr("fill",colorScale(val))
                            .attr("font-size",(this.yScale.bandwidth() * 0.8))
                            .text(entryLabel);
                    
                    valGroup.append("path").attr(
                        "d",
                        d3.arc().innerRadius(this.yScale(this.graphLabels[i])).outerRadius(this.yScale(this.graphLabels[i]) + this.yScale.bandwidth()).startAngle(0).endAngle(radialScale(val))
                    ).attr("fill", colorScale(val));  


                    let entryTooltip = this.hoverText;
                    entryTooltip = entryTooltip
                                .replace(new RegExp("label","g"),this.graphLabels[i])
                                .replace(new RegExp("percent","g"),(val/sum * 100).toFixed(2) + "%")
                                .replace(new RegExp("value","g"),val);

                    valGroup.on("mouseover",() => {
                        d3.select("#"+this.id+"tooltip").style("opacity",1).style("top",(d3.event.pageY + 10) + "px").style("left",(d3.event.pageX + 10) + "px").text(entryTooltip);
                    });
                    valGroup.on("mouseleave",() => {
                        d3.select("#"+this.id+"tooltip").style("opacity",0);
                    });
                    valGroup.on("mousemove",() => {
                        d3.select("#"+this.id+"tooltip").style("top",(d3.event.pageY + 10) + "px").style("left",(d3.event.pageX + 10) + "px").text(entryTooltip);
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
        font-family: Helvetica, Arial, sans-serif;
        font-weight: 400;
        font-size: 1.5vw;
        color: #6c757d;
    }
</style>