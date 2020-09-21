<script>
import { createEventDispatcher } from 'svelte';
export let scratches;
export let data;
const dispatch = createEventDispatcher();
function removeScratch() {
	scratches = scratches.filter(el => el !== data);
	dataUpdated();
}
function dataUpdated() {
	dispatch('message', {
		"type" : "update"
	});
}
</script>
<div>
	<div>face: <input bind:value={data.text} on:input={dataUpdated}></div>
	<div>id: <input bind:value={data.id} on:input={dataUpdated}></div>
	<div>date: {new Date(data.created).toUTCString()}</div>
	<br>
	<button on:click={removeScratch}>Remove</button>
</div>