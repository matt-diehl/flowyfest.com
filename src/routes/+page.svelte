<script lang="ts">
	import { onMount } from 'svelte';
	import Bubbles from '$lib/components/Bubbles/Bubbles.svelte';
	import Cats from '$lib/components/Cats/Cats.svelte';

	const ANIMATION_INTERVAL_MS = 200;

	let audioElement = $state<HTMLAudioElement>();
	let backgroundHueRotation = $state(12);
	let isAudioPlaying = $state(false);
	let totalBubblePops = $state(0);

	onMount(() => {
		const interval = setInterval(() => {
			backgroundHueRotation = Math.random() * 360;
		}, ANIMATION_INTERVAL_MS);

		document.addEventListener(
			'click',
			() => {
				if (!audioElement) return;
				if (!isAudioPlaying) {
					audioElement.play();
					isAudioPlaying = true;
				}
			},
			{ once: true }
		);

		return () => clearInterval(interval);
	});

	function playAudio() {
		if (!audioElement) return;
		audioElement.play();
		isAudioPlaying = true;
	}

	function pauseAudio() {
		if (!audioElement) return;
		audioElement.pause();
		isAudioPlaying = false;
	}

	function toggleAudio() {
		if (!audioElement) return;
		if (!isAudioPlaying) {
			playAudio();
		} else {
			pauseAudio();
		}
	}

	function onBubblePop() {
		totalBubblePops += 1;
	}
</script>

<div class="background" style="filter: hue-rotate({backgroundHueRotation}deg)">
	<h1 class="title">
		<span class="title-letter">F</span><span class="title-letter">l</span><span class="title-letter"
			>o</span
		><span class="title-letter">w</span><span class="title-letter">y</span>
		<span class="title-letter">F</span><span class="title-letter">e</span><span class="title-letter"
			>s</span
		><span class="title-letter">t</span>
	</h1>
	{#if totalBubblePops % 3 === 0 && totalBubblePops > 0}
		<Cats />
	{/if}
	<Bubbles {onBubblePop} />
	<audio bind:this={audioElement} loop controls={false} src="/music/background-music.mp3"></audio>
	<button class="music-button" onclick={toggleAudio}>
		{isAudioPlaying ? 'Pause music' : 'Play music'}
	</button>
</div>

<style lang="scss">
	:root {
		--neon-primary: #00f0ff; /* Electric cyan base */
		--neon-secondary: #ff007b; /* Accent glow */
	}

	.background {
		background-image: url('/img/flowy-dj-dt.jpg');
		background-size: cover;
		background-position: 50% 50%;
		height: 100dvh;
		width: 100dvw;
	}

	.music-button {
		background-color: var(--neon-primary);
		color: black;
		font-family: 'Modak', cursive;
		font-size: 1.2rem;
		padding: 0.5rem 1rem;
		border: 2px solid var(--neon-secondary);
		position: fixed;
		bottom: 20px;
		right: 20px;
		z-index: 10;
	}

	.title {
		color: #ffffff;
		color: white;
		font-family: 'Kablammo', system-ui, cursive;
		font-size: clamp(3rem, 15dvw, 10rem);
		left: 50%;
		letter-spacing: 0.05em;
		position: fixed;
		text-align: center;
		text-transform: uppercase;
		top: 5dvh;
		transform: translateX(-50%);
		transform-origin: center;
		user-select: none;
		white-space: nowrap;
		will-change: transform, color, text-shadow;
	}

	.title-letter {
		display: inline-block;
		animation:
			neon-pulse 3s ease-in-out infinite alternate,
			dance 0.5s ease-in-out infinite alternate;
		will-change: transform, color, text-shadow;
	}
	.title-letter:nth-child(1) {
		animation-delay: 0s;
	}
	.title-letter:nth-child(2) {
		animation-delay: 0.2s;
	}
	.title-letter:nth-child(3) {
		animation-delay: 0.4s;
	}
	.title-letter:nth-child(4) {
		animation-delay: 0.6s;
	}
	.title-letter:nth-child(5) {
		animation-delay: 0.8s;
	}
	.title-letter:nth-child(6) {
		animation-delay: 1s;
	}
	.title-letter:nth-child(7) {
		animation-delay: 1.2s;
	}
	.title-letter:nth-child(8) {
		animation-delay: 1.4s;
	}
	.title-letter:nth-child(9) {
		animation-delay: 1.6s;
	}

	@keyframes dance {
		0% {
			transform: translateY(0) rotate(-2deg);
		}
		100% {
			transform: translateY(-10px) rotate(2deg);
		}
	}
	@keyframes neon-pulse {
		0% {
			text-shadow:
				0 0 4px #ffffff,
				0 0 10px var(--neon-primary),
				0 0 20px var(--neon-primary),
				0 0 40px var(--neon-primary),
				0 0 70px var(--neon-secondary);
		}
		100% {
			text-shadow:
				0 0 2px #ffffff,
				0 0 6px var(--neon-primary),
				0 0 12px var(--neon-primary),
				0 0 25px var(--neon-primary),
				0 0 45px var(--neon-secondary);
		}
	}
</style>
