<script lang="ts">
	import { XMLParser, XMLBuilder, XMLValidator, type X2jOptions } from "fast-xml-parser";
	import { onMount } from "svelte";
	import Item, { type NoteData, type PageData } from "./item.svelte";

	const parserOptions: X2jOptions = {
		preserveOrder: true,
		ignoreAttributes: false
	}
	
	const xmlParser = new XMLParser(parserOptions);
	const xmlBuilder = new XMLBuilder();
	let teiTree: Array<any> = $state(new Array());

	let pages: PageData[][] = $state(new Array());
	let pageNumber = $state(0);

	function flattenTree(jsonTree: any[]): void {
		let pageData: PageData[] = [];

		const centDiv: any[] = jsonTree[4]?.TEI[1]?.text[0]?.body;
		if (!centDiv) return;
		console.log(JSON.stringify(centDiv))
		for (const block of centDiv) {
			const blockType = Object.keys(block)[0];
			const blockContent = block[blockType]?.[0];
			const blockNotes = block[blockType]?.slice(1);
			
			const blockNoteTypes = blockNotes.map((n: any) => (n[":@"]["@_type"]));

			if (blockType === "pb") {
				pages.push(pageData);
				pageData = [];
			} else {
				const text = typeof blockContent === 'object' && '#text' in blockContent
					? String(blockContent['#text'])
					: '';

					let notes: NoteData[] = new Array();
					if (blockNotes.length != 0) {
						const notesRaw = blockNotes.map((n: any, index: number) => {if(blockNoteTypes[index] == "collation") {return JSON.stringify(n.note[0]['list'])} else {return n.note[0]['#text']}});

						notes = notesRaw.map((_text: string, index: number) => { return { type: blockNoteTypes[index], text: _text}})
					}
					
					pageData.push({ type: blockType, text, notes });
			}
		}

		pages.push(pageData);
	}
	
	onMount(async () => {
		const res = await fetch("/joyce.xml");
		teiTree = xmlParser.parse(await res.text());

		console.log("Here:" + JSON.stringify(teiTree))

		flattenTree(teiTree);
		console.log(JSON.stringify(pages));
	});
</script>

<style>
	.book {
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		align-items: center;
		flex: 1 0 auto;
	}

	p, h1, h2, h3 {
		font: 1em 'Libre Baskerville';
	}

	h1 {
		font-weight: bold;
	}

	p {
		font-weight: normal;
	}

	.header_container {
		display: flex;
		justify-content: center;
	}

	.footer_container {
		display: flex;
		justify-content: space-around;
	}

	.book_container {
		display: flex;
		flex-direction: column;
		height: 100vh;
		padding: 5px;
    	box-sizing: border-box;
	}

	.page {
		flex: 1;
		min-width: 0;
	}
</style>

<div class="book_container">
	<div class="header_container"><h1>{teiTree[4]?.TEI[0].teiHeader[0].fileDesc[0].titleStmt[0].title[0]["#text"]} by James Joyce</h1></div>
	<div class="book">
		{#each pages.slice(pageNumber, pageNumber + 2) as page}
			<div class="page">
				{#each page as item}
					<Item pageData={item}></Item>
				{/each}
			</div>
		{/each}
	</div>
	<div class="footer_container"><p>1</p><p>2</p></div>
</div>
