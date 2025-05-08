<script lang="ts">
	import type { NoteData } from "./item.svelte";

	let {noteData, positionX, positionY, visible, internalRef = $bindable()} = $props();

</script>

<style>
	div {
		position: absolute;
		width: 500px;
		height: 500px;

		background-color: white;
		border: 20px solid black;

		padding: 5px;
	}
</style>

{#if visible == true}
<div style="left: {positionX}px; top: {positionY}px;" bind:this={internalRef}>
	<h1>Notes:</h1>
	{#each noteData as item}
	{#if item.type === 'culture'}
	<h2>Cultural Reference</h2>
	{/if}
	<p>{item.text}</p>
	{/each}
</div>
{/if}