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
		if (pct < 33) return 'bg-error-500';
		if (pct < 66) return 'bg-warning-500';
		return 'bg-success-500';
	}

	let ipCity = $state('Loading...');
	let geoCity = $state('Loading...');

	$effect(() => {
		fetch('https://ipinfo.io/json')
			.then((r) => r.json())
			.then((d) => { ipCity = d.city ?? 'Unknown'; })
			.catch(() => { ipCity = 'Unknown'; });

		navigator.geolocation.getCurrentPosition(
			async ({ coords }) => {
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
		<h2 class="h2">Location (IP): {ipCity}</h2>
		<h2 class="h2">Location (GPS): {geoCity}</h2>
	</div>
	<div class="space-y-6">
		{#each allergens as allergen}
			<div class="space-y-2">
				<div class="flex justify-between text-sm font-medium">
					<span>{allergen.name}</span>
					<span>{allergen.pct}%</span>
				</div>
				<Progress value={allergen.pct} max={100}>
					<Progress.Track class="bg-surface-200-800 h-3 rounded-full overflow-hidden">
						<Progress.Range class="{rangeClass(allergen.pct)} h-full rounded-full" />
					</Progress.Track>
				</Progress>
			</div>
		{/each}
	</div>
</main>
