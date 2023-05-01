<script>
    import * as d3 from 'd3';    
    import { voronoiTreemap } from 'd3-voronoi-treemap';
    // import { gdp } from "../data/globalEconomyByGDP.json.js";
    import { remitExp } from "../data/avgSpendingForRemit.json.js";
    import { polygonScaleArea } from 'geometric';    

    //begin: constants
    const _2PI = 2 * Math.PI;
    // const geometric = require("geometric"); // for rescale polygon
    //end: constants
    
    //begin: layout conf.
    var svgWidth = 850;
    var svgHeight = 500;
    var margin = {top: 10, right: 10, bottom: 10, left: 10};
    var height = svgHeight - margin.top - margin.bottom;
    var width = svgWidth - margin.left - margin.right;
    var halfWidth = width / 2;
    var halfHeight = height / 2;
    var quarterWidth = width / 4;
    var quarterHeight = height / 4;
    var titleY = 20;
    var legendsMinY = height - 20;
    var treemapRadius = 205;
    var treemapCenter = [halfWidth, halfHeight + 5];
    //end: layout conf.

    //begin: treemap conf.
    var _voronoiTreemap = voronoiTreemap();
    var hierarchy, circlingPolygon;
    //end: treemap conf.
    
    //begin: drawing conf.
    var fontScale = d3.scaleLinear()
        .domain([3, 20])
        .range([8, 20])
        .clamp(true);
    //end: drawing conf.

    function computeCirclingPolygon(radius) {
        var points = 60,
            increment = _2PI/points,
            circlingPolygon = [];
        
        for (var a = 0, i = 0; i < points; i++, a += increment) {
            circlingPolygon.push(
                [radius + radius * Math.cos(a), radius + radius * Math.sin(a)]
            )
        }
        
      	return circlingPolygon;
    };

    circlingPolygon = computeCirclingPolygon(treemapRadius);
    let legendHeight = 13;
    let interLegend = 4;
    let colorWidth = legendHeight * 6;
    let continents = remitExp.children.reverse();

    hierarchy = d3.hierarchy(remitExp).sum(function(d){ return d.weight; });
    _voronoiTreemap.clip(circlingPolygon)(hierarchy);
    var leaves 
    leaves = hierarchy.leaves();
   
    let showNoRemitStatus = 0;
    function showNoRemit() {
        // let something;
        if (showNoRemitStatus === 0) {
            d3.selectAll('.cell-remit')
                .data(leaves)
                .transition()
                .duration(500)
				.ease(d3.easeLinear)
                .attr("d", function(d) { 
                    return "M"+ polygonScaleArea(d.polygon, d.data.noRemitRatio).join(",")+"z"; 
                });

            d3.selectAll(".value")
                .data(leaves)
                .transition()
                .duration(500)
                .textTween(
                    (d) => (t) => {
                        const index = d3.interpolateNumber(d.data.remitAmount, d.data.noRemitAmount)(t).toFixed(2);           
                        return `$${index}`;
                    }
                )
                .end();
                
            d3.select(".button")
                .style("background-color", "#264561")
            showNoRemitStatus = 1;
        } else {
            d3.selectAll('.cell-remit')
                .data(leaves)
                .transition()
                .duration(500)
				.ease(d3.easeLinear)
                .attr("d", function(d) { 
                    return "M"+ d.polygon.join(",")+"z"; 
                });

            d3.select(".button")
                .style("background-color", "#7A8B9A");

            d3.selectAll(".value")
                .data(leaves)
                .transition()
                .duration(500)
                .textTween(
                    (d) => (t) => {
                        const index = d3.interpolateNumber(d.data.noRemitAmount, d.data.remitAmount)(t).toFixed(2);           
                        return `$${index}`;
                    }
                )
                .end();


    
            showNoRemitStatus = 0;
        }
        

    };

</script>


