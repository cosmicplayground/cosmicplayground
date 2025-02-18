<script lang="ts">
	import {onMount, tick} from 'svelte';
	import {random_float} from '@ryanatkn/belt/random.js';
	import {swallow} from '@ryanatkn/belt/dom.js';
	import {get_clock} from '$lib/clock.js';
	import {dev} from '$app/environment';

	import {enable_global_hotkeys} from '$lib/dom.js';
	import {get_dimensions} from '$lib/dimensions.js';
	import Soggy_Planet_Title_Screen from '$routes/soggy-planet/Soggy_Planet_Title_Screen.svelte';
	import MonthHud from '$lib/MonthHud.svelte';
	import SeaLevelHud from '$lib/SeaLevelHud.svelte';
	import DaylightHud from '$lib/DaylightHud.svelte';
	import Hud from '$lib/Hud.svelte';
	import EarthViewerPixi from '$lib/EarthViewerPixi.svelte';
	import {createResourcesStore} from '$lib/resources';
	import {get_settings} from '$lib/settings';
	import FloatingIconButton from '$lib/FloatingIconButton.svelte';
	import Soggy_Planet_Dev_Hud from '$routes/soggy-planet/Soggy_Planet_Dev_Hud.svelte';
	import Camera from '$lib/Camera.svelte';
	import {SHORE_COUNT} from '$routes/soggy-planet//constants';
	import FloatingTextButton from '$lib/FloatingTextButton.svelte';
	import Soggy_Planet_Tour from '$routes/soggy-planet/Soggy_Planet_Tour.svelte';
	import type Tour from '$lib/Tour.svelte';
	import Soggy_Planet_Menu from '$routes/soggy-planet/Soggy_Planet_Menu.svelte';
	import AppDialog from '$lib/AppDialog.svelte';

	const DEBUG_START_TIME = 0; // set to start the tour at any time for dev purposes
	const debug_start_time = dev ? DEBUG_START_TIME : 0;

	const clock = get_clock();

	let camera: Camera | undefined;
	$: x = camera?.x;
	$: y = camera?.y;
	$: scale = camera?.scale;
	$: width = camera?.width;
	$: height = camera?.height;

	const dimensions = get_dimensions();
	$: if (width) $width = $dimensions.width;
	$: if (height) $height = $dimensions.height;

	// TODO image metadata
	const image_width = 4096;
	const image_height = 2048;

	const initial_x = random_float(0, image_width);
	const initial_y = random_float($dimensions.height / 2, image_height - $dimensions.height / 2);
	const initial_width = $dimensions.width;
	const initial_height = $dimensions.height;

	const settings = get_settings();
	$: dev_mode = $settings.dev_mode;

	// TODO add auto pan button - share logic with Starlit Hanmmock and deep breath

	let show_hud = true;
	const toggle_hud = (value = !show_hud) => {
		show_hud = value;
	};

	// TODO refactor global hotkeys system (register them in this component, unregister on unmount)
	const keydown = (e: KeyboardEvent) => {
		if (show_title_screen) return;
		// map screen
		if (e.key === 'Escape' && !e.ctrlKey && enable_global_hotkeys(e.target)) {
			swallow(e);
			go_to_title_screen();
		} else if (e.key === '1' && enable_global_hotkeys(e.target)) {
			swallow(e);
			toggle_hud();
		}
	};

	const click_hud_toggle = (e: Event) => {
		swallow(e);
		toggle_hud();
	};

	// Earth's land
	const to_land_images = (min: number, max: number): string[] =>
		Array.from({length: 1 + max - min}, (_, i) => `/assets/earth/land_${i + 1 + min}.png`);
	let land_images = to_land_images(0, 11);
	let cycled_land_value = 0;
	$: cycled_land_index = Math.floor(cycled_land_value);

	// Earth's lights
	const DEFAULT_LIGHTS_OPACITY_MIN = 0;
	const DEFAULT_LIGHTS_OPACITY_MAX = 0.91;
	const LIGHTS_IMAGE = `/assets/earth/lights.png`;
	let lights_opacity_min = DEFAULT_LIGHTS_OPACITY_MIN;
	let lights_opacity_max = DEFAULT_LIGHTS_OPACITY_MAX;
	let lights_opacity = lights_opacity_min;
	const LIGHTS_OPACITY_CYCLE_TIMER = 3000; // larger is slower
	let lights_timer = LIGHTS_OPACITY_CYCLE_TIMER * 1.7;
	$: if (selected_daylight === null) lights_timer += $clock.dt;
	$: lights_opacity = to_lights_opacity(lights_timer);
	const to_lights_opacity = (time: number): number =>
		lights_opacity_min +
		((lights_opacity_max - lights_opacity_min) *
			(Math.sin(time / LIGHTS_OPACITY_CYCLE_TIMER) + 1)) /
			2;
	// TODO would be cool to have nightfall overlay the shape of the real thing, not global
	$: nightfall_opacity = (active_daylight - lights_opacity_min) * 0.48;

	// TODO this is weird because the shore images were bolted on after the fact.
	// Refactoring the sea images to have a single base and later the second/third above would be ideal

	// Earth's sea
	const sea_images = Array.from({length: 3}, (_, i) => `/assets/earth/sea_${i + 1}.png`);
	const shore_image = '/assets/earth/shore.png';
	const seashore_floor_index = SHORE_COUNT;
	const sea_index_max = sea_images.length + SHORE_COUNT - 1;
	const SEA_LEVEL_CYCLE_TIMER = 1000; // larger is slower
	let sea_level_timer = SEA_LEVEL_CYCLE_TIMER * 2.5;
	$: if (selected_sea_level === null && hovered_sea_level === null) {
		sea_level_timer += $clock.dt;
	}
	const to_sea_level = (time: number): number =>
		(sea_index_max * (Math.sin(time / SEA_LEVEL_CYCLE_TIMER) + 1)) / 2;
	let sea_level = to_sea_level(sea_level_timer);
	$: sea_level = to_sea_level(sea_level_timer);

	// update every clock tick
	const LAND_DELAY = 230;
	let land_timer = 0;
	$: if (selected_land_index === null && hovered_land_index === null) {
		land_timer += $clock.dt;
		cycled_land_value = (land_timer / LAND_DELAY) % land_images.length;
	}

	let selected_sea_level: number | null = null;
	let hovered_sea_level: number | null = null;
	$: active_sea_level = hovered_sea_level ?? selected_sea_level ?? sea_level;
	const select_sea_level = (value: number | null) => {
		selected_sea_level = value;
	};
	const hover_sea_level = (value: number | null) => {
		hovered_sea_level = value;
	};

	let selected_land_index: number | null = null;
	let hovered_land_index: number | null = null;
	$: active_land_index = hovered_land_index ?? selected_land_index ?? cycled_land_index;
	$: active_land_value =
		active_land_index === cycled_land_index ? cycled_land_value : active_land_index;
	const set_cycled_land_value = (value: number) => {
		land_timer = LAND_DELAY * value;
	};
	const select_land_index = (index: number | null) => {
		selected_land_index = index;
		if (index !== null) set_cycled_land_value(index);
	};
	const hover_land_index = (index: number | null) => {
		hovered_land_index = index;
		if (index !== null) set_cycled_land_value(index);
	};

	let selected_daylight: number | null = null;
	let hovered_daylight: number | null = null;
	$: active_daylight = hovered_daylight ?? selected_daylight ?? lights_opacity;
	const select_daylight = (value: number | null) => {
		selected_daylight = value;
	};
	const hover_daylight = (value: number | null) => {
		hovered_daylight = value;
	};

	// TODO use Pixi loader instead of the `ResourcesStore` - see the store module for more info
	const resources = createResourcesStore();
	land_images.forEach((url) => resources.addResource('image', url));
	sea_images.forEach((url) => resources.addResource('image', url));
	resources.addResource('image', shore_image);
	resources.addResource('image', LIGHTS_IMAGE);

	// in dev mode, bypass the title screen for convenience
	let show_title_screen = true;
	const proceed = () => {
		show_title_screen = !show_title_screen;
	};
	const go_to_title_screen = () => {
		show_title_screen = true;
	};
	const go_to_map = () => {
		show_title_screen = false;
	};
	onMount(() => {
		// in dev mode, bypass the title screen for convenience
		if (dev_mode) {
			go_to_map();
			void resources.load();
		}
	});

	let tour: Tour | undefined;
	$: touring = tour ? tour.touring : null;
	$: console.log(`$touring`, $touring);

	const start_tour = async (): Promise<void> => {
		if (show_title_screen) {
			go_to_map();
			await resources.load();
		}
		if (!tour) await tick();
		if (!tour) return; // TODO hmm?

		tour.begin_tour();
	};

	const on_begin_tour = () => {
		console.log(`on_begin_tour args`);
	};

	// TODO hacky
	let was_touring = false;
	$: {
		if (was_touring && !$touring) {
			lights_opacity_min = DEFAULT_LIGHTS_OPACITY_MIN;
			lights_opacity_max = DEFAULT_LIGHTS_OPACITY_MAX;
		}
		was_touring = !!$touring;
	}

	const update_land_images: (min: number, max: number) => void = (min, max) => {
		console.log(`update_land_images`, min, max);
		land_images = to_land_images(min, max);
		select_land_index(min === max ? max : null);
		hover_land_index(null);
	};
	const update_daylight: (min: number, max: number) => void = (min, max) => {
		lights_opacity_min = min;
		lights_opacity_max = max;
		select_daylight(max);
		hover_daylight(max);
	};
	const update_sea_level: (min: number, max: number) => void = (_min, max) => {
		// TODO would be nice to animate these better, but for now I'm just hacking some basic smoothing into the tour script with helpers
		select_sea_level(max);
		hover_sea_level(max);
	};
