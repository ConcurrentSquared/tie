<script lang="ts">
	import type { NoteData } from "./item.svelte";

	let {noteData, positionX, positionY, visible, internalRef = $bindable()} = $props();

</script>

<style>
	div {
		position: absolute;
		width: 300px;
		height: 200px;

		background-color: white;
		border: 2px solid black;

		padding: 5px;

		overflow: scroll;
	}

	p, h1, h2, h3 {
		font: 1em 'Libre Baskerville';
	}
</style>

{#if visible == true}
<div style="left: {positionX}px; top: {positionY}px;" bind:this={internalRef}>
	<h1>Notes:</h1>
	{#each noteData as item}
	{#if item.type === 'culture'}
	<h2>Cultural Reference</h2>
	{:else if item.type === 'e'}
	<h2>Editorial Commentary:</h2>
	{:else}
	<h2>Other Notes:</h2>
	{/if}
	<p>{item.text}</p>
	{/each}
</div>
{/if}