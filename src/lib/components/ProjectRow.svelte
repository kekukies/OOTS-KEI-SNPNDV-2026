<script lang="ts">
	import type { Project } from '$lib/data';

	type Props = {
		project: Project & { image?: string; category?: string };
		index: number;
	};

	let { project, index }: Props = $props();

	// State untuk mendeteksi hover
	let isHovered = $state(false);
</script>

<div
	role="region"
	aria-label={project.title}
	onmouseenter={() => (isHovered = true)}
	onmouseleave={() => (isHovered = false)}
	class="group relative flex flex-col gap-2.5 py-7 border-b border-line transition-colors hover:bg-surface/50 px-4 rounded-xl"
>
	<div class="flex items-center justify-between">
		<span class="font-mono text-[11px] tracking-[1.1px] text-muted">
			{String(index + 1).padStart(2, '0')}
		</span>

		<div class="flex items-center gap-5">
			<span class="font-mono text-[11px] tracking-[1.1px] uppercase text-muted">{project.stack}</span>
			<span class="font-mono text-[11px] tracking-[1.1px] text-muted">{project.year}</span>

			{#if project.href}
				<a href={project.href} aria-label="Buka {project.title}" class="text-primary transition-transform group-hover:scale-110">
					<svg
						width="16"
						height="16"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="1.5"
						stroke-linecap="round"
						stroke-linejoin="round"
					>
						<path d="M7 17 17 7M9 7h8v8" />
					</svg>
				</a>
			{/if}
		</div>
	</div>

	<div class="flex flex-col md:flex-row md:items-center md:justify-between gap-4">
		<div class="flex flex-col gap-1 max-w-[650px]">
			<h2 class="text-[28px] leading-[1.15] font-medium tracking-[-0.5px] group-hover:text-primary transition-colors">
				{project.title}
			</h2>
			<p class="text-[15px] leading-[1.55] text-muted">{project.description}</p>
		</div>

		<!-- Kotak Preview Gambar Statis di Samping (Muncul Saat Di-hover) -->
		{#if isHovered && project.image}
			<div
				class="hidden md:flex flex-col w-56 overflow-hidden rounded-xl border border-line bg-surface shadow-lg animate-fade-in shrink-0"
			>
				<div class="aspect-video w-full overflow-hidden bg-bg">
					<img src={project.image} alt={project.title} class="h-full w-full object-cover" />
				</div>
				<div class="p-2.5">
					{#if project.category}
						<span class="font-mono text-[9px] text-primary uppercase">{project.category}</span>
					{/if}
					<p class="text-xs font-semibold text-ink truncate">{project.title}</p>
				</div>
			</div>
		{/if}
	</div>
</div>