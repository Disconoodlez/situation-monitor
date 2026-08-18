<script lang="ts">
	import { onMount } from 'svelte';
	import { Panel } from '$lib/components/common';

	interface UAPSighting {
		id: string;
		date: string;
		city: string;
		state: string;
		country: string;
		lat: number;
		lon: number;
		shape: string;
		duration: string;
		summary: string;
		source: string;
	}

	// Seeded NUFORC-style sightings (real coordinates, representative descriptions)
	const SEED_SIGHTINGS: UAPSighting[] = [
		{
			id: 's1',
			date: '2026-03-28',
			city: 'Phoenix',
			state: 'AZ',
			country: 'USA',
			lat: 33.4484,
			lon: -112.074,
			shape: 'Triangle',
			duration: '4 min',
			summary: 'Silent triangular craft with white lights at each vertex, moving slowly NE toward Scottsdale',
			source: 'NUFORC'
		},
		{
			id: 's2',
			date: '2026-03-27',
			city: 'Chicago',
			state: 'IL',
			country: 'USA',
			lat: 41.8781,
			lon: -87.6298,
			shape: 'Orb',
			duration: '2 min',
			summary: 'Pulsing orange sphere hovering over Lake Michigan, performed 90° turn before accelerating upward',
			source: 'MUFON'
		},
		{
			id: 's3',
			date: '2026-03-26',
			city: 'Roswell',
			state: 'NM',
			country: 'USA',
			lat: 33.3943,
			lon: -104.5231,
			shape: 'Disk',
			duration: '8 min',
			summary: 'Classic saucer-shaped object stationary for several minutes then vanished without sound or trail',
			source: 'NUFORC'
		},
		{
			id: 's4',
			date: '2026-03-25',
			city: 'London',
			state: '',
			country: 'UK',
			lat: 51.5074,
			lon: -0.1278,
			shape: 'Light',
			duration: '10 min',
			summary: 'Formation of 5 white lights in chevron pattern traversed entire sky in under 10 minutes',
			source: 'UK-UAP'
		},
		{
			id: 's5',
			date: '2026-03-24',
			city: 'San Diego',
			state: 'CA',
			country: 'USA',
			lat: 32.7157,
			lon: -117.1611,
			shape: 'Tic-Tac',
			duration: '45 sec',
			summary: 'White elongated object near Naval Air Station North Island, no wings or exhaust, matched Nimitz description',
			source: 'NUFORC'
		},
		{
			id: 's6',
			date: '2026-03-23',
			city: 'Tokyo',
			state: '',
			country: 'Japan',
			lat: 35.6762,
			lon: 139.6503,
			shape: 'Sphere',
			duration: '3 min',
			summary: 'Metallic sphere tracked by multiple witnesses near Narita airport, ATC had no radar return',
			source: 'JUFORA'
		},
		{
			id: 's7',
			date: '2026-03-22',
			city: 'Houston',
			state: 'TX',
			country: 'USA',
			lat: 29.7604,
			lon: -95.3698,
			shape: 'Cylinder',
			duration: '6 min',
			summary: 'Dark cylindrical object drifting slowly over downtown, photographed by dozens of commuters',
			source: 'NUFORC'
		},
		{
			id: 's8',
			date: '2026-03-21',
			city: 'Melbourne',
			state: 'VIC',
			country: 'Australia',
			lat: -37.8136,
			lon: 144.9631,
			shape: 'Boomerang',
			duration: '12 min',
			summary: 'V-shaped craft with dim amber underside lights performed slow arc over Port Phillip Bay',
			source: 'AUFORN'
		},
		{
			id: 's9',
			date: '2026-03-20',
			city: 'Las Vegas',
			state: 'NV',
			country: 'USA',
			lat: 36.1699,
			lon: -115.1398,
			shape: 'Triangle',
			duration: '1 min',
			summary: 'Three lights in perfect triangle formation, no sound, rapidly gained altitude from standstill',
			source: 'NUFORC'
		},
		{
			id: 's10',
			date: '2026-03-19',
			city: 'São Paulo',
			state: 'SP',
			country: 'Brazil',
			lat: -23.5505,
			lon: -46.6333,
			shape: 'Orb',
			duration: '20 min',
			summary: 'Bright pulsing orb filmed by multiple residents, changed color from white to red, CINDACTA had no contact',
			source: 'ABRAFO'
		},
		{
			id: 's11',
			date: '2026-03-18',
			city: 'Seattle',
			state: 'WA',
			country: 'USA',
			lat: 47.6062,
			lon: -122.3321,
			shape: 'Disk',
			duration: '5 min',
			summary: 'Reflective disk hovering over Puget Sound, rotated in place before accelerating west',
			source: 'NUFORC'
		},
		{
			id: 's12',
			date: '2026-03-17',
			city: 'Mumbai',
			state: 'MH',
			country: 'India',
			lat: 19.076,
			lon: 72.8777,
			shape: 'Light',
			duration: '8 min',
			summary: 'Column of 4 white lights ascending vertically into overcast sky over Arabian Sea',
			source: 'UFOINDIA'
		}
	];

	let sightings = $state<UAPSighting[]>(SEED_SIGHTINGS);
	let selectedSighting = $state<UAPSighting | null>(null);
	let viewMode = $state<'map' | 'list'>('map');
	let loading = $state(false);
	let mapContainer: HTMLDivElement;

	// Shape color coding
	const SHAPE_COLORS: Record<string, string> = {
		Triangle: '#a855f7',
		'Tic-Tac': '#00ffcc',
		Disk: '#ff6b35',
		Orb: '#ffd700',
		Sphere: '#ffd700',
		Cylinder: '#60a5fa',
		Boomerang: '#f472b6',
		Light: '#ffffff'
	};

	function getShapeColor(shape: string): string {
		return SHAPE_COLORS[shape] || '#00ff88';
	}

	function getDaysAgo(dateStr: string): string {
		const diff = Math.floor((Date.now() - new Date(dateStr).getTime()) / 86400000);
		if (diff === 0) return 'Today';
		if (diff === 1) return '1d ago';
		return `${diff}d ago`;
	}

	// Initialize D3 mini-map
	async function initMap(): Promise<void> {
		if (!mapContainer) return;
		const d3 = await import('d3');
		const topojson = await import('topojson-client');

		const WIDTH = 600;
		const HEIGHT = 300;

		const svgEl = mapContainer.querySelector('svg');
		if (!svgEl) return;

		const svg = d3.select(svgEl).attr('viewBox', `0 0 ${WIDTH} ${HEIGHT}`);
		const g = svg.append('g');

		const projection = d3
			.geoEquirectangular()
			.scale(95)
			.center([0, 20])
			.translate([WIDTH / 2, HEIGHT / 2 - 10]);

		const path = d3.geoPath().projection(projection);

		// Zoom and pan
		const zoom = d3
			.zoom<SVGSVGElement, unknown>()
			.scaleExtent([1, 8])
			.filter((event) => {
				if (event.type === 'wheel') return false;
				if (event.type === 'dblclick') return false;
				return true;
			})
			.on('zoom', (event) => {
				g.attr('transform', event.transform.toString());
			});

		svg.call(zoom);

		// Load world topology
		try {
			const response = await fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/countries-110m.json');
			const world = await response.json();
			// eslint-disable-next-line @typescript-eslint/no-explicit-any
			const countries = topojson.feature(world, world.objects.countries as any) as unknown as GeoJSON.FeatureCollection;

			g.selectAll('path.country')
				.data(countries.features)
				.enter()
				.append('path')
				.attr('class', 'country')
				// eslint-disable-next-line @typescript-eslint/no-explicit-any
				.attr('d', path as any)
				.attr('fill', '#0a2a1f')
				.attr('stroke', '#1a4a35')
				.attr('stroke-width', '0.4');
		} catch {
			// Draw ocean fallback
			g.append('rect').attr('width', WIDTH).attr('height', HEIGHT).attr('fill', '#081a12');
		}

		// Plot sighting markers
		sightings.forEach((s) => {
			const coords = projection([s.lon, s.lat]);
			if (!coords) return;
			const [x, y] = coords;
			const color = getShapeColor(s.shape);

			// Pulse ring
			const pulseGroup = g.append('g').attr('class', 'uap-marker').style('cursor', 'pointer');

			pulseGroup
				.append('circle')
				.attr('cx', x)
				.attr('cy', y)
				.attr('r', 6)
				.attr('fill', 'none')
				.attr('stroke', color)
				.attr('stroke-width', 1)
				.attr('opacity', 0.4)
				.style('animation', `uapPulse 2s ease-out infinite`);

			// Core dot
			pulseGroup
				.append('circle')
				.attr('cx', x)
				.attr('cy', y)
				.attr('r', 3)
				.attr('fill', color)
				.attr('stroke', '#000')
				.attr('stroke-width', 0.5);

			pulseGroup.on('click', () => {
				selectedSighting = s;
			});
		});
	}

	onMount(() => {
		if (viewMode === 'map') {
			initMap();
		}
	});

	function switchToMap() {
		viewMode = 'map';
		setTimeout(() => initMap(), 50);
	}
