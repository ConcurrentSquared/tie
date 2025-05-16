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
		if (!jsonTree) return;
		console.log(JSON.stringify(jsonTree))
		for (const block of jsonTree) {
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

		flattenTree(teiTree[4]?.TEI[1]?.text[0]?.front);
		flattenTree(teiTree[4]?.TEI[1]?.text[1]?.body);
		flattenTree(teiTree[4]?.TEI[1]?.text[2]?.back);
		console.log(JSON.stringify(pages));
	});

	function onPreviousPage() {
		pageNumber -= 2;
		Math.min(Math.max(pageNumber, 0), 12);
	}

	function onNextPage() {
		pageNumber += 2;
		Math.min(Math.max(pageNumber, 0), 12);
	}

	// from: https://stackoverflow.com/questions/9083037/convert-a-number-into-a-roman-numeral-in-javascript
	function RomanizePageNumberIfNegative(numeral: number, pageZero: number) {
		var roman: any = {
			M: 1000,
			CM: 900,
			D: 500,
			CD: 400,
			C: 100,
			XC: 90,
			L: 50,
			XL: 40,
			X: 10,
			IX: 9,
			V: 5,
			IV: 4,
			I: 1
		};
		var str = '';
		if((numeral - pageZero) > 0) {
			return String((numeral - pageZero));
		}

		var newNumeral = Math.abs(numeral - pageZero); 
		for (var i of Object.keys(roman)) {
			var q = Math.floor(newNumeral / roman[i]);
			newNumeral -= q * roman[i];
			str += i.repeat(q);
		}

		return str;
	}
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
		padding: 100px;
		padding-top: 5px;
		padding-bottom: 5px;
    	box-sizing: border-box;
	}

	.page {
		flex: 1;
		min-width: 0;
	}

	.page_turn_buttons_container {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		pointer-events: none;
		padding: 5px;
	}

	.meta_container {
		display: flex;
		z-index: 40;
		height: 100vh;
		width: 100vw;
		position: absolute;
		top: 0;
		left: 0;
		flex-direction: column;
		justify-content: center;
		pointer-events: none;
	}

	button {
		height: 70px;
		width: 70px;

		background-color: white;
		border: 5px solid black;
		pointer-events: auto
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
	<div class="footer_container"><p>{RomanizePageNumberIfNegative(pageNumber, 8)}</p><p>{RomanizePageNumberIfNegative(pageNumber + 1, 8)}</p></div>
</div>

<div class="meta_container"><div class="page_turn_buttons_container"><button onclick={onPreviousPage}>Previous Page</button><button onclick={onNextPage}>Next Page</button></div></div>
