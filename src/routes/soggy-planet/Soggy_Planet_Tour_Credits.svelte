<script lang="ts">
	import {blur} from 'svelte/transition';

	import {get_settings} from '$lib/settings';
	import {base} from '$app/paths';

	const settings = get_settings();
	$: display_for_video = $settings.recording_mode;

	export let transition_duration: number;

	// transition vars
	const delay = 250;
	const blur_amount = 20;
</script>

<div class="tour-credits prose" class:display_for_video>
	<section in:blur|local={{duration: transition_duration, amount: blur_amount}}>
		<h2>created by</h2>
		Ryan Atkinson<br />
		<a href="https://www.ryanatkn.com">ryanatkn.com</a>
	</section>
	<section in:blur|local={{duration: transition_duration, delay: delay * 1, amount: blur_amount}}>
		<h2>interactive webpage</h2>
		<a href="https://www.cosmicplayground.org/soggy-planet">cosmicplayground.org/soggy-planet</a>
	</section>
	<section in:blur|local={{duration: transition_duration, delay: delay * 2, amount: blur_amount}}>
		<h2>open source code & assets</h2>
		<a href="https://github.com/ryanatkn/cosmicplayground">github.com/ryanatkn/cosmicplayground</a>
	</section>
	<section in:blur|local={{duration: transition_duration, delay: delay * 3, amount: blur_amount}}>
		<h2>music</h2>
		<a href="{base}/assets/audio/Alexander_Nakarada__Piña_Colada.mp3">"Piña Colada"</a> by Alexander
		Nakarada<br />
		<a href="https://www.serpentsoundstudios.com">serpentsoundstudios.com</a>
		<br /><br />
		<a href="/assets/audio/water_trickle.mp3">recording of water trickling</a> <br /> by
		<a href="https://freesound.org/people/FractalStudios/sounds/363120/"
			>FractalStudios on freesound.org</a
		>,
		<a href="https://creativecommons.org/publicdomain/zero/1.0/">CC0 1.0</a>
	</section>
	<section in:blur|local={{duration: transition_duration, delay: delay * 4, amount: blur_amount}}>
		<h2>imagery & data</h2>
		<a href="https://visibleearth.nasa.gov/collection/1484/blue-marble">
			Blue Marble Next Generation
		</a>
		by Reto Stöckli <br />
		<a href="https://visibleearth.nasa.gov">NASA Visible Earth</a> and
		<a href="https://earthobservatory.nasa.gov">NASA Earth Observatory</a>
		(2004) <br />
		<br /> Earth <a href="https://visibleearth.nasa.gov/images/73934/topography">topography</a> and
		<a href="https://visibleearth.nasa.gov/images/73963/bathymetry">bathymetry</a>
		by Jesse Allen <br />
		<a href="https://visibleearth.nasa.gov">NASA Visible Earth</a> and
		<a href="https://earthobservatory.nasa.gov">NASA Earth Observatory</a>
		<br /> using data from the
		<a href="https://www.gebco.net/">General Bathymetric Chart of the Oceans</a>
		<br /> produced by the <a href="https://www.bodc.ac.uk/">British Oceanographic Data Centre</a>
		(2005)
		<br /> <br />
		<a href="https://earthobservatory.nasa.gov/features/NightLights">Earth at Night</a>
		(<a href="https://earthobservatory.nasa.gov/features/NightLights/page3.php">flat maps</a>) by
		<a href="https://earthobservatory.nasa.gov/about/michael-carlowicz">Michael Carlowicz</a><br />
		<a href="https://earthobservatory.nasa.gov">NASA Earth Observatory</a>
		(2017)
	</section>
</div>

<style>
	.tour-credits {
		width: 100%;
		height: 100%;
		overflow: auto;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		font-size: 28px;
		font-weight: 300;
		text-align: center;
		text-shadow: 1px 2px rgba(0, 0, 0, 0.8);
	}
	.tour-credits.display_for_video a {
		color: var(--text_color);
	}
	section {
		max-width: 900px;
		margin: 24px 0;
	}
	section:first-child {
		margin-top: 0;
	}
	h2 {
		font-weight: normal;
		font-size: 20px;
		opacity: 0.5;
		margin-bottom: 14px;
	}
	.tour-credits.display_for_video h2 {
		color: var(--ocean_text_color);
		opacity: 1;
	}
</style>