<main>
    <label>
        <button
            class = "button"
            aria-label = "Show no remit"
            on:click={() => showNoRemit()}
        >
            Show no remit
        </button>
    </label>
    
    <svg 
        width={svgWidth} 
        height={svgHeight}
    >
        <g 
            class = "drawingArea" 
            transform = "translate({margin.left}, {margin.top})"
        >
            <g 
                class = "treemap-container"
                transform = "translate({treemapCenter})"
            >
                <path 
                    class = "world"
                    transform = "translate({-treemapRadius}, {-treemapRadius})"
                    d = "M{circlingPolygon.join(",")}Z"
                />
                <g
                    class = "cells"
                    transform = "translate({-treemapRadius}, {-treemapRadius})"
                >
                    {#each leaves as country}
                        <path 
                            class = "cell cell-remit-background"
                            d = "M{country.polygon.join(",")}Z"
                            fill = {country.parent.data.color}
                            fill-opacity = 0.2
                            visibility="visible"
                        />
                        <path 
                            class = "cell cell-remit"
                            d = "M{country.polygon.join(",")}Z"
                            fill = {country.parent.data.color}
                        />
                    {/each}
                </g>

                <g
                    class = "labels"
                    transform = "translate({-treemapRadius}, {-treemapRadius})"
                >
                    {#each leaves as country}
                        <g
                            class = "label"
                            font-size = {fontScale(country.data.weight)}
                        >
                            <text 
                                class = "name" 
                                x = "{country.polygon.site.x}" 
                                y = "{country.polygon.site.y}"
                            >
                                {country.data.name}
                            </text>
                            <text class = "value"
                                x = "{country.polygon.site.x}" 
                                y = "{country.polygon.site.y}"
                            >
                                ${country.data.remitAmount.toFixed(2)}
                            </text>
                        </g>
                    {/each}
                </g>

                <g
                    class = "hoverers"
                    transform = "translate({-treemapRadius}, {-treemapRadius})"
                >
                    {#each leaves as country}
                        <path 
                            class = "hoverer"
                            d = "M{country.polygon.join(",")}z"
                        >
                            <title>
                                {country.data.name + "\n" + country.value + "%"}
                            </title>
                        </path>
                        
                    {/each}
                </g>
            </g>
        </g>


        <text 
            id = "title" 
            transform = "translate({halfWidth}, {titleY})"
            text-anchor = "middle"
        >
            Household Expenditures by Categories
        </text>

        <text 
            class = "tiny light" 
            transform = "translate(0, {height})"
            text-anchor = "start"
        >
            Footnote 1
        </text>

        <text 
            class = "tiny light" 
            transform = "translate({halfWidth}, {height})"
            text-anchor = "middle"
        >
            Footnote 2
        </text>

        <text 
            class = "tiny light" 
            transform = "translate({width}, {height})"
            text-anchor = "end"
        >
            Footnote 3
        </text>

        <g
            class = "legend"
            transform = "translate(0, {legendsMinY})"
        >
            {#each continents as continent, i}
                
                    <g
                        class = "legend"
                        transform = "translate(0, {-i * (legendHeight + interLegend)})"
                    >
                        <rect
                            class = "legend-color"
                            y = {-legendHeight}
                            width = {colorWidth}
                            height = {legendHeight}
                            fill = {continent.color}
                        />
                        <text
                            class = "tiny"
                            x = "{colorWidth + 5}"
                            y = -2
                        >
                            {continent.name}
                        </text>
                    </g>
            {/each}
            <text
                transform = "translate(0, {-continents.length * (legendHeight + interLegend) - 5})"
            >
                Categories
            </text>
        </g>
    
    </svg>
</main>




<style>
    main{
        margin:0;
        width:100%;
        display:flex;
        flex-direction:column;
        align-items:center;
    }
    svg {
        /* background-color: rgb(250,250,250); */
    }
    
    #title {
        letter-spacing: 4px;
        font-weight: 700;
        font-size: x-large;
    }

    text.tiny {
        font-size: 10pt;
    }
    text.light {
        fill: lightgrey
    }
    
    .world {
        stroke: lightgrey;
        stroke-width: 4px;
        fill: white;
    }

    .cell {
        stroke: white;
        stroke-width: 1px;
      }

    .label {
        text-anchor: middle;
        /* fill: white; */
        color: black;
        
        /* position: relative; */
    }
    
    .label>.name {
        dominant-baseline: text-after-edge;
       
    }
    
    .label>.value {
        dominant-baseline: text-before-edge;
        top:1;
    }

    .hoverer {
        fill: transparent;
        stroke: white;
        stroke-width:0px;
    }
    
    .hoverer:hover {
        stroke-width: 3px;
    }

    .legend-color {
        stroke-width: 1px;
        stroke:darkgrey;
    }

    .button {
        background-color: #7A8B9A;
        border: none;
        color: white;
        padding: 15px 32px;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        font-size: 16px;
    }
    

</style>