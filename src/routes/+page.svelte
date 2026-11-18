<script lang="ts">
	import { enhance } from '$app/forms';
	import { onMount } from 'svelte';
	import ProjectRow from '$lib/components/ProjectRow.svelte';

	type Theme = 'light' | 'dark';
	type Status = 'idle' | 'loading' | 'success' | 'error';

	let theme: Theme = $state('light');
	let status: Status = $state('idle');
	let selectedCategory: string = $state('SEMUA');
	let copied: boolean = $state(false);
	let mobileMenuOpen: boolean = $state(false);
	let showScrollTop: boolean = $state(false);

	// State untuk fitur Now Playing
	let isPlaying: boolean = $state(true);
	let audioEl: HTMLAudioElement | undefined = $state();

	const nowPlaying = {
		title: 'Coffee Time',
		artist: 'None',
		audioSrc: '/Audio/[no copyright music] _ coffee time _ cute vlog music.mp3'
	};

	// State untuk Tell Me Something (Guestbook)
	type GuestMessage = {
		name: string;
		message: string;
		date: string;
	};

	let guestName = $state('');
	let guestMessage = $state('');
	let guestList = $state<GuestMessage[]>([]);

	// State untuk Game Center Launcher ('none' | 'bug' | 'memory')
	let activeGame: 'none' | 'bug' | 'memory' = $state('none');

	// State untuk Mini-Game "Catch the Bug"
	let bugGameStarted = $state(false);
	let bugScore = $state(0);
	let bugTimeLeft = $state(15);
	let bugPosition = $state({ top: 50, left: 50 });
	let bugTimerInterval: any = null;

	// State untuk Mini-Game "Memory Card" (6 Pasang = 12 Kartu, Waktu 30 Detik)
	type Card = {
		id: number;
		icon: string;
		isFlipped: boolean;
		isMatched: boolean;
	};
	const cardIcons = ['🚀', '💻', '☕', '⚡', '🎮', '🔥'];
	let memoryCards = $state<Card[]>([]);
	let flippedCards = $state<Card[]>([]);
	let memoryScore = $state(0);
	let memoryTimeLeft = $state(30);
	let memoryGameOver = $state(false);
	let isChecking = $state(false);
	let memoryTimerInterval: any = null;

	const categories = ['SEMUA', ...new Set(projects.map((p) => p.category))];

	let filteredProjects = $derived(
		selectedCategory === 'SEMUA'
			? projects
			: projects.filter((p) => p.category === selectedCategory)
	);

	onMount(() => {
		const savedTheme = localStorage.getItem('theme') as Theme | null;
		if (savedTheme === 'dark' || savedTheme === 'light') {
			theme = savedTheme;
		}

		const handleScroll = () => {
			showScrollTop = window.scrollY > 450;
		};
		window.addEventListener('scroll', handleScroll);
		return () => {
			window.removeEventListener('scroll', handleScroll);
			if (bugTimerInterval) clearInterval(bugTimerInterval);
			if (memoryTimerInterval) clearInterval(memoryTimerInterval);
		};
	});

	function toggleTheme() {
		theme = theme === 'dark' ? 'light' : 'dark';
		localStorage.setItem('theme', theme);
	}

	function toggleMusic() {
		isPlaying = !isPlaying;
		if (!audioEl) return;
		if (isPlaying) {
			audioEl.play().catch(() => {});
		} else {
			audioEl.pause();
		}
	}

	function addGuestMessage(e: Event) {
		e.preventDefault();
		if (!guestName.trim() || !guestMessage.trim()) return;

		const newMessage: GuestMessage = {
			name: guestName.trim(),
			message: guestMessage.trim(),
			date: new Date().toLocaleDateString('id-ID', { day: 'numeric', month: 'short', year: 'numeric' })
		};

		guestList = [newMessage, ...guestList];

		guestName = '';
		guestMessage = '';
	}

	function exitGameMenu() {
		activeGame = 'none';
		bugGameStarted = false;
		if (bugTimerInterval) clearInterval(bugTimerInterval);
		if (memoryTimerInterval) clearInterval(memoryTimerInterval);
	}

	// Fungsi Catch the Bug
	function startBugGame() {
		bugScore = 0;
		bugTimeLeft = 15;
		bugGameStarted = true;
		moveBug();

		if (bugTimerInterval) clearInterval(bugTimerInterval);
		bugTimerInterval = setInterval(() => {
			bugTimeLeft -= 1;
			if (bugTimeLeft <= 0) {
				clearInterval(bugTimerInterval);
				bugGameStarted = false;
			}
		}, 1000);
	}

	function moveBug() {
		if (!bugGameStarted) return;
		bugPosition = {
			top: Math.floor(Math.random() * 70) + 15,
			left: Math.floor(Math.random() * 80) + 10
		};
	}

	function catchBug() {
		if (!bugGameStarted) return;
		bugScore += 1;
		moveBug();
	}

	// Fungsi Memory Card
	function startMemoryGame() {
		memoryScore = 0;
		memoryTimeLeft = 30;
		memoryGameOver = false;
		flippedCards = [];
		isChecking = false;

		const duplicated = [...cardIcons, ...cardIcons];
		memoryCards = duplicated
			.sort(() => Math.random() - 0.5)
			.map((icon, index) => ({
				id: index,
				icon,
				isFlipped: false,
				isMatched: false
			}));

		if (memoryTimerInterval) clearInterval(memoryTimerInterval);
		memoryTimerInterval = setInterval(() => {
			memoryTimeLeft -= 1;
			if (memoryTimeLeft <= 0) {
				clearInterval(memoryTimerInterval);
				memoryGameOver = true;
			}
		}, 1000);
	}

	function flipCard(card: Card) {
		if (memoryGameOver || isChecking || card.isFlipped || card.isMatched) return;

		card.isFlipped = true;
		flippedCards = [...flippedCards, card];

		if (flippedCards.length === 2) {
			isChecking = true;
			if (flippedCards[0].icon === flippedCards[1].icon) {
				flippedCards[0].isMatched = true;
				flippedCards[1].isMatched = true;
				memoryScore += 1;
				flippedCards = [];
				isChecking = false;

				if (memoryScore === cardIcons.length && memoryTimerInterval) {
					clearInterval(memoryTimerInterval);
				}
			} else {
				setTimeout(() => {
					flippedCards[0].isFlipped = false;
					flippedCards[1].isFlipped = false;
					flippedCards = [];
					isChecking = false;
				}, 700);
			}
		}
	}

	async function copyEmail() {
		try {
			await navigator.clipboard.writeText('keishaputriratri@gmail.com');
			copied = true;
			setTimeout(() => (copied = false), 2000);
		} catch {}
	}

	function scrollToTop() {
		window.scrollTo({ top: 0, behavior: 'smooth' });
	}