</script>

<svelte:window on:keydown={keydown} />

<Camera
	bind:this={camera}
	initialX={initial_x}
	initialY={initial_y}
	initialWidth={initial_width}
	initialHeight={initial_height}
/>

{#if camera && x && y && scale}
	<div class="soggy-planet">
		{#if !show_title_screen && $resources.status === 'success'}
			<EarthViewerPixi
				{camera}
				landImages={land_images}
				seaImages={sea_images}
				shoreImage={shore_image}
				shoreImageCount={SHORE_COUNT}
				seashoreFloorIndex={seashore_floor_index}
				lightsImage={LIGHTS_IMAGE}
				lightsOpacity={active_daylight}
				nightfallOpacity={nightfall_opacity}
				showLights={true}
				activeLandValue={active_land_value}
				activeSeaLevel={active_sea_level}
				imageWidth={image_width}
				imageHeight={image_height}
			/>
			<Soggy_Planet_Tour
				{camera}
				bind:tour
				on:begin={on_begin_tour}
				{update_land_images}
				{update_daylight}
				{update_sea_level}
			/>
			<Hud>
				<!-- TODO these conditions are awkward copypasta from deep-breath -->
				{#if tour && $touring}
					<FloatingIconButton label="cancel tour" on:click={tour.cancel}>✕</FloatingIconButton>
				{:else if show_hud}
					<FloatingIconButton label="go back to title screen" on:click={go_to_title_screen}>
						⇦
					</FloatingIconButton>
				{:else}
					<div class="hud-top-controls">
						<FloatingIconButton
							pressed={show_hud}
							label="toggle hud controls"
							on:click={click_hud_toggle}
						>
							∙∙∙
						</FloatingIconButton>
					</div>
				{/if}
				{#if show_hud && (!$touring || dev_mode)}
					<div class="hud-top-controls">
						<FloatingIconButton
							pressed={show_hud}
							label="toggle hud controls"
							on:click={click_hud_toggle}
						>
							∙∙∙
						</FloatingIconButton>
						<FloatingTextButton
							label="start the tour of our soggy planet with history and myth"
							on:click={start_tour}
						>
							tour
						</FloatingTextButton>
					</div>
					<div class="hud-left-controls">
						<DaylightHud
							daylight={active_daylight}
							{selected_daylight}
							{select_daylight}
							{hover_daylight}
						/>
						{#if dev_mode}
							<Soggy_Planet_Dev_Hud tour={tour || null} {x} {y} {scale} {debug_start_time} />
						{/if}
					</div>
					<div class="month-wrapper">
						<MonthHud
							{active_land_index}
							{selected_land_index}
							{select_land_index}
							{hover_land_index}
						/>
					</div>
					<SeaLevelHud
						sea_level={active_sea_level}
						{sea_index_max}
						{selected_sea_level}
						{select_sea_level}
						{hover_sea_level}
					/>
				{/if}
			</Hud>
		{:else}
			<Soggy_Planet_Title_Screen {resources} {proceed} {start_tour} />
		{/if}
		<!-- {#if dev_mode}
		<div
			style="position: fixed; left: calc(50% - 3px); top: calc(50% - 3px); width: 7px; height: 7px;
			background-color: rgba(255, 50, 50, 1);"
		/>
	{/if} -->
	</div>
{/if}

<AppDialog>
	<Soggy_Planet_Menu {clock} />
</AppDialog>

<style>
	.soggy-planet {
		position: relative;
	}

	.hud-top-controls {
		position: absolute;
		left: var(--hud_element_size);
		top: 0;
		display: flex;
	}
	.hud-left-controls {
		position: fixed;
		left: 0;
		top: var(--hud_element_size);
		height: calc(100% - 2 * var(--hud_element_size));
		font-size: 72px;
	}
	.month-wrapper {
		/* TODO make this not fixed */
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
	}
</style>
