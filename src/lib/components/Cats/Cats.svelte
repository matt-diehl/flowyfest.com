<div class="container">
	<video src="/video/cats.m4v" autoplay muted loop playsinline></video>
</div>

<style lang="scss">
	.container {
		animation: bounceAround 3s ease-in-out infinite;
		aspect-ratio: 9/16;
		background-color: black;
		border-radius: 6px;
		height: 60dvh;
		padding: 6px;
		position: fixed;
	}

	video {
		width: 100%;
		height: auto;
		border-radius: 4px;
	}

	@keyframes bounceAround {
		0%,
		100% {
			transform: translateY(0) rotate(0deg);
		}
		25% {
			transform: translateX(80vw) translateY(50vh) rotate(-5deg);
		}
		50% {
			transform: translateX(30vw) translateY(20vh) rotate(15deg);
		}
		75% {
			transform: translateX(70vw) translateY(10vh) rotate(-70deg);
		}
	}
</style>
