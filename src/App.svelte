<script>
    import Graph from "./Graph.svelte"
    import GraphNode from "./GraphNode.svelte"
    import GraphEdge from "./GraphEdge.svelte"
    import Nav from "./Nav.svelte"
    import {onMount} from "svelte"

    let graphData = undefined

    let layout = "concentric"

    onMount(async () => {
        const response = await fetch("http://localhost:8080/graph")
        graphData = await response.json()
        console.log(graphData)
    })


</script>
<div class="app">

    <Nav/>
    {#if !graphData}
        <p class="loading">loading...</p>
    {:else}
        <div class="row">
            <label>
                <span>Select the layout:</span>
                <select bind:value={layout}>
                    <option value="concentric">Concentric</option>
                    <option value="random">Random</option>
                    <option value="grid">Grid</option>
                    <option value="circle">Circle</option>
                    <option value="breadthfirst">Breadthfirst</option>
                </select>
            </label>
        </div>

        <Graph layout={layout}>
            {#each graphData.nodes as node}
                <GraphNode node={node}/>
            {/each}

            {#each graphData.edges as edge}
                <GraphEdge edge={edge}/>
            {/each}
        </Graph>
    {/if}
</div>


<style>
    .app {
        display: flex;
        flex-direction: column;
        height: 100%;
        margin: auto;
    }

    .row {
        display: flex;
        flex-direction: row;
        justify-content: start;
        align-items: center;
    }

    .row span {
        margin-right: 12px;


    }

    .row label:hover {

    }

    .row select {
        border-radius: 4px;
        cursor: pointer;
        border: 1px solid #3B3F4F;
        opacity: 0.8;
    }

    .loading {
        opacity: 0;
        animation: 0.4s 0.8s forwards fade-in;
        font-size: 80px;
        color: darkblue;
        position: sticky;
        top: calc(50% - 40px);
    }

    @keyframes fade-in {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }
</style>

