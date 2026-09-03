<script lang="ts">
	import { onMount } from 'svelte';
	let { onBubblePop } = $props();

	let canvas: HTMLCanvasElement;

	class Bubble {
		x: number;
		y: number;
		radius: number;
		baseSpeed: number;
		wobbleSpeed: number;
		wobbleAmp: number;
		seed: number;

		constructor(w: number) {
			this.radius = 28 + Math.random() * 22;
			this.x = Math.random() * w;
			this.y = -this.radius - Math.random() * 40;
			this.baseSpeed = 0.4 + Math.random() * 0.5;
			this.wobbleSpeed = 0.01 + Math.random() * 0.01;
			this.wobbleAmp = 0.4 + Math.random() * 0.5;
			this.seed = Math.random() * 1000;
		}

		update(time: number): void {
			this.y += this.baseSpeed;
			this.x += Math.sin(time * this.wobbleSpeed + this.seed) * this.wobbleAmp;
		}

		draw(ctx: CanvasRenderingContext2D): void {
			ctx.save();
			ctx.translate(this.x, this.y);

			const grad = ctx.createRadialGradient(
				-this.radius * 0.2,
				-this.radius * 0.2,
				this.radius * 0.1,
				0,
				0,
				this.radius
			);
			grad.addColorStop(0, 'rgba(255, 255, 255, 0.45)');
			grad.addColorStop(0.65, 'rgba(235, 240, 255, 0.25)');
			grad.addColorStop(0.92, 'rgba(255, 255, 255, 0.65)');
			grad.addColorStop(1, 'rgba(255, 255, 255, 0)');

			ctx.fillStyle = grad;
			ctx.beginPath();
			ctx.arc(0, 0, this.radius, 0, Math.PI * 2);
			ctx.fill();

			ctx.beginPath();
			ctx.arc(-this.radius * 0.35, -this.radius * 0.35, this.radius * 0.18, 0, Math.PI * 2);
			ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
			ctx.fill();

			ctx.restore();
		}
	}

	class FogParticle {
		x: number;
		y: number;
		vx: number;
		vy: number;
		radius: number;
		maxRadius: number;
		alpha: number;
		decay: number;
		expansion: number;

		constructor(x: number, y: number, parentRadius: number) {
			this.x = x;
			this.y = y;
			const angle = Math.random() * Math.PI * 2;
			const speed = 0.25 + Math.random() * 1.4;
			this.vx = Math.cos(angle) * speed;
			this.vy = Math.sin(angle) * speed + 0.15;
			this.radius = parentRadius * 0.35;
			this.maxRadius = parentRadius * (1.2 + Math.random() * 0.8);
			this.alpha = 0.4 + Math.random() * 0.25;
			this.decay = 0.0035 + Math.random() * 0.003;
			this.expansion = (this.maxRadius - this.radius) * this.decay * 1.2;
		}

		update(): void {
			this.x += this.vx;
			this.y += this.vy;
			this.vx *= 0.98;
			this.vy *= 0.98;
			this.radius += this.expansion;
			this.alpha -= this.decay;
		}

		draw(ctx: CanvasRenderingContext2D): void {
			if (this.alpha <= 0) return;

			ctx.save();
			const grad = ctx.createRadialGradient(this.x, this.y, 0, this.x, this.y, this.radius);
			grad.addColorStop(0, `rgba(240, 245, 255, ${this.alpha})`);
			grad.addColorStop(0.5, `rgba(230, 235, 250, ${this.alpha * 0.4})`);
			grad.addColorStop(1, 'rgba(255, 255, 255, 0)');

			ctx.fillStyle = grad;
			ctx.beginPath();
			ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
			ctx.fill();
			ctx.restore();
		}
	}

	onMount(() => {
		const ctx = canvas.getContext('2d');
		if (!ctx) return;

		let animId: number;
		let width = (canvas.width = window.innerWidth);
		let height = (canvas.height = window.innerHeight);

		const bubbles: Bubble[] = [];
		const particles: FogParticle[] = [];
		let lastSpawn = 0;
		let pointer: { x: number; y: number } | null = null;

		const handleResize = (): void => {
			width = canvas.width = window.innerWidth;
			height = canvas.height = window.innerHeight;
		};

		const popBubble = (index: number): void => {
			const b = bubbles[index];
			const count = 16 + Math.floor(Math.random() * 6);
			for (let i = 0; i < count; i++) {
				particles.push(new FogParticle(b.x, b.y, b.radius));
			}
			bubbles.splice(index, 1);
			onBubblePop();
		};

		const checkCollision = (px: number, py: number): void => {
			for (let i = bubbles.length - 1; i >= 0; i--) {
				const b = bubbles[i];
				const dx = px - b.x;
				const dy = py - b.y;
				if (dx * dx + dy * dy <= b.radius * b.radius) {
					popBubble(i);
				}
			}
		};

		const handlePointerMove = (e: PointerEvent): void => {
			const rect = canvas.getBoundingClientRect();
			pointer = {
				x: e.clientX - rect.left,
				y: e.clientY - rect.top
			};
			checkCollision(pointer.x, pointer.y);
		};

		const handlePointerLeave = (): void => {
			pointer = null;
		};

		window.addEventListener('resize', handleResize);
		canvas.addEventListener('pointermove', handlePointerMove);
		canvas.addEventListener('pointerdown', handlePointerMove);
		canvas.addEventListener('pointerleave', handlePointerLeave);

		const animate = (time: number): void => {
			ctx.clearRect(0, 0, width, height);

			if (time - lastSpawn > 1100 && bubbles.length < 18) {
				bubbles.push(new Bubble(width));
				lastSpawn = time;
			}

			for (let i = particles.length - 1; i >= 0; i--) {
				const p = particles[i];
				p.update();
				p.draw(ctx);
				if (p.alpha <= 0) particles.splice(i, 1);
			}

			for (let i = bubbles.length - 1; i >= 0; i--) {
				const b = bubbles[i];
				b.update(time * 0.03);
				b.draw(ctx);
				if (b.y - b.radius > height + 20) bubbles.splice(i, 1);
			}

			// Check collision each frame for bubbles drifting into a stationary cursor
			if (pointer) {
				checkCollision(pointer.x, pointer.y);
			}

			animId = requestAnimationFrame(animate);
		};

		animId = requestAnimationFrame(animate);

		return () => {
			cancelAnimationFrame(animId);
			window.removeEventListener('resize', handleResize);
			canvas.removeEventListener('pointermove', handlePointerMove);
			canvas.removeEventListener('pointerdown', handlePointerMove);
			canvas.removeEventListener('pointerleave', handlePointerLeave);
		};
	});
</script>

<canvas bind:this={canvas}></canvas>

<style>
	canvas {
		position: fixed;
		inset: 0;
		width: 100vw;
		height: 100vh;
		display: block;
		z-index: 0;
		pointer-events: auto;
	}
</style>
