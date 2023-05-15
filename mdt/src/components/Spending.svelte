<script>
    import * as d3 from 'd3';    
    // import { seedrandom } from 'seedrandom';
    // import {randomUniform} from 'd3-random';
    import {randomLcg, randomNormal} from "d3-random";

    import { voronoiTreemap } from 'd3-voronoi-treemap';
    // import { gdp } from "../data/globalEconomyByGDP.json.js";
    import { remitExp } from "../data/avgSpendingForRemit.json.js";
    import { polygonScaleArea } from 'geometric';    

    //begin: constants
    const _2PI = 2 * Math.PI;
    // const geometric = require("geometric"); // for rescale polygon
    //end: constants

    export let state = 0;
    
    //begin: layout conf.
    
    var svgWidth = 840; //850
    var svgHeight = 550; //500
    var margin = {top: 5, right: 5, bottom: 10, left: 5};
    var height = svgHeight - margin.top - margin.bottom;
    var width = svgWidth - margin.left - margin.right;
    var halfWidth = width / 2;
    var halfHeight = height / 2;
    var legendsMinY = height - 20;
    var treemapRadius = 235; // 205
    var treemapCenter = [halfWidth, halfHeight - 20]; // halfHeight + 5
    var tooptipBoxWidth = 180;
    var tooptipBoxHeight = 90;
    //end: layout conf.

    //begin: treemap conf.
    var _voronoiTreemap = voronoiTreemap();
    var hierarchy, circlingPolygon;
    const seed = 0.58971573888282423; // any number in [0, 1)
    // 0.44871573888282423, 0.58871573888282423
    const random_seeds = randomNormal.source(randomLcg(seed))(0, 1);
    _voronoiTreemap.prng(random_seeds);
    //end: treemap conf.
    
    //begin: drawing conf.
    var fontScale = d3.scaleLinear()
        .domain([3, 20])
        .range([10, 20]) // 8 20
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
    function showNoRemit(value) {
        // let something;
        if (value === 1 && showNoRemitStatus === 0) {
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

            d3.selectAll("#button-remit")
			    .classed("active", false);

            d3.select("#button-no-remit")
                .classed("active", true);
    
            showNoRemitStatus = 1;
        } else if (value === 0 && showNoRemitStatus === 1) {
            d3.selectAll('.cell-remit')
                .data(leaves)
                .transition()
                .duration(500)
				.ease(d3.easeLinear)
                .attr("d", function(d) { 
                    return "M"+ d.polygon.join(",")+"z"; 
                });

            d3.selectAll("#button-remit")
			    .classed("active", true);

            d3.select("#button-no-remit")
                .classed("active", false);

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

    function transition() {
        console.log("Current state", state);
        switch(state) {
            case 0: 
                showNoRemit(0);
                break;
            case 3: 
                showNoRemit(1);
                break;
            case 4: 
                showNoRemit(1);
                break;
            default:
                showNoRemit(0);
                break;

        }
    }

    $: state, transition();

</script>


<main>
    <label class="selections">
        Choose household type:
        <button id = "button-remit" class = "button active" on:click={() => showNoRemit(0)}> With Remittance Support </button>
        <button id = "button-no-remit" class = "button" on:click={() => showNoRemit(1)}> Without Remittance Support </button>
        <!-- <button
            class = "button active"
            aria-label = "Show no remit"
            on:click={() => showNoRemit()}
        >
        {#if showNoRemitStatus === 0}
            Show Without Remittances
        {:else}
            Show With Remittances
        {/if}
        </button> -->
    </label>

    <h2 class="title">Average monthly expenditure of households who <b>{showNoRemitStatus == 0 ? "receive remittance support" : "do not receive remittance support"}</b></h2>
    
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

                        <g class = "hoverer-cell">
                            <path 
                                class = "hoverer"
                                d = "M{country.polygon.join(",")}z"
                            >
                                <!-- <title >
                                    {country.data.name + "\nWith remittance: $" + 
                                    country.data.remitAmount.toFixed(2) +
                                    "\nWithout remittance: $" + 
                                    country.data.noRemitAmount.toFixed(2) +
                                    "\nDifference: " +
                                    (((country.data.remitAmount/country.data.noRemitAmount) - 1) * 100).toFixed(1) +
                                    "%"}
                                </title> -->
                            </path>
                            <rect class = "hoverer-textbox hoverer-textbox-rect" 
                                x = "{country.polygon.site.x}" 
                                y = "{country.polygon.site.y}" 
                                width="{tooptipBoxWidth}" 
                                height="{tooptipBoxHeight - 8}"/>
                            <text 
                                class = "hoverer-textbox hoverer-textbox-text hoverer-title" 
                                x = "{country.polygon.site.x + tooptipBoxWidth / 18}" 
                                y = "{country.polygon.site.y + tooptipBoxHeight / 5}" 
                            >
                                {country.data.name}
                            </text>
                            <text 
                                class = "hoverer-textbox hoverer-textbox-text" 
                                x = "{country.polygon.site.x + tooptipBoxWidth / 18 }" 
                                y = "{country.polygon.site.y + tooptipBoxHeight / 5 * 2}"  
                            >
                                With remittance: ${country.data.remitAmount.toFixed(2)}
                            </text>
                            <text 
                                class = "hoverer-textbox hoverer-textbox-text" 
                                x = "{country.polygon.site.x + tooptipBoxWidth / 18 }" 
                                y = "{country.polygon.site.y + tooptipBoxHeight / 5 * 3}"  
                            >
                                Without remittance: ${country.data.noRemitAmount.toFixed(2)}
                            </text>
                            <text 
                                class = "hoverer-textbox hoverer-textbox-text" 
                                x = "{country.polygon.site.x + tooptipBoxWidth / 18 }" 
                                y = "{country.polygon.site.y + tooptipBoxHeight / 5 * 4}"  
                            >
                            Difference: +{(((country.data.remitAmount/country.data.noRemitAmount) - 1) * 100).toFixed(1)}%
                            </text>
                        </g>
                    
                    {/each}
                </g>
            </g>
        </g>
<!-- 

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
        </text> -->

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
                font-weight = 300
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
        stroke-width: 0px;
    }

    .hoverer:hover{
        stroke-width: 4px;
    }

    .hoverer-textbox {
        visibility: hidden;
    }

    .hoverer-cell:hover .hoverer-textbox {
        visibility: visible;
    }

    .hoverer-textbox-rect {
        fill: #FFFFFF;
        fill-opacity: 0.85;
    }

    .hoverer-textbox-text {
        font-family: 'Nunito', sans-serif;
        font-size: 80%;
    }

    .hoverer-title {
        font-weight: 600;
    }

    .legend-color {
        stroke-width: 1px;
        stroke:darkgrey;
    }

    b {
        color: #B990EC;
    }
</style>
