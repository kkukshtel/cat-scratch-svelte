<script>
	//add in onMount
	import { onMount } from 'svelte';
	import Scratch from './Scratch.svelte';
	import { getNonce } from './util.js';
	//attach a function to onMount
	onMount(() => {
        vscode.postMessage({
            type: 'init-view',
		});
	});

	let dataType = "";
	let jsonData = {};

	function windowMessage(event) {
		const message = event.data; // The json data that the extension sent
		switch (message.type) {
			case 'init':
				//the extension is sending us an init event with the document text
				//note: this is the document NOT the state, the state takes precendece, so if any state exists use that instead
				const state = vscode.getState();
				if (state) {
					//we push this state from the vscode workspace to the JSON this component is looking at
					jsonData = JSON.parse(state.text);
				}
				else {
					// this pings the document to send us its state
					// it's then recieved in the windowMessage function where we update our content
					jsonData = JSON.parse(message.text);
				}
				dataType = message.dataType;
				return;
			case 'update':
				const text = message.text;
				// Update our webview's content
				jsonData = JSON.parse(text);
				// Then persist state information.
				// This state is returned in the call to `vscode.getState` below when a webview is reloaded.
				vscode.setState({ text });

				return;
		}
	}

	let scratchCharacters = ['😸', '😹', '😺', '😻', '😼', '😽', '😾', '🙀', '😿', '🐱'];
	function addNewScratch() {
		let character = scratchCharacters[Math.floor(Math.random() * scratchCharacters.length)];
		jsonData.scratches = [
			...(Array.isArray(jsonData.scratches) ? jsonData.scratches : []),
			{
				id: getNonce(),
				text: character,
				created: Date.now(),
			}
		];
		pushDataUpdate()
	}

	function pushDataUpdate() {
		vscode.postMessage({
			type: 'update',
			data: jsonData
		});
	}

function handleMessage(event) {
	switch (event.detail.type) {
		case "update":
			pushDataUpdate();
			break;
		default:
			break;
	}
}
</script>

<svelte:window on:message={windowMessage}/>
{#if dataType == ""}
	<p>Loading</p>
{:else if dataType === "dbark"}
	<h1>WOOF</h1>
{:else if dataType === "cscratch"}
	{#each jsonData.scratches as scratch}
		<Scratch bind:scratches={jsonData.scratches} bind:data={scratch} on:message={handleMessage}/>
	{/each}
	<button on:click={addNewScratch}>Add Scratch</button>
	<pre>{JSON.stringify({jsonData},null,2)}</pre>
{/if}