<script>
	import { Progress } from '@skeletonlabs/skeleton-svelte';

	const allergens = [
		{ name: 'Peanuts', pct: 15 },
		{ name: 'Shellfish', pct: 60 },
		{ name: 'Dairy', pct: 40 },
		{ name: 'Gluten', pct: 85 },
		{ name: 'Tree Nuts', pct: 25 },
	];

	function rangeClass(pct) {
		if (pct < 33) return 'bg-success-500';
		if (pct < 66) return 'bg-warning-500';
		return 'bg-error-500';
	}

	function aqiClass(aqi) {
		if (aqi <= 50) return 'bg-success-500';
		if (aqi <= 100) return 'bg-warning-500';
		return 'bg-error-500';
	}

	let ipCity = $state('Loading...');
	let geoCity = $state('Loading...');
	let aqi = $state(null);

	async function fetchAqi(lat, lon) {
		try {
			const r = await fetch(
				`https://air-quality-api.open-meteo.com/v1/air-quality?latitude=${lat}&longitude=${lon}&current=us_aqi&forecast_days=1`
			);
			const d = await r.json();
			aqi = d.current?.us_aqi ?? null;
		} catch {}
	}

	let locationText = $derived((() => {
		const gpsKnown = geoCity !== 'Loading...' && geoCity !== 'Unknown';
		const ipKnown = ipCity !== 'Loading...' && ipCity !== 'Unknown';
		if (!gpsKnown && !ipKnown) {
			return geoCity === 'Loading...' || ipCity === 'Loading...' ? 'Loading...' : 'Unknown';
		}
		if (gpsKnown && ipKnown) return `${geoCity} (GPS) / ${ipCity} (IP)`;
		if (ipKnown) return `${ipCity} (IP) / ${geoCity} (GPS)`;
		return `${geoCity} (GPS) / ${ipCity} (IP)`;
	})());

	$effect(() => {
		fetch('https://ipinfo.io/json')
			.then((r) => r.json())
			.then((d) => {
				ipCity = d.city ?? 'Unknown';
				if (d.loc) {
					const [lat, lon] = d.loc.split(',').map(Number);
					fetchAqi(lat, lon);
				}
			})
			.catch(() => { ipCity = 'Unknown'; });

		navigator.geolocation.getCurrentPosition(
			async ({ coords }) => {
				fetchAqi(coords.latitude, coords.longitude);
				try {
					const r = await fetch(
						`https://nominatim.openstreetmap.org/reverse?lat=${coords.latitude}&lon=${coords.longitude}&format=json`
					);
					const d = await r.json();
					geoCity = d.address?.city ?? d.address?.town ?? d.address?.village ?? 'Unknown';
				} catch {
					geoCity = 'Unknown';
				}
			},
			() => { geoCity = 'Unknown'; }
		);
	});
</script>

<main class="max-w-lg mx-auto p-8 space-y-8">
	<div>
		<h1 class="h1">Allergen Tracker</h1>
		<h4 class="h4">Location: {locationText}</h4>
	</div>
	<div class="space-y-6">
		<div class="space-y-2">
			<div class="flex justify-between text-sm font-medium">
				<span>US AQI</span>
				<span>{aqi ?? '...'}</span>
			</div>
			<Progress value={aqi ?? 0} max={300}>
				<Progress.Track class="bg-surface-200-800 h-6 rounded overflow-hidden">
					<Progress.Range class="{aqi != null ? aqiClass(aqi) : ''} h-full rounded" />
				</Progress.Track>
			</Progress>
		</div>
		{#each allergens as allergen}
			<div class="space-y-2">
				<div class="flex justify-between text-sm font-medium">
					<span>{allergen.name}</span>
					<span>{allergen.pct}%</span>
				</div>
				<Progress value={allergen.pct} max={100}>
					<Progress.Track class="bg-surface-200-800 h-6 rounded overflow-hidden">
						<Progress.Range class="{rangeClass(allergen.pct)} h-full rounded" />
					</Progress.Track>
				</Progress>
			</div>
		{/each}
	</div>
</main>
