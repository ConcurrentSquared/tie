<script lang="ts">
	import type { NoteData } from "./item.svelte";

	let {noteData, positionX, positionY, visible, internalRef = $bindable()} = $props();
	let collationList: any = $state(new Array());
	let displayText: boolean = $state(true)

	noteData.forEach((note: any) => {
		if ((note.type == 'collation')) {
			collationList = JSON.parse(note.text);
			note.text = ''
		}
	});
</script>

<style>
	div {
		position: absolute;
		width: 400px;
		height: 300px;

		background-color: white;
		border: 2px solid black;

		padding: 5px;

		overflow-x: scroll;
	}

	p, h1, h2, ul, li {
		font: 1em 'Libre Baskerville';
	}

	ul {
		list-style: none;
		padding: 0;
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
	{:else if item.type === 'collation'}
	<h2>Collation:</h2>
	<ul>
	{#each collationList as collation}
		<li>{collation.item[0]["#text"]}</li>
	{/each}
	</ul>
	{:else}
	<h2>Other Notes:</h2>
	{/if}
	{#if displayText == true}
		<p>{item.text}</p>
	{/if}
	{/each}
</div>
{/if}