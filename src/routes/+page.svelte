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

	let city = $state(null);

	$effect(() => {
		fetch('https://ipapi.co/json/')
			.then((r) => r.json())
			.then((d) => { city = d.city; });
	});
</script>

<main class="max-w-lg mx-auto p-8 space-y-8">
	<div>
		<h1 class="h1">Allergen Tracker</h1>
		{#if city}
			<p class="text-surface-400 mt-1">{city}</p>
		{/if}
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
