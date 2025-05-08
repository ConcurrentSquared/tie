<script lang="ts">
	import { tick } from "svelte";

	import Modal from "./modal.svelte";

	let {pageData} = $props();
	let modalVisible = $state(false);
	let modalX =  $state(0);
	let modalY =  $state(0);

	function openNotes(event: MouseEvent) {
		const { clientX, clientY } = event;
		console.log("here")
		modalVisible = true;
		modalX = clientX;
		modalY = clientY;

		event.stopPropagation();
	}
	
	let container: any = $state();
	async function closeNotes(event: MouseEvent) {
		await tick(); 
		if ((container != null) && (modalVisible == true)) {
			if (!container.contains(event.target)) {
				console.log('clicked outside');
				modalVisible = false;
			}
		}
	}
</script>
<script module>
	export type PageData = {
		type: string,
		text: string,
		notes: NoteData[]
	}

	export type NoteData = {
		type: string,
		text: string
	}
</script>

<style>
	p, h1, h2, h3 {
		font: 1em 'Libre Baskerville';
	}
</style>

{#if pageData.type === 'p'}
	<p onclick={openNotes}>{pageData.text}</p>
{:else if pageData.type === 'head'}
	<h3>{pageData.text}</h3>
{/if}

<svelte:window onclick={closeNotes}></svelte:window>

<Modal bind:internalRef={container} noteData={pageData.notes} positionX={modalX} positionY={modalY} visible={modalVisible}></Modal>