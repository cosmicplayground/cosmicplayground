<script lang="ts">
	import type {ClockStore} from '$lib/clock.js';

	import AppControlsTable from '$lib/AppControlsTable.svelte';

	export let clock: ClockStore; // TODO or use context?
</script>

<AppControlsTable {clock}
	><tr><td><code>[Space]</code></td><td>reset simulation</td></tr><tr
		><td><code>[ctrl+S]</code></td><td>save to local storage</td></tr
	></AppControlsTable
>
