<script>
    import {onMount} from "svelte"

    import {scaleLinear, scaleOrdinal} from "d3-scale"
    import {schemeCategory10} from "d3-scale-chromatic"
    import {select, selectAll} from "d3-selection"
    import {drag} from "d3-drag"
    import {
        forceSimulation,
        forceLink,
        forceManyBody,
        forceCenter
    } from "d3-force"

    import {event as currentEvent} from "d3-selection"  // Needed to get drag working, see: https://github.com/d3/d3/issues/2733
    let d3 = {
        scaleLinear,
        scaleOrdinal,
        schemeCategory10,
        select,
        selectAll,
        drag,
        forceSimulation,
        forceLink,
        forceManyBody,
        forceCenter
    }

    export let graph

    let xScale, yScale, xTicks, yTicks, d3yScale, links, nodes
    let svg
    let width = 500
    let height = 600

    const padding = {top: 20, right: 40, bottom: 40, left: 25}

    $: xScale = scaleLinear()
            .domain([0, 20])
            .range([padding.left, width - padding.right])

    $: yScale = scaleLinear()
            .domain([0, 12])
            .range([height - padding.bottom, padding.top])

    $: xTicks = width > 180 ?
            [0, 4, 8, 12, 16, 20] :
            [0, 10, 20]

    $: yTicks = height > 180 ?
            [0, 2, 4, 6, 8, 10, 12] :
            [0, 4, 8, 12]

    $: d3yScale = scaleLinear()
            .domain([0, height])
            .range([height, 0])

    $: links = graph.links.map(d => Object.create(d))
    $: nodes = graph.nodes.map((d, idX) => {
        Object.create(d)
        d.id = d.title ? d.title : idX
        return d
    })

    const colourScale = d3.scaleOrdinal(d3.schemeCategory10)

    onMount(network)

    function resize() {
        ({width, height} = svg.getBoundingClientRect())
        console.log("resize()", width, height)
    }

    let simulation

    function dragstarted() {
        if (!currentEvent.active) simulation.alphaTarget(0.3).restart()
        currentEvent.subject.fx = currentEvent.subject.x
        currentEvent.subject.fy = currentEvent.subject.y
    }

    function dragged() {
        currentEvent.subject.fx = currentEvent.x
        currentEvent.subject.fy = currentEvent.y
    }

    function dragended() {
        if (!currentEvent.active) simulation.alphaTarget(0)
        currentEvent.subject.fx = null
        currentEvent.subject.fy = null
    }

    function dragsubject() {
        return simulation.find(currentEvent.x, currentEvent.y)
    }

    function network() {
        resize()

        links.forEach(link => {
            link.source = nodes[link.source] || (nodes[link.source] = {
                name:
                        "unknown"
            })
            link.target = nodes[link.target] || (nodes[link.source] = {
                name:
                        "unknown"
            })


        })

        simulation = d3.forceSimulation(nodes)
                .force("link", d3.forceLink(links).id(d => d.id))
                .force("charge", d3.forceManyBody())
                .force("center", d3.forceCenter(width / 2, height / 2))
                .on("tick", function ticked() {
                    simulation.tick()
                    nodes = [...nodes]
                    links = [...links]
                })

        d3.select(svg)
                .call(d3.drag()
                        .container(svg)
                        .subject(dragsubject)
                        .on("start", dragstarted)
                        .on("drag", dragged)
                        .on("end", dragended))
    }
</script>

<svelte:window on:resize='{resize}'/>

<!-- SVG was here -->
<svg bind:this={svg}>
    {#each links as link}
        <g stroke='#999' stroke-opacity='0.6'>
            <line x1='{link.source.x}' y1='{link.source.y}'
                  x2='{link.target.x}' y2='{link.target.y}'
                  transform='translate(0 {height}) scale(1 -1)'>
                <title>{link.source.id}</title>
            </line>
        </g>
    {/each}

    {#each nodes as point}
        {#if point.label === "customer"}
            {point.label}
            <circle class='node' r='5' fill='blue' cx='{point.x}' cy='{point
            .y}'
                    transform='translate(0 {height}) scale(1 -1)'>
                <title>{point.id}</title></circle>
        {:else}
            <circle class='node' r='5' fill='red' cx='{point.x}' cy='{point
            .y}'
                    transform='translate(0 {height}) scale(1 -1)'>
                <title>{point.id}</title></circle>
        {/if}
    {/each}

</svg>

<style>
    svg {
        width: 80%;
        height: 80%;
        float: left;
    }

    circle {
        stroke: #fff;
        stroke-width: 1.5;
    }

    .tick line {
        stroke: #ddd;
        stroke-dasharray: 2;
    }

    text {
        font-size: 12px;
        fill: #999;
    }

    .x-axis text {
        text-anchor: middle;
    }

    .y-axis text {
        text-anchor: end;
    }
</style>