<script>
	import Graph from './NetworkGraph.svelte';
	// import Graph from './NetworkGraph.svelte';
	import {onMount} from "svelte"

	let graphData


	onMount(async () => {
		const response = await fetch("http://localhost:8080/graph")
		console.log(response)
		graphData = await response.json()
	})

</script>

<style>
	.chart {
		width: 100%;
		max-width: 640px;
		height: calc(100% - 4em);
		min-height: 280px;
		max-height: 480px;
		margin: 0 auto;
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
		from { opacity: 0; }
		to { opacity: 1; }
	}
</style>

<div class="chart">
	{#if !graphData}
		<p class="loading">loading...</p>
	{:else}
		<Graph graph={graphData}/>
	{/if}


</div>