</script>

<!-- Audio Player Tersembunyi -->
<audio bind:this={audioEl} src={nowPlaying.audioSrc} loop></audio>

<div class="min-h-dvh bg-bg text-ink transition-colors duration-300 {theme === 'dark' ? 'dark' : ''}">
	<!-- nav -->
	<header class="sticky top-0 z-50 backdrop-blur-md bg-bg/80 border-b border-line">
		<div
			class="mx-auto flex h-[60px] max-w-[1440px] items-center justify-between px-6 lg:h-[72px] lg:px-24"
		>
			<a href="/" class="text-[15px] font-semibold tracking-tight hover:opacity-85 transition-opacity lg:text-base">
				Portofolio Kei <span class="text-primary font-mono text-xs">:p</span>
			</a>

			<div class="flex items-center gap-3 lg:gap-8">
				<nav class="hidden items-center gap-7 lg:flex">
					<a href="#proyek" class="text-sm text-muted hover:text-ink transition-colors">Proyek</a>
					<a href="#tentang" class="text-sm text-muted hover:text-ink transition-colors">Tentang</a>
					<a href="#minigame" class="text-sm text-muted hover:text-ink transition-colors">Arcade Game</a>
					<a href="#guestbook" class="text-sm text-muted hover:text-ink transition-colors">Tell Me Something</a>
					<a href="#kontak" class="text-sm text-muted hover:text-ink transition-colors">Kontak</a>
				</nav>

				<!-- TOMBOL TEMA -->
				<button
					type="button"
					aria-label="Ganti tema"
					onclick={toggleTheme}
					class="flex h-10 w-10 items-center justify-center rounded-xl border border-line bg-surface hover:border-primary transition-all active:scale-95 lg:h-11 lg:w-11 shadow-sm"
				>
					<svg
						width="17"
						height="17"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						stroke-linejoin="round"
						class="dark:hidden transition-transform hover:rotate-45"
					>
						<path d="M21 12.8A9 9 0 1 1 11.2 3a7 7 0 0 0 9.8 9.8Z" />
					</svg>
					<svg
						width="17"
						height="17"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						class="hidden dark:block transition-transform hover:rotate-90"
					>
						<circle cx="12" cy="12" r="4" />
						<path
							d="M12 2v2M12 20v2M4.9 4.9l1.4 1.4M17.7 17.7l1.4 1.4M2 12h2M20 12h2M4.9 19.1l1.4-1.4M17.7 6.3l1.4-1.4"
						/>
					</svg>
				</button>

				<!-- TOMBOL MENU MOBILE -->
				<button
					type="button"
					aria-label="Buka menu navigasi"
					onclick={() => (mobileMenuOpen = !mobileMenuOpen)}
					class="flex h-10 w-10 items-center justify-center rounded-xl border border-line bg-surface hover:border-primary transition-all lg:hidden shadow-sm"
				>
					<svg
						width="18"
						height="18"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
					>
						{#if mobileMenuOpen}
							<path d="M18 6 6 18M6 6l12 12" />
						{:else}
							<path d="M4 7h16M4 12h16M4 17h16" />
						{/if}
					</svg>
				</button>
			</div>
		</div>

		<!-- DROPDOWN MENU MOBILE -->
		{#if mobileMenuOpen}
			<nav class="flex flex-col gap-3 border-t border-line bg-bg px-6 py-4 lg:hidden">
				<a
					href="#proyek"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					Proyek
				</a>
				<a
					href="#tentang"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					Tentang
				</a>
				<a
					href="#minigame"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					Arcade Game
				</a>
				<a
					href="#guestbook"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					Tell Me Something
				</a>
				<a
					href="#kontak"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					Kontak
				</a>
			</nav>
		{/if}
	</header>

	<!-- hero -->
	<section class="mx-auto max-w-[1440px] px-6 pt-16 pb-14 lg:px-24 lg:pt-[120px]">
		<h1
			class="text-[52px] leading-[0.92] font-semibold tracking-[-2px] lg:text-[130px] lg:tracking-[-6px] text-ink"
		>
			Keisha Putri Ratri
		</h1>

		<div class="mt-7 flex flex-col gap-6">
			<p class="max-w-[560px] text-[17px] leading-[1.6] text-muted lg:text-xl">
				Siswa SMK jurusan PPLG. Membangun antarmuka web yang cepat, rapi, dan mudah dipakai.
			</p>

			<div class="flex flex-wrap items-center gap-3">
				<a
					href="#proyek"
					class="flex h-12 items-center rounded-xl bg-primary px-6 text-[15px] font-medium text-on-primary shadow-sm hover:opacity-95 transition-all"
				>
					Lihat proyek
				</a>
				<a
					href="#kontak"
					class="flex h-12 items-center rounded-xl border border-line px-6 text-[15px] text-ink hover:border-primary transition-all"
				>
					Hubungi saya
				</a>
			</div>

			<div class="flex flex-col justify-between gap-4 pt-2 sm:flex-row sm:items-center">
				<div class="flex items-center gap-2">
					<span class="h-2 w-2 rounded-full bg-success animate-pulse"></span>
					<p class="font-mono text-[11px] tracking-[1.1px] text-muted">
						TERSEDIA UNTUK PROYEK BARU — 2026
					</p>
				</div>

				<!-- WIDGET NOW PLAYING -->
				<button
					type="button"
					onclick={toggleMusic}
					title="Klik untuk Play / Pause audio"
					class="flex w-fit items-center gap-3.5 rounded-2xl border border-line bg-surface px-4 py-2.5 text-left shadow-sm transition-all hover:border-primary active:scale-98"
				>
					<!-- Animasi Equalizer Bar -->
					<div class="flex h-4 items-end gap-[3px]">
						<span
							class="w-[3px] rounded-full bg-primary transition-all duration-300 {isPlaying
								? 'h-4 animate-bounce'
								: 'h-1.5'}"
						></span>
						<span
							class="w-[3px] rounded-full bg-primary transition-all duration-300 {isPlaying
								? 'h-2.5 animate-pulse'
								: 'h-1.5'}"
						></span>
						<span
							class="w-[3px] rounded-full bg-primary transition-all duration-300 {isPlaying
								? 'h-3.5 animate-bounce'
								: 'h-1.5'}"
						></span>
					</div>

					<div class="flex flex-col">
						<span class="font-mono text-[10px] tracking-[1.1px] text-primary uppercase">
							{isPlaying ? 'NOW PLAYING' : 'PAUSED'}
						</span>
						<span class="text-xs font-medium text-ink">
							{nowPlaying.title} <span class="text-muted font-normal">— {nowPlaying.artist}</span>
						</span>
					</div>
				</button>
			</div>
		</div>
	</section>

	<!-- band (GIF Cat Working) -->
	<div class="border-t border-line overflow-hidden">
		<img
			src="/images/Cat Working GIF.gif"
			alt="Cat Working"
			class="h-[220px] w-full object-cover lg:h-[420px]"
		/>
		<div class="mx-auto max-w-[1440px] px-6 py-4 lg:px-24">
			<p class="font-mono text-[11px] tracking-[1.1px] text-muted">SMK NEGERI 8 SEMARANG · 2026</p>
		</div>
	</div>

	<!-- proyek -->
	<section id="proyek" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<div class="flex flex-col justify-between gap-4 sm:flex-row sm:items-center">
				<p class="font-mono text-[11px] tracking-[1.1px] text-ink">
					PROYEK ({filteredProjects.length})
				</p>

				<!-- FILTER KATEGORI PROYEK -->
				<div class="flex flex-wrap gap-2">
					{#each categories as category}
						<button
							type="button"
							onclick={() => (selectedCategory = category)}
							class="rounded-xl border px-3 py-1.5 font-mono text-[11px] tracking-[1.1px] transition-all {selectedCategory === category
								? 'border-primary bg-primary text-on-primary shadow-sm'
								: 'border-line bg-surface text-muted hover:text-ink'}"
						>
							{category}
						</button>
					{/each}
				</div>
			</div>

			<div class="mt-7 flex flex-col divide-y divide-line lg:mt-10">
				{#each filteredProjects as project, i (project.title)}
					<ProjectRow {project} index={i} />
				{/each}
			</div>
		</div>
	</section>

	<!-- tentang -->
	<section id="tentang" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px] text-ink">TENTANG</p>

			<div class="mt-7 flex flex-col items-start gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<!-- FOTO PROFIL -->
				<div class="aspect-[4/5] w-full lg:w-[420px] shrink-0 overflow-hidden rounded-2xl border border-line shadow-sm">
					<img
						src="/images/Dance Dancing GIF.gif"
						alt="Dance Dancing"
						class="h-full w-full object-cover"
					/>
				</div>

				<div class="flex w-full flex-col gap-7">
					<p class="max-w-[520px] text-[17px] leading-[1.6] text-muted lg:text-xl">
						{profile.bio}
					</p>

					<!-- KOTAK INFO TENTANG -->
					<div class="flex flex-col divide-y divide-line rounded-2xl border border-line bg-surface p-4 shadow-sm">
						{#each profile.facts as fact}
							<div class="flex items-center justify-between gap-8 py-3">
								<span class="font-mono text-[11px] tracking-[1.1px] text-muted">
									{fact.label}
								</span>
								<span class="text-[15px] text-ink text-right">{fact.value}</span>
							</div>
						{/each}
					</div>

					<div class="flex flex-col gap-3">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">KEAHLIAN</span>
						<div class="flex flex-wrap gap-2">
							{#each profile.skills as skill}
								<!-- BADGE KEAHLIAN -->
								<span
									class="border border-line bg-surface px-3.5 py-1.5 rounded-xl font-mono text-[11px] tracking-[1.1px] text-ink shadow-sm hover:border-primary transition-colors"
								>
									{skill}
								</span>
							{/each}
						</div>
					</div>
				</div>
			</div>
		</div>
	</section>

	<!-- ARCADE GAME CENTER (2 IKON PILIHAN GAME) -->
	<section id="minigame" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
				<div>
					<p class="font-mono text-[11px] tracking-[1.1px] text-ink">ARCADE GAME CENTER</p>
					<h2 class="text-2xl font-medium text-ink mt-1">Pilih Mini-Game Favoritmu 🎮</h2>
				</div>
				{#if activeGame !== 'none'}
					<button
						type="button"
						onclick={exitGameMenu}
						class="h-10 px-4 rounded-xl border border-line bg-surface text-ink text-xs font-mono hover:border-primary transition-all shadow-sm active:scale-95 self-start"
					>
						← Kembali ke Menu Game
					</button>
				{/if}
			</div>

			<div class="mt-7 rounded-2xl border border-line bg-surface p-8 shadow-sm">
				{#if activeGame === 'none'}
					<!-- MENU UTAMA: 2 IKON GAME -->
					<div class="grid grid-cols-1 sm:grid-cols-2 gap-6 py-6 max-w-2xl mx-auto">
						<!-- Tombol Ikon Game 1: Catch the Bug -->
						<button
							type="button"
							onclick={() => { activeGame = 'bug'; startBugGame(); }}
							class="group flex flex-col items-center gap-4 p-8 rounded-2xl border border-line bg-bg hover:border-primary transition-all shadow-sm hover:shadow-md active:scale-98 text-center cursor-pointer"
						>
							<div class="flex h-16 w-16 items-center justify-center rounded-2xl bg-primary/10 text-3xl group-hover:scale-110 transition-transform">
								🐛
							</div>
							<div>
								<h3 class="text-lg font-semibold text-ink group-hover:text-primary transition-colors">Catch the Bug!</h3>
								<p class="text-xs text-muted mt-1">Tangkap error code sebanyak-banyaknya dalam 15 detik.</p>
							</div>
						</button>

						<!-- Tombol Ikon Game 2: Memory Card -->
						<button
							type="button"
							onclick={() => { activeGame = 'memory'; startMemoryGame(); }}
							class="group flex flex-col items-center gap-4 p-8 rounded-2xl border border-line bg-bg hover:border-primary transition-all shadow-sm hover:shadow-md active:scale-98 text-center cursor-pointer"
						>
							<div class="flex h-16 w-16 items-center justify-center rounded-2xl bg-primary/10 text-3xl group-hover:scale-110 transition-transform">
								🃏
							</div>
							<div>
								<h3 class="text-lg font-semibold text-ink group-hover:text-primary transition-colors">Memory Card</h3>
								<p class="text-xs text-muted mt-1">Cocokkan 6 pasang kartu ikon teknologi dalam 30 detik.</p>
							</div>
						</button>
					</div>

				{:else if activeGame === 'bug'}
					<!-- GAME 1: CATCH THE BUG -->
					<div class="flex flex-col gap-6">
						<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
							<div>
								<h3 class="text-xl font-medium text-ink">Catch the Bug! 🐛</h3>
								<p class="text-sm text-muted mt-0.5">Bantu tangkap bug error sebelum waktu habis!</p>
							</div>
							<div class="flex flex-wrap items-center gap-3">
								<div class="font-mono text-xs text-ink bg-bg px-4 py-2 rounded-xl border border-line">
									Skor: <span class="text-primary font-bold">{bugScore}</span>
								</div>
								<div class="font-mono text-xs text-ink bg-bg px-4 py-2 rounded-xl border border-line">
									Waktu: <span class="text-danger font-bold">{bugTimeLeft}s</span>
								</div>
								{#if !bugGameStarted}
									<button
										type="button"
										onclick={startBugGame}
										class="h-10 px-5 rounded-xl bg-primary text-on-primary font-medium text-sm hover:opacity-95 transition-all shadow-sm active:scale-95"
									>
										{bugScore > 0 ? 'Main Lagi' : 'Mulai Main'}
									</button>
								{/if}
							</div>
						</div>

						<div class="relative h-[280px] w-full rounded-xl border border-line bg-bg overflow-hidden shadow-inner flex items-center justify-center">
							{#if !bugGameStarted}
								<div class="text-center flex flex-col items-center gap-2 p-4">
									<p class="text-sm text-muted">
										{bugScore > 0 ? `Permainan Selesai! Skor akhir kamu: ${bugScore} 🚀 Mantap!` : 'Tekan tombol "Mulai Main" di atas untuk mulai.'}
									</p>
								</div>
							{:else}
								<button
									type="button"
									onclick={catchBug}
									style="top: {bugPosition.top}%; left: {bugPosition.left}%;"
									class="absolute -translate-x-1/2 -translate-y-1/2 flex items-center justify-center h-12 w-12 rounded-2xl bg-primary text-on-primary shadow-lg animate-bounce transition-all hover:scale-110 active:scale-95 cursor-pointer"
									title="Klik Bug Ini!"
								>
									🐛
								</button>
							{/if}
						</div>
					</div>

				{:else if activeGame === 'memory'}
					<!-- GAME 2: MEMORY CARD (12 KARTU / 6 PASANG, 30 DETIK) -->
					<div class="flex flex-col gap-6">
						<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
							<div>
								<h3 class="text-xl font-medium text-ink">Memory Card Match 🃏</h3>
								<p class="text-sm text-muted mt-0.5">Temukan ke-6 pasangan kartu sebelum waktu 30 detik habis!</p>
							</div>
							<div class="flex flex-wrap items-center gap-3">
								<div class="font-mono text-xs text-ink bg-bg px-4 py-2 rounded-xl border border-line">
									Cocok: <span class="text-primary font-bold">{memoryScore} / {cardIcons.length}</span>
								</div>
								<div class="font-mono text-xs text-ink bg-bg px-4 py-2 rounded-xl border border-line">
									Waktu: <span class="text-danger font-bold">{memoryTimeLeft}s</span>
								</div>
								<button
									type="button"
									onclick={startMemoryGame}
									class="h-10 px-5 rounded-xl bg-primary text-on-primary font-medium text-sm hover:opacity-95 transition-all shadow-sm active:scale-95"
								>
									Reset Game
								</button>
							</div>
						</div>

						<div class="relative min-h-[280px] w-full rounded-xl border border-line bg-bg p-6 flex items-center justify-center">
							{#if memoryScore === cardIcons.length}
								<div class="text-center flex flex-col items-center gap-3 py-10">
									<p class="text-lg font-semibold text-primary">🎉 Hebat! Semua pasangan cocok dengan sisa waktu {memoryTimeLeft} detik!</p>
									<button
										type="button"
										onclick={startMemoryGame}
										class="h-10 px-6 rounded-xl bg-primary text-on-primary text-sm font-medium shadow-sm hover:opacity-95 transition-all"
									>
										Main Lagi
									</button>
								</div>
							{:else if memoryGameOver}
								<div class="text-center flex flex-col items-center gap-3 py-10">
									<p class="text-lg font-semibold text-danger">⏰ Waktu Habis! Skor pasanganmu: {memoryScore} / {cardIcons.length}</p>
									<button
										type="button"
										onclick={startMemoryGame}
										class="h-10 px-6 rounded-xl bg-primary text-on-primary text-sm font-medium shadow-sm hover:opacity-95 transition-all"
									>
										Coba Lagi
									</button>
								</div>
							{:else}
								<div class="grid grid-cols-3 sm:grid-cols-4 gap-3 sm:gap-4 w-full max-w-lg mx-auto">
									{#each memoryCards as card}
										<button
											type="button"
											onclick={() => flipCard(card)}
											class="aspect-square flex items-center justify-center rounded-xl border border-line text-2xl sm:text-3xl transition-all shadow-sm {card.isFlipped || card.isMatched ? 'bg-surface text-ink border-primary scale-105' : 'bg-bg text-transparent hover:border-primary cursor-pointer'}"
										>
											{#if card.isFlipped || card.isMatched}
												{card.icon}
											{:else}
												❓
											{/if}
										</button>
									{/each}
								</div>
							{/if}
						</div>
					</div>
				{/if}
			</div>
		</div>
	</section>

	<!-- TELL ME SOMETHING! -->
	<section id="guestbook" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px] text-ink">TELL ME SOMETHING!</p>

			<div class="mt-7 flex flex-col gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<div class="flex max-w-[420px] flex-col gap-5">
					<p class="text-[17px] leading-[1.6] text-muted lg:text-xl">
						Tinggalkan pesan, kesan, atau sapaan singkat setelah berkunjung ke portofolio saya!
					</p>

					<!-- Form Input -->
					<form
						onsubmit={addGuestMessage}
						class="flex w-full flex-col gap-4 p-6 rounded-2xl shadow-sm border border-line bg-surface"
					>
						<label class="flex flex-col gap-2">
							<span class="font-mono text-[11px] tracking-[1.1px] text-muted">NAMA / PANGGILAN</span>
							<input
								type="text"
								bind:value={guestName}
								required
								placeholder="Nama kamu"
								class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
							/>
						</label>

						<label class="flex flex-col gap-2">
							<span class="font-mono text-[11px] tracking-[1.1px] text-muted">PESAN / KESAN</span>
							<textarea
								bind:value={guestMessage}
								required
								rows="3"
								placeholder="Tulis pesanmu di sini..."
								class="min-h-[100px] w-full border border-line bg-bg px-3.5 py-3 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
							></textarea>
						</label>

						<button
							type="submit"
							class="flex h-12 items-center justify-center gap-2 px-6 text-[15px] font-medium rounded-xl bg-primary text-on-primary hover:opacity-95 transition-all shadow-sm active:scale-95"
						>
							Kirim Pesan
						</button>
					</form>
				</div>

				<!-- Daftar Pesan -->
				<div class="flex w-full flex-col gap-4 lg:flex-1">
					<span class="font-mono text-[11px] tracking-[1.1px] text-muted">PESAN TERBARU ({guestList.length})</span>
					{#if guestList.length === 0}
						<div class="flex flex-col items-center justify-center rounded-2xl border border-dashed border-line bg-surface/50 p-12 text-center">
							<p class="text-sm text-muted">Belum ada pesan yang dikirim. Jadilah yang pertama mengirim pesan!</p>
						</div>
					{:else}
						<div class="flex flex-col gap-3 max-h-[500px] overflow-y-auto pr-2">
							{#each guestList as item}
								<div class="flex flex-col gap-2 rounded-2xl border border-line bg-surface p-5 shadow-sm transition-all hover:border-primary">
									<div class="flex items-center justify-between">
										<span class="font-semibold text-ink text-[15px]">{item.name}</span>
										<span class="font-mono text-[10px] text-muted">{item.date}</span>
									</div>
									<p class="text-sm text-muted leading-relaxed">{item.message}</p>
								</div>
							{/each}
						</div>
					{/if}
				</div>
			</div>
		</div>
	</section>

	<!-- kontak -->
	<section id="kontak" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px] text-ink">KONTAK</p>

			<div class="mt-7 flex flex-col gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<div class="flex max-w-[420px] flex-col gap-5">
					<p class="text-[17px] leading-[1.6] text-muted lg:text-xl">
						Punya pertanyaan atau ingin bekerja sama? Kirim pesan langsung lewat formulir atau salin email saya.
					</p>

					<!-- KARTU SALIN EMAIL CEPAT -->
					<div class="flex items-center justify-between gap-3 rounded-2xl border border-line bg-surface p-4 shadow-sm">
						<span class="font-mono text-xs text-ink truncate">keishaputriratri@gmail.com</span>
						<button
							type="button"
							onclick={copyEmail}
							class="shrink-0 rounded-xl border border-line bg-bg px-3 py-1.5 font-mono text-[11px] tracking-[1.1px] text-ink hover:border-primary transition-all active:scale-95"
						>
							{copied ? 'TERSALIN ✓' : 'SALIN'}
						</button>
					</div>
				</div>

				<!-- FORM KONTAK -->
				<form
					method="POST"
					action="?/kirim"
					use:enhance={() => {
						status = 'loading';
						return async ({ result, update }) => {
							await update({ reset: result.type === 'success' });
							status = result.type === 'success' ? 'success' : 'error';
						};
					}}
					class="flex w-full flex-col gap-4 lg:w-[440px] lg:shrink-0 p-6 rounded-2xl shadow-sm border border-line bg-surface"
				>
					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">NAMA</span>
						<input
							name="nama"
							required
							autocomplete="name"
							placeholder="Nama lengkap kamu"
							class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">EMAIL</span>
						<input
							name="email"
							type="email"
							required
							autocomplete="email"
							placeholder="nama@email.com"
							class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">PESAN</span>
						<textarea
							name="pesan"
							required
							rows="5"
							placeholder="Tulis pesan kamu di sini"
							class="min-h-[120px] w-full border border-line bg-bg px-3.5 py-3 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
						></textarea>
					</label>

					<button
						type="submit"
						disabled={status === 'loading'}
						class="flex h-12 items-center justify-center gap-2 px-6 text-[15px] font-medium rounded-xl transition-all {status === 'loading' ? 'border border-line bg-surface text-muted' : 'bg-primary text-on-primary hover:opacity-95'}"
					>
						{#if status === 'loading'}
							<svg
								width="16"
								height="16"
								viewBox="0 0 24 24"
								fill="none"
								stroke="currentColor"
								stroke-width="1.5"
								stroke-linecap="round"
								class="animate-spin"
							>
								<path d="M12 3a9 9 0 1 0 9 9" />
							</svg>
							Mengirim...
						{:else}
							Kirim pesan
						{/if}
					</button>

					<!-- KOTAK STATUS -->
					{#if status !== 'idle'}
						<div class="flex items-center gap-3 border border-line bg-bg px-4 py-3.5 rounded-xl">
							{#if status === 'loading'}
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									class="animate-spin text-muted"
								>
									<path d="M12 3a9 9 0 1 0 9 9" />
								</svg>
								<span class="text-sm text-muted">Mengirim pesan...</span>
							{:else if status === 'success'}
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
									class="text-success"
								>
									<path d="M20 6 9 17l-5-5" />
								</svg>
								<span class="text-sm font-medium text-success">Pesan terkirim. Terima kasih!</span>
							{:else}
								<svg
									width="16"
									height="16"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="1.5"
									stroke-linecap="round"
									stroke-linejoin="round"
									class="text-danger"
								>
									<circle cx="12" cy="12" r="9" />
									<path d="M12 8v5M12 16.5v.01" />
								</svg>
								<span class="text-sm font-medium text-danger">Gagal mengirim. Coba lagi.</span>
							{/if}
						</div>
					{/if}
				</form>
			</div>
		</div>
	</section>

	<!-- footer -->
	<footer class="border-t border-line">
		<div class="mx-auto flex max-w-[1440px] flex-col gap-10 px-6 py-14 lg:px-24">
			<p
				class="max-w-[640px] text-[34px] leading-[1.05] font-medium tracking-[-0.6px] lg:text-[48px] lg:tracking-[-1.2px] text-ink"
			>
				Mari membangun sesuatu.
			</p>

			<div
				class="flex flex-col items-start gap-4 lg:flex-row lg:items-center lg:justify-between lg:gap-6"
			>
				<a href="mailto:keishaputriratri@gmail.com" class="font-mono text-xs tracking-[1.1px] text-primary">
					keishaputriratri@gmail.com
				</a>
				<div class="flex items-center gap-6">
					<a href="#proyek" class="font-mono text-[11px] tracking-[1.1px] text-muted hover:text-ink transition-colors">PROYEK</a>
					<a
						href="https://github.com/kekukies"
						target="_blank"
						rel="noopener noreferrer"
						class="font-mono text-[11px] tracking-[1.1px] text-muted hover:text-ink transition-colors">GITHUB</a
					>
					<a href="mailto:keishaputriratri@gmail.com" class="font-mono text-[11px] tracking-[1.1px] text-muted hover:text-ink transition-colors">
						EMAIL
					</a>
				</div>
			</div>
		</div>
	</footer>

	<!-- TOMBOL KEMBALI KE ATAS -->
	{#if showScrollTop}
		<button
			type="button"
			aria-label="Kembali ke atas"
			onclick={scrollToTop}
			class="fixed right-6 bottom-6 z-40 flex h-11 w-11 items-center justify-center rounded-xl border border-line bg-surface text-ink shadow-md hover:border-primary transition-all active:scale-95"
		>
			<svg
				width="18"
				height="18"
				viewBox="0 0 24 24"
				fill="none"
				stroke="currentColor"
				stroke-width="1.5"
				stroke-linecap="round"
				stroke-linejoin="round"
			>
				<path d="m18 15-6-6-6 6" />
			</svg>
		</button>
	{/if}
</div>

<script module lang="ts">
	export const profile = {
		bio: "Siswa SMK Negeri 8 Semarang jurusan Rekayasa Perangkat Lunak. Berfokus pada pengembangan antarmuka web yang interaktif, cepat, dan rapi.",
		facts: [
			{ label: "STATUS", value: "Siswa PPLG · SMK Negeri 8 Semarang" },
			{ label: "FOKUS", value: "Web Development & UI/UX" },
			{ label: "LOKASI", value: "Semarang, Jawa Tengah" },
			{ label: "KONTAK", value: "keishaputriratri@gmail.com" }
		],
		skills: ["SvelteKit", "TypeScript", "Tailwind CSS", "JavaScript", "HTML/CSS", "Git", "Construct 3"]
	};

	export const projects = [
		{
			title: "E-Canteen",
			category: "WEB APPLICATION",
			stack: "SvelteKit · Tailwind",
			year: "2026",
			description: "Platform pemesanan makanan kantin sekolah berbasis web yang mempercepat transaksi dan meminimalisir antrean.",
			tech: ["SvelteKit", "Tailwind CSS", "TypeScript"],
			href: "https://github.com/kekukies",
			image: "/images/Cat Working GIF.gif"
		},
		{
			title: "JagaCuan",
			category: "MOBILE APP",
			stack: "JavaScript · Tailwind",
			year: "2026",
			description: "Aplikasi pencatatan tabungan dan keuangan harian yang dirancang untuk membantu mengelola anggaran secara praktis.",
			tech: ["JavaScript", "Tailwind CSS"],
			href: "https://github.com/kekukies",
			image: "/images/Dance Dancing GIF.gif"
		},
		{
			title: "Mini Mo Adventure!",
			category: "GAME DEVELOPMENT",
			stack: "Construct 3",
			year: "2026",
			description: "Game platformer petualangan seru dengan rintangan menantang yang dibangun menggunakan Construct 3.",
			tech: ["Construct 3", "Game Design"],
			href: "https://github.com/kekukies",
			image: "/images/Question Mark Cat GIF.gif"
		},
		{
			title: "Knight's Vow: Rise of The Kingdom",
			category: "GAME DEVELOPMENT",
			stack: "Construct 3 · Pixel Art",
			year: "2026",
			description: "Game aksi petualangan platformer over-world bertema kesatria dengan eksplorasi peta luas dan pertempuran epik.",
			tech: ["Construct 3", "Game Design", "Pixel Art"],
			href: "https://github.com/kekukies",
			image: "/images/Cat Working GIF.gif"
		}
	];
</script>