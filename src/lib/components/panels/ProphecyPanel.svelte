<script lang="ts">
	import { Panel } from '$lib/components/common';
	import { allNewsItems } from '$lib/stores';
	import type { NewsItem } from '$lib/types';

	interface ProphecyTheme {
		id: string;
		reference: string;
		book: string;
		verses: string;
		theme: string;
		description: string;
		keywords: string[];
		status: 'watching' | 'active' | 'escalating';
	}

	const PROPHECY_THEMES: ProphecyTheme[] = [
		{
			id: 'gog-magog',
			reference: 'Ezekiel 38–39',
			book: 'Ezekiel',
			verses: '38:2–6',
			theme: 'Gog / Magog Coalition',
			description:
				'A great northern power (Gog) leads a coalition against Israel including Persia (Iran), Cush, and Put. Linked historically to Russia + Iran alliance.',
			keywords: ['russia', 'iran', 'israel', 'coalition', 'northern', 'gog', 'alliance', 'strike'],
			status: 'escalating'
		},
		{
			id: 'mark-beast',
			reference: 'Revelation 13',
			book: 'Revelation',
			verses: '13:16–17',
			theme: 'Mark of the Beast / Economic Control',
			description:
				'No one can buy or sell without a mark. Tied to CBDC implementation, digital IDs, and centralized transaction monitoring.',
			keywords: ['cbdc', 'digital currency', 'digital id', 'surveillance', 'social credit', 'ban', 'transaction'],
			status: 'active'
		},
		{
			id: 'wars-rumors',
			reference: 'Matthew 24',
			book: 'Matthew',
			verses: '24:6–7',
			theme: 'Wars and Rumors of Wars',
			description:
				'Nation rising against nation, kingdom against kingdom. Multiple simultaneous conflict theaters.',
			keywords: ['war', 'conflict', 'attack', 'military', 'strike', 'invasion', 'offensive', 'missile', 'troops'],
			status: 'escalating'
		},
		{
			id: 'desolation',
			reference: 'Matthew 24',
			book: 'Matthew',
			verses: '24:15–16',
			theme: 'Abomination of Desolation',
			description:
				'Desecration of the holy place. Linked to events at Temple Mount, Jerusalem, and Israeli sovereignty disputes.',
			keywords: ['temple mount', 'jerusalem', 'al-aqsa', 'holy site', 'israel', 'sacrifice'],
			status: 'watching'
		},
		{
			id: 'pestilence',
			reference: 'Matthew 24 / Revelation 6',
			book: 'Revelation',
			verses: '6:8',
			theme: 'Pestilence / Plague',
			description:
				'Death given power over a fourth of the earth through sword, famine, plague, and wild beasts.',
			keywords: ['pandemic', 'outbreak', 'disease', 'virus', 'plague', 'epidemic', 'mpox', 'bird flu'],
			status: 'watching'
		},
		{
			id: 'one-world-govt',
			reference: 'Revelation 13',
			book: 'Revelation',
			verses: '13:1–4',
			theme: 'One World Authority / Beast System',
			description:
				'A global power with authority over all peoples, tribes, and tongues. Tied to UN agenda, WEF Great Reset, AI governance proposals.',
			keywords: ['un', 'global governance', 'world government', 'wef', 'great reset', 'digital governance', 'ai regulation'],
			status: 'active'
		},
		{
			id: 'israel-surrounded',
			reference: 'Luke 21',
			book: 'Luke',
			verses: '21:20',
			theme: 'Jerusalem Surrounded by Armies',
			description:
				'When Jerusalem is surrounded by armies, its desolation is near. Tracks military encirclement of Israel.',
			keywords: ['hezbollah', 'hamas', 'gaza', 'west bank', 'siege', 'encircle', 'border', 'israel surrounded'],
			status: 'escalating'
		},
		{
			id: 'apostasy',
			reference: '2 Thessalonians 2',
			book: '2 Thessalonians',
			verses: '2:3',
			theme: 'Great Apostasy / Falling Away',
			description:
				'A great falling away precedes the Day of the Lord. Tied to declining church attendance, institutional religious decline.',
			keywords: ['church decline', 'atheism', 'secularism', 'apostasy', 'religious decline', 'faith', 'deconversion'],
			status: 'watching'
		}
	];

	const STATUS_CONFIG = {
		escalating: { label: 'ESCALATING', color: '#ff4444', bg: 'rgba(255,68,68,0.1)' },
		active: { label: 'ACTIVE', color: '#ffcc00', bg: 'rgba(255,204,0,0.1)' },
		watching: { label: 'WATCHING', color: '#00ff88', bg: 'rgba(0,255,136,0.05)' }
	};

	let selectedTheme = $state<ProphecyTheme | null>(null);
	let filterStatus = $state<'all' | 'escalating' | 'active' | 'watching'>('all');

	const newsItems = $derived($allNewsItems);

	function getMatchingNews(theme: ProphecyTheme): NewsItem[] {
		const kws = theme.keywords.map((k) => k.toLowerCase());
		return newsItems
			.filter((item) => {
				const text = `${item.title} ${item.description || ''}`.toLowerCase();
				return kws.some((kw) => text.includes(kw));
			})
			.slice(0, 5);
	}

	const filteredThemes = $derived(
		filterStatus === 'all'
			? PROPHECY_THEMES
			: PROPHECY_THEMES.filter((t) => t.status === filterStatus)
	);

	const escalatingCount = $derived(PROPHECY_THEMES.filter((t) => t.status === 'escalating').length);
	const activeCount = $derived(PROPHECY_THEMES.filter((t) => t.status === 'active').length);

	function getMatchCount(theme: ProphecyTheme): number {
		return getMatchingNews(theme).length;
	}
