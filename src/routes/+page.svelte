<script lang="ts">
	import { enhance } from '$app/forms';
	import { onMount, tick } from 'svelte';
	import ProjectRow from '$lib/components/ProjectRow.svelte';

	type Theme = 'light' | 'dark';
	type Status = 'idle' | 'loading' | 'success' | 'error';
	type Language = 'id' | 'en';

	let theme: Theme = $state('light');
	let lang: Language = $state('id');
	let status: Status = $state('idle');
	let selectedCategory: string = $state('SEMUA');
	let copied: boolean = $state(false);
	let mobileMenuOpen: boolean = $state(false);
	let showScrollTop: boolean = $state(false);

	// Kamus Bahasa (Bilingual Dictionary)
	const dict = {
		id: {
			navProjects: "Proyek",
			navAbout: "Tentang",
			navCert: "Sertifikat",
			navGame: "Mini Game",
			navGuestbook: "Guestbook",
			navTerminal: "Terminal",
			navContact: "Kontak",
			heroTitle: "Keisha Putri Ratri",
			heroDesc: "Siswa SMK jurusan PPLG. Membangun antarmuka web yang cepat, rapi, dan mudah dipakai.",
			btnProjects: "Lihat proyek",
			btnContact: "Hubungi saya",
			availableStatus: "TERSEDIA UNTUK PROYEK BARU — 2026",
			projectsTitle: "PROYEK",
			aboutTitle: "TENTANG",
			certTitle: "SERTIFIKAT & PENCAPAIAN",
			certSub: "Penghargaan & Kompetensi 🏆",
			certDesc: "Dokumentasi prestasi akademik & kejuruan",
			gameTitle: "MINI GAME",
			gameSub: "Pilih Mini-Game 🎮",
			gameBack: "← Kembali ke Menu Game",
			guestTitle: "TELL ME SOMETHING!",
			guestDesc: "Tinggalkan pesan, kesan, atau sapaan singkat setelah berkunjung ke portofolio saya!",
			guestName: "NAMA / PANGGILAN",
			guestMsg: "PESAN / KESAN",
			guestSend: "Kirim Pesan",
			guestListTitle: "PESAN TERBARU",
			terminalTitle: "INTERACTIVE CLI",
			terminalSub: "KeiOS Terminal 💻",
			contactTitle: "KONTAK",
			contactDesc: "Punya pertanyaan atau ingin bekerja sama? Kirim pesan langsung lewat formulir atau hubungi lewat media sosial saya.",
			nameLabel: "NAMA",
			emailLabel: "EMAIL",
			messageLabel: "PESAN",
			sendButton: "Kirim pesan",
			footerText: "Mari membangun sesuatu."
		},
		en: {
			navProjects: "Projects",
			navAbout: "About",
			navCert: "Certificates",
			navGame: "Mini Game",
			navGuestbook: "Guestbook",
			navTerminal: "Terminal",
			navContact: "Contact",
			heroTitle: "Keisha Putri Ratri",
			heroDesc: "Vocational high school student majoring in Software Engineering. Building fast, clean, and user-friendly web interfaces.",
			btnProjects: "View projects",
			btnContact: "Contact me",
			availableStatus: "AVAILABLE FOR NEW PROJECTS — 2026",
			projectsTitle: "PROJECTS",
			aboutTitle: "ABOUT",
			certTitle: "CERTIFICATES & ACHIEVEMENTS",
			certSub: "Awards & Competencies 🏆",
			certDesc: "Documentation of academic & vocational achievements",
			gameTitle: "MINI GAME",
			gameSub: "Choose a Mini-Game 🎮",
			gameBack: "← Back to Game Menu",
			guestTitle: "TELL ME SOMETHING!",
			guestDesc: "Leave a message, impression, or a short greeting after visiting my portfolio!",
			guestName: "NAME / NICKNAME",
			guestMsg: "MESSAGE / IMPRESSION",
			guestSend: "Send Message",
			guestListTitle: "RECENT MESSAGES",
			terminalTitle: "INTERACTIVE CLI",
			terminalSub: "KeiOS Terminal 💻",
			contactTitle: "CONTACT",
			contactDesc: "Have questions or want to collaborate? Send a message directly via the form or reach out through my social media.",
			nameLabel: "NAME",
			emailLabel: "EMAIL",
			messageLabel: "MESSAGE",
			sendButton: "Send message",
			footerText: "Let's build something together."
		}
	};

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

	// State untuk Interactive CLI / Terminal
	type TerminalLine = {
		type: 'command' | 'output';
		text: string;
	};
	let cliInput = $state('');
	let cliContainer: HTMLDivElement | undefined = $state();
	let cliHistory = $state<TerminalLine[]>([
		{ type: 'output', text: 'Selamat datang di KeiOS v2.6.0 Terminal!' },
		{ type: 'output', text: 'Ketik "help" untuk melihat daftar perintah yang tersedia.' }
	]);

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

		// Otomatis coba putar audio di awal (di-catch jika diblokir browser sebelum interaksi)
		if (audioEl) {
			audioEl.play().catch(() => {
				isPlaying = false;
			});
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

	function toggleLanguage() {
		lang = lang === 'id' ? 'en' : 'id';
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

	// Fungsi Terminal CLI
	async function handleCliSubmit(e: Event) {
		e.preventDefault();
		const rawCmd = cliInput.trim();
		if (!rawCmd) return;

		const cmd = rawCmd.toLowerCase();
		cliHistory = [...cliHistory, { type: 'command', text: `guest@kei-portfolio:~$ ${rawCmd}` }];
		cliInput = '';

		if (cmd === 'clear') {
			cliHistory = [];
			return;
		}

		let responseLines: string[] = [];

		switch (cmd) {
			case 'help':
				responseLines = [
					'Daftar perintah yang tersedia:',
					'  about      - Informasi singkat tentang Keisha',
					'  skills     - Daftar keahlian dan teknologi yang dikuasai',
					'  projects   - Daftar proyek unggulan yang pernah dibuat',
					'  cert       - Daftar sertifikat & pencapaian',
					'  contact    - Email dan tautan media sosial',
					'  clear      - Bersihkan layar terminal'
				];
				break;
			case 'about':
				responseLines = [
					'Nama   : Keisha Putri Ratri',
					'Status : Siswa SMK Negeri 8 Semarang (Jurusan PPLG)',
					'Fokus  : Web Development & Game Development',
					'Bio    : Membangun antarmuka web yang interaktif, cepat, dan rapi.'
				];
				break;
			case 'skills':
				responseLines = [
					'Tech Stack & Tools:',
					`  ${profile.skills.join(' · ')}`
				];
				break;
			case 'projects':
				responseLines = [
					'Proyek Unggulan (2026):',
					...projects.map((p, idx) => `  [${idx + 1}] ${p.title} (${p.category})`)
				];
				break;
			case 'cert':
				responseLines = [
					'Sertifikat & Pencapaian:',
					...certificates.map((c, idx) => `  [${idx + 1}] ${c.title} (${c.issuer} - ${c.year})`)
				];
				break;
			case 'contact':
				responseLines = [
					'Email     : keishaputriratri@gmail.com',
					'GitHub    : github.com/kekukies',
					'Instagram : @kekukies'
				];
				break;
			default:
				responseLines = [
					`Perintah tidak dikenali: "${rawCmd}". Ketik "help" untuk bantuan.`
				];
		}

		cliHistory = [
			...cliHistory,
			...responseLines.map((line) => ({ type: 'output' as const, text: line }))
		];

		await tick();
		if (cliContainer) {
			cliContainer.scrollTop = cliContainer.scrollHeight;
		}
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
				<nav class="hidden items-center gap-6 lg:flex">
					<a href="#proyek" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navProjects}</a>
					<a href="#tentang" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navAbout}</a>
					<a href="#sertifikat" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navCert}</a>
					<a href="#minigame" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navGame}</a>
					<a href="#guestbook" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navGuestbook}</a>
					<a href="#terminal" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navTerminal}</a>
					<a href="#kontak" class="text-sm text-muted hover:text-ink transition-colors">{dict[lang].navContact}</a>
				</nav>

				<!-- TOMBOL TOGGLE BAHASA (BILINGUAL) -->
				<button
					type="button"
					aria-label="Ganti Bahasa"
					onclick={toggleLanguage}
					class="flex h-10 px-3 items-center justify-center rounded-xl border border-line bg-surface font-mono text-xs tracking-wider text-ink hover:border-primary transition-all active:scale-95 shadow-sm"
				>
					{lang === 'id' ? 'EN 🇬🇧' : 'ID 🇮🇩'}
				</button>

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
					{dict[lang].navProjects}
				</a>
				<a
					href="#tentang"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					{dict[lang].navAbout}
				</a>
				<a
					href="#sertifikat"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					{dict[lang].navCert}
				</a>
				<a
					href="#minigame"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					{dict[lang].navGame}
				</a>
				<a
					href="#guestbook"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					{dict[lang].navGuestbook}
				</a>
				<a
					href="#terminal"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					{dict[lang].navTerminal}
				</a>
				<a
					href="#kontak"
					onclick={() => (mobileMenuOpen = false)}
					class="py-1 text-sm text-muted hover:text-ink transition-colors"
				>
					{dict[lang].navContact}
				</a>
			</nav>
		{/if}
	</header>

	<!-- hero -->
	<section class="mx-auto max-w-[1440px] px-6 pt-16 pb-14 lg:px-24 lg:pt-[120px]">
		<h1
			class="text-[52px] leading-[0.92] font-semibold tracking-[-2px] lg:text-[130px] lg:tracking-[-6px] text-ink"
		>
			{dict[lang].heroTitle}
		</h1>

		<div class="mt-7 flex flex-col gap-6">
			<p class="max-w-[560px] text-[17px] leading-[1.6] text-muted lg:text-xl">
				{dict[lang].heroDesc}
			</p>

			<div class="flex flex-wrap items-center gap-3">
				<a
					href="#proyek"
					class="flex h-12 items-center rounded-xl bg-primary px-6 text-[15px] font-medium text-on-primary shadow-sm hover:opacity-95 transition-all"
				>
					{dict[lang].btnProjects}
				</a>
				<a
					href="#kontak"
					class="flex h-12 items-center rounded-xl border border-line px-6 text-[15px] text-ink hover:border-primary transition-all"
				>
					{dict[lang].btnContact}
				</a>
			</div>

			<div class="flex flex-col justify-between gap-4 pt-2 sm:flex-row sm:items-center">
				<div class="flex items-center gap-2">
					<span class="h-2 w-2 rounded-full bg-success animate-pulse"></span>
					<p class="font-mono text-[11px] tracking-[1.1px] text-muted">
						{dict[lang].availableStatus}
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
			src="/images/computer desk.gif"
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
					{dict[lang].projectsTitle} ({filteredProjects.length})
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
			<p class="font-mono text-[11px] tracking-[1.1px] text-ink">{dict[lang].aboutTitle}</p>

			<div class="mt-7 flex flex-col items-start gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<!-- FOTO PROFIL -->
				<div class="aspect-[4/5] w-full lg:w-[420px] shrink-0 overflow-hidden rounded-2xl border border-line shadow-sm">
					<img
						src="/images/gambar keisha cantik.jpeg"
						alt="Pretty Kei"
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

	<!-- SERTIFIKAT & PENCAPAIAN -->
	<section id="sertifikat" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
				<div>
					<p class="font-mono text-[11px] tracking-[1.1px] text-ink">{dict[lang].certTitle}</p>
					<h2 class="text-2xl font-medium text-ink mt-1">{dict[lang].certSub}</h2>
				</div>
				<p class="font-mono text-xs text-muted">{dict[lang].certDesc}</p>
			</div>

			<div class="mt-7 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
				{#each certificates as cert}
					<div class="flex flex-col justify-between rounded-2xl border border-line bg-surface p-6 shadow-sm hover:border-primary transition-all group">
						<div class="flex flex-col gap-4">
							<div class="flex items-center justify-between">
								<span class="inline-flex items-center justify-center h-10 w-10 rounded-xl bg-primary/10 text-primary text-lg group-hover:scale-110 transition-transform">
									{cert.icon}
								</span>
								<span class="font-mono text-[11px] text-primary bg-bg px-3 py-1 rounded-lg border border-line">
									{cert.year}
								</span>
							</div>
							<div>
								<h3 class="text-base font-semibold text-ink group-hover:text-primary transition-colors">{cert.title}</h3>
								<p class="text-xs font-mono text-muted mt-1">{cert.issuer}</p>
							</div>
							<p class="text-sm text-muted leading-relaxed">{cert.description}</p>
						</div>
						<div class="mt-6 pt-4 border-t border-line flex items-center justify-between">
							<span class="font-mono text-[10px] tracking-[1.1px] text-muted uppercase">Verified Credentials</span>
							<span class="font-mono text-xs text-primary font-medium">Aktif ✓</span>
						</div>
					</div>
				{/each}
			</div>
		</div>
	</section>

	<!-- ARCADE GAME CENTER (2 IKON PILIHAN GAME) -->
	<section id="minigame" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
				<div>
					<p class="font-mono text-[11px] tracking-[1.1px] text-ink">{dict[lang].gameTitle}</p>
					<h2 class="text-2xl font-medium text-ink mt-1">{dict[lang].gameSub}</h2>
				</div>
				{#if activeGame !== 'none'}
					<button
						type="button"
						onclick={exitGameMenu}
						class="h-10 px-4 rounded-xl border border-line bg-surface text-ink text-xs font-mono hover:border-primary transition-all shadow-sm active:scale-95 self-start"
					>
						{dict[lang].gameBack}
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
								<p class="text-xs text-muted mt-1">Tangkap bug sebanyak-banyaknya dalam waktu 15 detik.</p>
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
								<p class="text-xs text-muted mt-1">Cocokkan 6 pasang kartu dalam 30 detik.</p>
							</div>
						</button>
					</div>

				{:else if activeGame === 'bug'}
					<!-- GAME 1: CATCH THE BUG -->
					<div class="flex flex-col gap-6">
						<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
							<div>
								<h3 class="text-xl font-medium text-ink">Catch the Bug! 🐛</h3>
								<p class="text-sm text-muted mt-0.5">Bantu tangkap bug sebelum waktu habis!</p>
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
										{bugScore > 0 ? `Permainan Selesai! Skor akhir kamu: ${bugScore} Keren!` : 'Tekan tombol "Mulai Main" di atas untuk mulai.'}
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
									<p class="text-lg font-semibold text-primary">🎉 Keren! Semua pasangan klop, waktunya masih nyisa {memoryTimeLeft} detik!</p>
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
			<p class="font-mono text-[11px] tracking-[1.1px] text-ink">{dict[lang].guestTitle}</p>

			<div class="mt-7 flex flex-col gap-8 lg:mt-10 lg:flex-row lg:gap-24">
				<div class="flex max-w-[420px] flex-col gap-5">
					<p class="text-[17px] leading-[1.6] text-muted lg:text-xl">
						{dict[lang].guestDesc}
					</p>

					<!-- Form Input -->
					<form
						onsubmit={addGuestMessage}
						class="flex w-full flex-col gap-4 p-6 rounded-2xl shadow-sm border border-line bg-surface"
					>
						<label class="flex flex-col gap-2">
							<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{dict[lang].guestName}</span>
							<input
								type="text"
								bind:value={guestName}
								required
								placeholder="Nama kamu"
								class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
							/>
						</label>

						<label class="flex flex-col gap-2">
							<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{dict[lang].guestMsg}</span>
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
							{dict[lang].guestSend}
						</button>
					</form>
				</div>

				<!-- Daftar Pesan -->
				<div class="flex w-full flex-col gap-4 lg:flex-1">
					<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{dict[lang].guestListTitle} ({guestList.length})</span>
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

	<!-- INTERACTIVE TERMINAL (CLI) -->
	<section id="terminal" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-2">
				<div>
					<p class="font-mono text-[11px] tracking-[1.1px] text-ink">{dict[lang].terminalTitle}</p>
					<h2 class="text-2xl font-medium text-ink mt-1">{dict[lang].terminalSub}</h2>
				</div>
				<p class="font-mono text-xs text-muted">Coba ketik: <span class="text-primary font-semibold">help</span>, <span class="text-primary font-semibold">cert</span>, atau <span class="text-primary font-semibold">projects</span></p>
			</div>

			<div class="mt-7 overflow-hidden rounded-2xl border border-line bg-[#1c1917] text-[#f5f5f4] shadow-md font-mono text-xs sm:text-sm">
				<!-- Header Bar Ala MacOS / Linux Terminal -->
				<div class="flex items-center justify-between border-b border-[#292524] bg-[#292524] px-4 py-3">
					<div class="flex items-center gap-2">
						<span class="h-3 w-3 rounded-full bg-[#ef4444]"></span>
						<span class="h-3 w-3 rounded-full bg-[#eab308]"></span>
						<span class="h-3 w-3 rounded-full bg-[#22c55e]"></span>
					</div>
					<span class="text-[11px] text-[#a8a29e]">guest@kei-portfolio: ~</span>
					<div class="w-10"></div>
				</div>

				<!-- Layar Output Terminal -->
				<div
					bind:this={cliContainer}
					class="flex h-[260px] flex-col gap-1.5 overflow-y-auto p-5 leading-relaxed"
				>
					{#each cliHistory as line}
						<div class="{line.type === 'command' ? 'text-[#d4a373] font-semibold mt-1' : 'text-[#e7e5e4] whitespace-pre-wrap'}">
							{line.text}
						</div>
					{/each}
				</div>

				<!-- Baris Input Perintah -->
				<form
					onsubmit={handleCliSubmit}
					class="flex items-center gap-2 border-t border-[#292524] bg-[#1c1917] px-5 py-3.5"
				>
					<span class="text-[#d4a373] font-bold shrink-0">guest@kei:~$</span>
					<input
						type="text"
						bind:value={cliInput}
						placeholder="Ketik perintah di sini..."
						autocomplete="off"
						spellcheck="false"
						class="w-full bg-transparent text-[#f5f5f4] outline-none placeholder:text-[#57534e]"
					/>
					<button
						type="submit"
						class="rounded-lg bg-[#292524] px-3 py-1 text-xs text-[#d4a373] hover:bg-[#44403c] transition-colors shrink-0"
					>
						Enter ↵
					</button>
				</form>
			</div>
		</div>
	</section>

	<!-- kontak -->
	<section id="kontak" class="border-t border-line">
		<div class="mx-auto max-w-[1440px] px-6 pt-5 pb-14 lg:px-24 lg:pb-24">
			<p class="font-mono text-[11px] tracking-[1.1px] text-ink">{dict[lang].contactTitle}</p>

			<div class="mt-7 grid grid-cols-1 lg:grid-cols-12 gap-8 lg:gap-16 items-start lg:mt-10">
				<!-- Sisi Kiri (Teks & Tombol Salin Email / Tautan Sosial Media) - Mengambil 5 Kolom -->
				<div class="flex flex-col gap-5 lg:col-span-5">
					<p class="text-[17px] leading-[1.6] text-muted lg:text-xl">
						{dict[lang].contactDesc}
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

					<!-- KARTU INSTAGRAM & SOSIAL MEDIA -->
					<div class="flex items-center gap-3">
						<a
							href="https://instagram.com/kekukies"
							target="_blank"
							rel="noopener noreferrer"
							class="flex items-center gap-3 flex-1 rounded-2xl border border-line bg-surface p-4 shadow-sm hover:border-primary transition-all group"
						>
							<div class="flex h-10 w-10 items-center justify-center rounded-xl bg-primary/10 text-primary group-hover:scale-110 transition-transform">
								<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round">
									<rect width="20" height="20" x="2" y="2" rx="5" ry="5"/>
									<path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z"/>
									<line x1="17.5" x2="17.51" y1="6.5" y2="6.5"/>
								</svg>
							</div>
							<div class="flex flex-col">
								<span class="font-mono text-[10px] tracking-[1.1px] text-muted uppercase">Instagram</span>
								<span class="text-xs font-medium text-ink">@kekukies</span>
							</div>
						</a>
					</div>
				</div>

				<!-- Sisi Kanan (Form Kontak) - Mengambil 7 Kolom -->
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
					class="flex w-full flex-col gap-4 lg:col-span-7 p-6 md:p-8 rounded-2xl shadow-sm border border-line bg-surface"
				>
					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{dict[lang].nameLabel}</span>
						<input
							name="nama"
							required
							autocomplete="name"
							placeholder="Nama lengkap kamu"
							class="h-12 w-full border border-line bg-bg px-3.5 text-[15px] text-ink outline-none rounded-xl transition-all focus:border-primary"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{dict[lang].emailLabel}</span>
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
						<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{dict[lang].messageLabel}</span>
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
							{dict[lang].sendButton}
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
				{dict[lang].footerText}
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
					<a
						href="https://instagram.com/kekukies"
						target="_blank"
						rel="noopener noreferrer"
						class="font-mono text-[11px] tracking-[1.1px] text-muted hover:text-ink transition-colors">INSTAGRAM</a
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
			{ label: "FOKUS", value: "Web Development · Game Development" },
			{ label: "LOKASI", value: "Semarang, Jawa Tengah" },
			{ label: "KONTAK", value: "keishaputriratri@gmail.com" }
		],
		skills: ["SvelteKit", "TypeScript", "Tailwind CSS", "JavaScript", "HTML/CSS", "Git", "Construct 3", "MySQL", "PHP"]
	};

	export const certificates = [
		{
			title: "Sertifikat Uji Sertifikasi Bahasa Inggris",
			issuer: "Pelajar SMP Kota Semarang",
			year: "2024",
			description: "Sertifikasi resmi tingkat kota yang memvalidasi kemampuan komunikasi, tata bahasa, dan pemahaman bahasa Inggris.",
			icon: "🎓"
		},
		{
			title: "Sertifikasi TOEIC (Test of English)",
			issuer: "Standar Internasional",
			year: "2025",
			description: "Sertifikasi kemahiran bahasa Inggris bertaraf internasional untuk mengukur kesiapan komunikasi di lingkungan profesional.",
			icon: "⭐"
		},
		{
			title: "Pengurus Snapan Developer Community",
			issuer: "SMK Negeri 8 Semarang",
			year: "2026",
			description: "Memegang penuh pengelolaan media sosial Snapan Developer untuk membangun interaksi dan publikasi kegiatan komunitas.",
			icon: "💻"
		}
	];

	export const projects = [
		{
			title: "E-Canteen",
			category: "WEB APPLICATION",
			stack: "-",
			year: "2026",
			description: "Platform pemesanan makanan kantin sekolah berbasis web yang mempercepat transaksi dan meminimalisir antrean.",
			tech: ["-"],
			href: "https://github.com/kekukies",
			image: "/images/gambar e canteen.jpeg"
		},
		{
			title: "JagaCuan",
			category: "MOBILE APP",
			stack: "-",
			year: "2026",
			description: "Aplikasi pencatatan tabungan dan keuangan harian yang dirancang untuk membantu mengelola anggaran secara praktis.",
			tech: ["-"],
			href: "https://github.com/kekukies",
			image: "/images/gambar jaga cuan.jpeg"
		},
		{
			title: "Mini Mo Adventure!",
			category: "GAME DEVELOPMENT",
			stack: "Construct 3",
			year: "2026",
			description: "Game platformer petualangan seru dengan rintangan menantang yang dibangun menggunakan Construct 3.",
			tech: ["Construct 3", "Game Design"],
			href: "https://github.com/kekukies",
			image: "/images/gambar mini mok.jpeg"
		},
		{
			title: "Knight's Vow: Rise of The Kingdom",
			category: "GAME DEVELOPMENT",
			stack: "Unity · Pixel Art",
			year: "2026",
			description: "Game aksi petualangan platformer over-world bertema kesatria dengan eksplorasi peta luas dan pertempuran epik.",
			tech: ["Unity", "Game Design", "Pixel Art"],
			href: "https://github.com/kekukies",
			image: "/images/Working Chis Sweet Home GIF.gif"
		}
	];
</script>