<script lang="ts">
	import { isBefore, differenceInMilliseconds, intervalToDuration } from "date-fns";

	// IST is UTC+05:30. Convert fixed IST schedule to UTC to avoid local-time pitfalls.
	const start = new Date(Date.UTC(2025, 8, 20, 2, 30, 0)); // 20 Sep 2025, 08:00 IST
	const end = new Date(Date.UTC(2025, 8, 20, 11, 30, 0)); // 20 Sep 2025, 17:00 IST

	// Temporary: start at 11:30 PM on Sep 19, 2025 (local time)
	// const start = new Date(2025, 8, 19, 23, 30, 0);
	// const end = new Date(start.getTime() + 4 * 60 * 60 * 1000);

	let now = $state(Date.now());
	let focusMode = $state(false);

	$effect(() => {
		const id = setInterval(() => (now = Date.now()), 1000);
		return () => clearInterval(id);
	});

	const phase = $derived(isBefore(now, start) ? "pre" : isBefore(now, end) ? "live" : "post");
	const targetDate = $derived(phase === "pre" ? start : phase === "live" ? end : now);

	const breakdown = $derived.by(() => {
		const d = intervalToDuration({ start: now, end: targetDate });
		return {
			hours: d.hours ?? 0,
			minutes: d.minutes ?? 0,
			seconds: d.seconds ?? 0
		};
	});

	const progress = $derived.by(() => {
		if (phase !== "live") return phase === "pre" ? 0 : 100;
		const elapsed = differenceInMilliseconds(now, start);
		const total = differenceInMilliseconds(end, start);
		return Math.min(100, Math.max(0, (elapsed / total) * 100));
	});

	const label = $derived(
		phase === "pre" ? "Starts in" : phase === "live" ? "Ends in" : "Event ended"
	);

	function pad(n: string | number) {
		return String(n).padStart(2, "0");
	}
</script>

<main
	data-focused={focusMode}
	class="group relative flex min-h-screen items-center justify-center bg-transparent transition-colors data-focused:bg-neutral-950 p-6 text-neutral-50"
>
	<button
		aria-pressed={focusMode}
		onclick={() => (focusMode = !focusMode)}
		class="fixed top-4 right-4 z-50 rounded-md bg-neutral-900/80 px-3 py-1.5 text-xs ring-1 ring-white/10 hover:bg-neutral-900/90"
	>
		{focusMode ? "Exit Focus" : "Focus"}
	</button>

	<div class="w-full max-w-4xl">
		<div class="group-data-focused:fade-out">
			<h1
				class="text-center font-mono uppercase tracking-widest text-3xl font-semibold sm:text-4xl xl:text-5xl"
			>
				CodeDrop
			</h1>
			<p class="mt-2 text-center text-sm text-neutral-400 xl:text-base">
				20 Sep 2025 • 08:00-17:00 IST (UTC+5:30)
			</p>
		</div>

		<section class="mt-10 xl:mt-14">
			<p
				class="text-center text-xs tracking-widest text-neutral-400 uppercase group-data-focused:fade-out xl:text-sm"
			>
				{label}
			</p>

			<div
				aria-live="polite"
				class="mt-4 grid grid-cols-3 gap-3 transition-transform group-data-focused:slide-up-delayed sm:gap-4"
			>
				{@render block(breakdown.hours, "Hours")}
				{@render block(breakdown.minutes, "Minutes")}
				{@render block(breakdown.seconds, "Seconds")}
			</div>

			{#if phase === "live"}
				<div class="mt-8 group-data-focused:fade-out">
					<div class="h-2 overflow-hidden rounded-full bg-neutral-900/70 ring-1 ring-white/10">
						<div
							class="h-full bg-[#BC0CE2] transition-[width] duration-700"
							style={`width: ${progress}%`}
						></div>
					</div>
					<div class="mt-2 flex justify-between text-[10px] text-neutral-500 sm:text-xs xl:text-sm">
						<span>08:00 IST</span>
						<span>17:00 IST</span>
					</div>
				</div>
			{/if}
		</section>
	</div>

	<!-- <div aria-hidden="true" class="pointer-events-none absolute inset-0 overflow-hidden">
		<div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2">
			<div
				class="h-[75vmin] w-[75vmin] rounded-full blur-3xl group-data-focused:fade-out"
				style="background: radial-gradient(closest-side, rgba(115, 7, 148, 0.2), rgba(115, 7, 148, 0) 70%)"
			></div>
		</div>
	</div> -->


</main>

	<img
		src="/bg.png"
		alt=""
		class="pointer-events-none absolute inset-0 h-full w-full object-cover opacity-100 transition-opacity group-data-focused:opacity-0 -bg-conic-90 -z-10"
	/>

{#snippet block(num: number, label: string)}
	<div
		class="space-y-1 rounded-xl bg-neutral-900/70 p-4 text-center ring-1 ring-white/10 sm:p-5 xl:space-y-2"
	>
		<div class="font-mono text-3xl tabular-nums sm:text-5xl xl:text-7xl">{pad(num)}</div>
		<div class="text-ssm tracking-wider text-neutral-400 uppercase sm:text-xs xl:text-sm">
			{label}
		</div>
	</div>
{/snippet}
