<script lang="ts">
	import { XMLParser, XMLBuilder, XMLValidator, type X2jOptions } from "fast-xml-parser";
	import { onMount } from "svelte";
	import Item, { type NoteData, type PageData } from "./item.svelte";

	const parserOptions: X2jOptions = {
		preserveOrder: true
	}
	
	const xmlParser = new XMLParser(parserOptions);
	const xmlBuilder = new XMLBuilder();
	let teiTree: Array<any> = $state(new Array());

	let pages: PageData[][] = $state(new Array());
	let pageNumber = $state(0);

	function flattenTree(jsonTree: any[]): void {
		let pageData: PageData[] = [];

		const centDiv: any[] = jsonTree[1]?.TEI[1]?.text[0]?.body[0]?.div;
		if (!centDiv) return;

		for (const block of centDiv) {
			const blockType = Object.keys(block)[0];
			const blockContent = block[blockType]?.[0];
			const blockNotes = block[blockType]?.[1];
			console.log(JSON.stringify(blockNotes))

			if (blockType === "pb") {
				pages.push(pageData);
				pageData = [];
			} else {
				const text = typeof blockContent === 'object' && '#text' in blockContent
					? String(blockContent['#text'])
					: '';

					let notes: NoteData[] = new Array();
					if (blockNotes != null) {
						const notesRaw = blockNotes.notes.map((n: any) => n['#text']);
						notes = notesRaw.map((_text: string) => { return { type: "culture", text: _text}})
					}
					
					pageData.push({ type: blockType, text, notes });
			}
		}

		pages.push(pageData);
	}
	
	onMount(async () => {
		const res = await fetch("/example_book.xml");
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
		padding: 50px;
    	box-sizing: border-box;
	}
</style>

<div class="book_container">
	<div class="header_container"><h1>{teiTree[1]?.TEI[0].teiHeader[0].fileDesc[0].titleStmt[0].title[0]["#text"]} by {teiTree[1]?.TEI[0].teiHeader[0].fileDesc[0].titleStmt[1].author[0]["#text"]}</h1></div>
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