</script>

<Panel
	id="prophecy"
	title="Prophecy Tracker"
	count="{escalatingCount} escalating"
	status="DATA ONLY"
	statusClass="monitoring"
>
	<div class="prophecy-panel">
		<div class="disclaimer">
			Correlating verifiable news events to textual prophetic themes. No interpretation implied.
		</div>

		<!-- Filter tabs -->
		<div class="filter-tabs">
			<button class:active={filterStatus === 'all'} onclick={() => (filterStatus = 'all')}>All ({PROPHECY_THEMES.length})</button>
			<button class:active={filterStatus === 'escalating'} onclick={() => (filterStatus = 'escalating')}>
				<span class="dot" style="background:#ff4444"></span>Escalating ({escalatingCount})
			</button>
			<button class:active={filterStatus === 'active'} onclick={() => (filterStatus = 'active')}>
				<span class="dot" style="background:#ffcc00"></span>Active ({activeCount})
			</button>
			<button class:active={filterStatus === 'watching'} onclick={() => (filterStatus = 'watching')}>
				<span class="dot" style="background:#00ff88"></span>Watching
			</button>
		</div>

		<!-- Theme list -->
		<div class="theme-list">
			{#each filteredThemes as theme (theme.id)}
				{@const cfg = STATUS_CONFIG[theme.status]}
				{@const matches = getMatchCount(theme)}
				<button
					class="theme-row"
					class:selected={selectedTheme?.id === theme.id}
					onclick={() => (selectedTheme = selectedTheme?.id === theme.id ? null : theme)}
					style="border-left-color: {cfg.color}"
				>
					<div class="theme-top">
						<div class="theme-ref">{theme.reference}</div>
						<div class="theme-name">{theme.theme}</div>
						<div class="theme-badges">
							{#if matches > 0}
								<span class="match-badge">{matches} hits</span>
							{/if}
							<span class="status-badge" style="color:{cfg.color}; background:{cfg.bg}">{cfg.label}</span>
						</div>
					</div>

					{#if selectedTheme?.id === theme.id}
						<div class="theme-detail">
							<p class="theme-desc">{theme.description}</p>
							<div class="keyword-list">
								{#each theme.keywords as kw (kw)}
									<span class="kw-chip">{kw}</span>
								{/each}
							</div>

							{#if matches > 0}
								<div class="news-hits">
									<div class="hits-label">Current Event Correlations</div>
									{#each getMatchingNews(theme) as item (item.id)}
										<a href={item.link} target="_blank" rel="noopener noreferrer" class="hit-item">
											<span class="hit-source">{item.source}</span>
											<span class="hit-title">{item.title}</span>
										</a>
									{/each}
								</div>
							{:else}
								<div class="no-hits">No current news correlations detected</div>
							{/if}
						</div>
					{/if}
				</button>
			{/each}
		</div>
	</div>
</Panel>

<style>
	.prophecy-panel {
		display: flex;
		flex-direction: column;
		gap: 0.5rem;
	}

	.disclaimer {
		font-size: 0.58rem;
		color: var(--text-secondary);
		background: rgba(255, 255, 255, 0.03);
		border: 1px solid var(--border);
		border-radius: 3px;
		padding: 0.35rem 0.5rem;
		line-height: 1.4;
		font-style: italic;
	}

	.filter-tabs {
		display: flex;
		gap: 0.25rem;
		flex-wrap: wrap;
	}

	.filter-tabs button {
		all: unset;
		display: flex;
		align-items: center;
		gap: 0.25rem;
		font-size: 0.58rem;
		padding: 0.15rem 0.4rem;
		border: 1px solid var(--border);
		border-radius: 2px;
		cursor: pointer;
		color: var(--text-secondary);
		text-transform: uppercase;
		letter-spacing: 0.04em;
		transition: all 0.15s;
	}

	.filter-tabs button.active {
		background: rgba(var(--accent-rgb), 0.12);
		border-color: var(--accent);
		color: var(--accent);
	}

	.dot {
		width: 6px;
		height: 6px;
		border-radius: 50%;
		flex-shrink: 0;
	}

	.theme-list {
		display: flex;
		flex-direction: column;
		gap: 0.3rem;
		max-height: 400px;
		overflow-y: auto;
	}

	.theme-row {
		all: unset;
		display: block;
		padding: 0.4rem 0.5rem;
		border: 1px solid var(--border);
		border-left-width: 3px;
		border-radius: 3px;
		cursor: pointer;
		transition: background 0.15s;
	}

	.theme-row:hover {
		background: rgba(255, 255, 255, 0.03);
	}

	.theme-row.selected {
		background: rgba(255, 255, 255, 0.04);
	}

	.theme-top {
		display: flex;
		align-items: center;
		gap: 0.4rem;
		flex-wrap: wrap;
	}

	.theme-ref {
		font-size: 0.58rem;
		color: var(--text-secondary);
		font-family: monospace;
		min-width: 80px;
	}

	.theme-name {
		font-size: 0.65rem;
		font-weight: 600;
		color: var(--text-primary);
		flex: 1;
	}

	.theme-badges {
		display: flex;
		gap: 0.25rem;
		align-items: center;
	}

	.match-badge {
		font-size: 0.55rem;
		padding: 0.1rem 0.35rem;
		background: rgba(var(--accent-rgb), 0.15);
		color: var(--accent);
		border-radius: 2px;
	}

	.status-badge {
		font-size: 0.55rem;
		font-weight: 700;
		padding: 0.1rem 0.35rem;
		border-radius: 2px;
		letter-spacing: 0.04em;
	}

	/* Detail view */
	.theme-detail {
		margin-top: 0.5rem;
		padding-top: 0.5rem;
		border-top: 1px solid var(--border);
		display: flex;
		flex-direction: column;
		gap: 0.4rem;
	}

	.theme-desc {
		font-size: 0.63rem;
		color: var(--text-primary);
		line-height: 1.5;
		margin: 0;
	}

	.keyword-list {
		display: flex;
		flex-wrap: wrap;
		gap: 0.25rem;
	}

	.kw-chip {
		font-size: 0.55rem;
		padding: 0.1rem 0.35rem;
		background: rgba(255, 255, 255, 0.06);
		border: 1px solid var(--border);
		border-radius: 2px;
		color: var(--text-secondary);
		font-family: monospace;
	}

	.news-hits {
		display: flex;
		flex-direction: column;
		gap: 0.25rem;
	}

	.hits-label {
		font-size: 0.58rem;
		font-weight: 600;
		text-transform: uppercase;
		letter-spacing: 0.05em;
		color: var(--text-secondary);
	}

	.hit-item {
		display: flex;
		gap: 0.4rem;
		align-items: flex-start;
		text-decoration: none;
		padding: 0.25rem 0.4rem;
		background: rgba(255, 255, 255, 0.03);
		border-radius: 2px;
		border: 1px solid var(--border);
		transition: background 0.1s;
	}

	.hit-item:hover {
		background: rgba(255, 255, 255, 0.06);
	}

	.hit-source {
		font-size: 0.58rem;
		color: var(--accent);
		white-space: nowrap;
		min-width: 50px;
		font-weight: 600;
	}

	.hit-title {
		font-size: 0.6rem;
		color: var(--text-primary);
		line-height: 1.4;
	}

	.no-hits {
		font-size: 0.6rem;
		color: var(--text-secondary);
		font-style: italic;
		padding: 0.25rem 0;
	}
</style>
