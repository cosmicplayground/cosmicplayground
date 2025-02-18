<script lang="ts">
	import Pending_Animation from '@ryanatkn/fuz/Pending_Animation.svelte';
	import {base} from '$app/paths';

	import Soggy_Planet_Thumbnail from '$routes/soggy-planet/Soggy_Planet_Thumbnail.svelte';
	import Hud from '$lib/Hud.svelte';
	import HomeButton from '$lib/HomeButton.svelte';
	import ResourcesLoadingProgress from '$lib/ResourcesLoadingProgress.svelte';
	import Panel from '$lib/Panel.svelte';
	import ChunkyButton from '$lib/ChunkyButton.svelte';
	import Soggy_Planet_Credits from '$routes/soggy-planet/Soggy_Planet_Credits.svelte';
	import CreditsProjectSignature from '$routes/about/CreditsProjectSignature.svelte';
	import AboutPortalPreview from '$routes/about/Preview.svelte';
	import PortalPreview from '$lib/PortalPreview.svelte';
	import type {ResourcesStore} from '$lib/resources';
	import PortalLink from '$lib/PortalLink.svelte';
	import {points_of_interest} from '$routes/soggy-planet/soggy_planet_tour_data';

	export let resources: ResourcesStore;
	export let proceed: () => void;
	export let start_tour: () => void;

	// The user can click the title image to load the interactive,
	// but only if the user has clicked the load button directly before.
	// The information around the button is important for the user to understand
	// because the download size is so large.
	const HAS_LOADED_KEY = 'cpg__hasLoadedSoggyPlanet';
	const has_loaded = !!localStorage.getItem(HAS_LOADED_KEY);
	const enable_loading_by_clicking_thumbnail = has_loaded;

	let loading = false;
	const load = async () => {
		loading = true;
		await resources.load();
		loading = false;
		if ($resources.status === 'success') {
			if (!has_loaded) localStorage.setItem(HAS_LOADED_KEY, 'true');
			proceed();
		}
	};
</script>

<Hud>
	<HomeButton />
</Hud>
<div class="soggy_planet_title_screen box w_100">
	<div class="box width_md">
		<div style:margin-bottom="55px">
			<Soggy_Planet_Thumbnail on_click={enable_loading_by_clicking_thumbnail ? load : null} />
		</div>
		<div class="loading_animation">
			{#if loading}
				<Pending_Animation />
			{/if}
		</div>
		<Panel>
			<section class="prose">
				<p>
					Soggy Planet is an interactive map of Earth where sea levels rise and fall and the lights
					of civilization shine through the night. It displays <a
						href="https://wikipedia.org/wiki/Past_sea_level">past sea levels</a
					>
					roughly between:
				</p>
				<ul>
					<li>
						the lowest point during
						<a href="https://wikipedia.org/wiki/Last_Glacial_Maximum">the Last Glacial Maximum</a>
						before
						<a href="https://wikipedia.org/wiki/Early_Holocene_sea_level_rise"
							>the early Holocene sea level rise</a
						>
					</li>
					<li>
						and one
						<a href="https://wikipedia.org/wiki/Sea_level_rise">estimated possible maximum</a>
					</li>
				</ul>
				<p>For a 3 minute musical tour of some of our soggy planet's sunken history and myths:</p>
				<ul>
					<li>
						<p style:margin-bottom={0}>
							click the <button class="inline" on:click={start_tour}>tour</button>
							button on <button class="inline" on:click={load}>the map</button>
						</p>
					</li>
					<li>
						see
						<a href="https://www.youtube.com/watch?v=akP5J3jfjt4">the video on YouTube</a>
					</li>
					<li>or learn more below</li>
				</ul>
				<p>
					This map is resource-intensive and may be broken or slow depending on your hardware and
					browser. More optimizations like <a
						href="https://github.com/ryanatkn/cosmicplayground/issues/56">this one</a
					> would help.
				</p>
				<p>
					The code and image data are
					<a href="https://github.com/ryanatkn/cosmicplayground">open source on GitHub</a>. Credits
					are below.
				</p>
				<p>
					For a different experience about sea level rise see <PortalLink slug="deep-breath" /> 🌚
				</p>
			</section>
			<hr />
			{#if $resources.status === 'success'}
				<ChunkyButton on:click={proceed}>back to the map!</ChunkyButton>
			{:else if $resources.status !== 'initial'}
				<ResourcesLoadingProgress {resources} />
			{:else}
				<p>The download is about 81MB of images. If that's ok with you, continue on:</p>
				<ChunkyButton on:click={load}>continue with 81MB download!</ChunkyButton>
			{/if}
		</Panel>
		<Panel>
			<section class="prose">
				<h2>Tour</h2>
				<p>
					Soggy Planet includes a 3 minute tour of some points of interest related to sea levels at
					the <a href="https://wikipedia.org/wiki/Last_Glacial_Maximum">Last Glacial Maximum</a>.
					Click the <button class="inline" on:click={start_tour}>start tour</button>
					button on <button class="inline" on:click={load}>the map</button> or see
					<a href="https://www.youtube.com/watch?v=akP5J3jfjt4">the video on YouTube</a>.
				</p>
				<p>Learn about the tour's points of interest on Wikipedia:</p>
				<ul>
					{#each points_of_interest as point_of_interest (point_of_interest)}
						<li>
							<a href={point_of_interest.url}>{point_of_interest.name}</a>
							{#if point_of_interest.myth}(myth?){/if}
						</li>
					{/each}
				</ul>
			</section>
		</Panel>
		<Panel>
			<section class="prose">
				<h2>Credits</h2>
				<Soggy_Planet_Credits />
			</section>
		</Panel>
		<Panel>
			<section class="prose">
				<CreditsProjectSignature />
			</section>
		</Panel>
		<!-- TODO pull this href from portal data? -->
		<section>
			<PortalPreview href="{base}/about">
				<AboutPortalPreview />
			</PortalPreview>
		</section>
	</div>
</div>

<style>
	.soggy_planet_title_screen {
		padding: var(--space_xl5) 0;
	}

	/* TODO hacky */
	.soggy_planet_title_screen :global(.portal_preview) {
		margin: 0;
	}

	.loading_animation {
		height: 0;
		position: relative;
		top: 48px;
		color: var(--ocean_color);
	}
</style>
