<script>
    import * as d3 from 'd3';    
    import { voronoiTreemap } from 'd3-voronoi-treemap';
    import { gdp } from "../data/globalEconomyByGDP.json.js";
    import { remitExp } from "../data/avgSpendingForRemit.json.js";

    //begin: constants
    const _2PI = 2 * Math.PI;
    //end: constants
    
    //begin: layout conf.
    var svgWidth = 960;
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


    // async function populate() {
    //     await d3.json("../data/globalEconomyByGDP.json")
    //         .then(function(rootData) {
    //         initData();
    //         initLayout(rootData);
    //         hierarchy = d3.hierarchy(rootData).sum(function(d){ return d.weight; });
    //         _voronoiTreemap
    //             .clip(circlingPolygon)
    //             (hierarchy);
            
    //         drawTreemap(hierarchy);
    //     }
    // }
    
    // fetch("../data/globalEconomyByGDP.json")
    
    // $: { d3.json("../data/globalEconomyByGDP.json")
    //     .then(function(rootData) {
    //         initData();
    //         initLayout(rootData);
    //         hierarchy = d3.hierarchy(rootData).sum(function(d){ return d.weight; });
    //         _voronoiTreemap
    //             .clip(circlingPolygon)
    //             (hierarchy);
            
    //         drawTreemap(hierarchy);
    //     });
    // }

    // let data = [];
    // onMount(async () => {
	// 	data = await d3.json("../data/globalEconomyByGDP.json");
    // populate();

	// gdp.filter((rootData) => function(rootData) {
    //         initData();
    //         initLayout(rootData);
    //         hierarchy = d3.hierarchy(rootData).sum(function(d){ return d.weight; });
    //         _voronoiTreemap
    //             .clip(circlingPolygon)
    //             (hierarchy);
            
    //         drawTreemap(hierarchy);
    // });

    // initData();

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
    var leaves = hierarchy.leaves();
   

</script>


<main>
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
                            class = "cell"
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
                            tranform = "translate({country.polygon.site.x}, {country.polygon.site.y})"
                            font-size = {fontScale(country.data.weight)}
                        >
                        <!-- TODO: (d.data.weight<1)? d.data.code : d.data.name; -->
                            <text class = "name">
                                {(country.data.weight<1)? country.data.code : country.data.name}
                            </text>
                            <text class = "value">
                                {country.data.weight + "%"}
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
            Expenditures of Households Receiving Remittances
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
                            transform = "translate({colorWidth + 5}, -2})"
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
    svg {
        background-color: rgb(250,250,250);
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
    }

    .cell {
        stroke: white;
        stroke-width: 1px;
    }

    .legend-color {
        stroke-width: 1px;
        stroke:darkgrey;
    }

    .label {
        text-anchor: middle;
        /* fill: white; */
        color: black;
    }
    
    .label>.name {
        dominant-baseline: text-after-edge;
    }
    
    .label>.value {
        dominant-baseline: text-before-edge;
    }

    .hoverer {
        fill: transparent;
        stroke: white;
        stroke-width:0px;
    }
    
    .hoverer:hover {
        stroke-width: 3px;
    }
    

</style>