</script>

<Panel id="uap" title="UAP / UFO Sightings" count={sightings.length} status="LIVE" statusClass="elevated" {loading}>
	{#snippet actions()}
		<div class="view-toggle">
			<button class:active={viewMode === 'map'} onclick={switchToMap}>Map</button>
			<button class:active={viewMode === 'list'} onclick={() => (viewMode = 'list')}>List</button>
		</div>
	{/snippet}

	<div class="uap-panel">
		{#if viewMode === 'map'}
			<div class="map-wrap" bind:this={mapContainer}>
				<svg class="uap-map"></svg>
				<div class="map-hint">Click marker for details · Drag to pan</div>
			</div>

			{#if selectedSighting}
				<div class="sighting-detail">
					<div class="detail-header">
						<span class="shape-badge" style="color: {getShapeColor(selectedSighting.shape)}">
							◈ {selectedSighting.shape}
						</span>
						<span class="detail-loc">{selectedSighting.city}{selectedSighting.state ? ', ' + selectedSighting.state : ''}, {selectedSighting.country}</span>
						<span class="detail-date">{getDaysAgo(selectedSighting.date)}</span>
					</div>
					<p class="detail-summary">{selectedSighting.summary}</p>
					<div class="detail-meta">
						<span>⏱ {selectedSighting.duration}</span>
						<span>📡 {selectedSighting.source}</span>
					</div>
				</div>
			{:else}
				<div class="map-prompt">Select a marker to view report details</div>
			{/if}
		{:else}
			<div class="sighting-list">
				{#each sightings as s (s.id)}
					<button class="sighting-row" class:selected={selectedSighting?.id === s.id} onclick={() => (selectedSighting = s)}>
						<div class="row-top">
							<span class="shape-dot" style="background: {getShapeColor(s.shape)}"></span>
							<span class="row-shape">{s.shape}</span>
							<span class="row-loc">{s.city}{s.state ? ', ' + s.state : ''}</span>
							<span class="row-date">{getDaysAgo(s.date)}</span>
						</div>
						{#if selectedSighting?.id === s.id}
							<div class="row-detail">
								<p>{s.summary}</p>
								<div class="row-meta">⏱ {s.duration} · 📡 {s.source}</div>
							</div>
						{/if}
					</button>
				{/each}
			</div>
		{/if}

		<!-- Shape legend -->
		<div class="legend">
			{#each Object.entries(SHAPE_COLORS) as [shape, color] (shape)}
				<div class="legend-item">
					<span class="legend-dot" style="background: {color}"></span>
					<span>{shape}</span>
				</div>
			{/each}
		</div>
	</div>
</Panel>

<style>
	.uap-panel {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.map-wrap {
		position: relative;
		background: #081812;
		border: 1px solid #1a3a2a;
		border-radius: 3px;
		overflow: hidden;
	}

	.uap-map {
		width: 100%;
		height: 200px;
		display: block;
	}

	.map-hint {
		position: absolute;
		bottom: 4px;
		right: 6px;
		font-size: 0.55rem;
		color: rgba(255, 255, 255, 0.25);
		pointer-events: none;
	}

	.map-prompt {
		text-align: center;
		font-size: 0.65rem;
		color: var(--text-secondary);
		padding: 0.5rem;
	}

	.sighting-detail {
		background: rgba(168, 85, 247, 0.07);
		border: 1px solid rgba(168, 85, 247, 0.25);
		border-radius: 3px;
		padding: 0.5rem;
	}

	.detail-header {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin-bottom: 0.35rem;
		flex-wrap: wrap;
	}

	.shape-badge {
		font-size: 0.65rem;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.detail-loc {
		font-size: 0.65rem;
		color: var(--text-primary);
		flex: 1;
	}

	.detail-date {
		font-size: 0.6rem;
		color: var(--text-secondary);
	}

	.detail-summary {
		font-size: 0.65rem;
		color: var(--text-primary);
		margin: 0 0 0.35rem;
		line-height: 1.5;
	}

	.detail-meta {
		display: flex;
		gap: 0.75rem;
		font-size: 0.6rem;
		color: var(--text-secondary);
	}

	/* List view */
	.sighting-list {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
		max-height: 280px;
		overflow-y: auto;
	}

	.sighting-row {
		all: unset;
		display: block;
		padding: 0.35rem 0.5rem;
		border: 1px solid var(--border);
		border-radius: 3px;
		cursor: pointer;
		transition: background 0.15s;
	}

	.sighting-row:hover,
	.sighting-row.selected {
		background: rgba(168, 85, 247, 0.08);
		border-color: rgba(168, 85, 247, 0.3);
	}

	.row-top {
		display: flex;
		align-items: center;
		gap: 0.4rem;
	}

	.shape-dot {
		width: 7px;
		height: 7px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	.row-shape {
		font-size: 0.6rem;
		font-weight: 600;
		color: var(--text-secondary);
		text-transform: uppercase;
		min-width: 55px;
	}

	.row-loc {
		font-size: 0.65rem;
		color: var(--text-primary);
		flex: 1;
	}

	.row-date {
		font-size: 0.6rem;
		color: var(--text-secondary);
	}

	.row-detail {
		margin-top: 0.35rem;
		padding-top: 0.35rem;
		border-top: 1px solid var(--border);
	}

	.row-detail p {
		font-size: 0.63rem;
		color: var(--text-primary);
		margin: 0 0 0.25rem;
		line-height: 1.5;
	}

	.row-meta {
		font-size: 0.6rem;
		color: var(--text-secondary);
	}

	/* Legend */
	.legend {
		display: flex;
		flex-wrap: wrap;
		gap: 0.4rem 0.75rem;
		padding-top: 0.35rem;
		border-top: 1px solid var(--border);
	}

	.legend-item {
		display: flex;
		align-items: center;
		gap: 0.25rem;
		font-size: 0.55rem;
		color: var(--text-secondary);
	}

	.legend-dot {
		width: 6px;
		height: 6px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	/* View toggle */
	.view-toggle {
		display: flex;
		gap: 2px;
	}

	.view-toggle button {
		background: none;
		border: 1px solid var(--border);
		color: var(--text-secondary);
		padding: 0.1rem 0.4rem;
		font-size: 0.55rem;
		cursor: pointer;
		border-radius: 2px;
		text-transform: uppercase;
		letter-spacing: 0.05em;
	}

	.view-toggle button.active {
		background: rgba(var(--accent-rgb), 0.15);
		border-color: var(--accent);
		color: var(--accent);
	}

	:global(.uap-marker circle:first-child) {
		animation: uapPulse 2s ease-out infinite;
	}

	@keyframes uapPulse {
		0% { r: 3; opacity: 0.6; }
		100% { r: 10; opacity: 0; }
	}
</style>
