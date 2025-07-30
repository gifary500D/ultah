<script lang="ts">
	import { onMount } from 'svelte';
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';

	let mounted = false;
	let showGiftAnimation = false;
	let showWishModal = false;
	let wishText = '';
	let showWishConfirmation = false;
	let audioElement: HTMLAudioElement;
	let isPlaying = false;

	let balloons: Array<{
		id: number;
		x: number;
		y: number;
		color: string;
		isDragging: boolean;
		dragOffset: { x: number; y: number };
		autoMoveX: number;
		autoMoveY: number;
	}> = [];

	// Animation stores
	const titleScale = tweened(0, { duration: 1000, easing: cubicOut });
	const heartScale = tweened(0, { duration: 800, easing: cubicOut });

	// Initialize balloons with auto movement
	const initBalloons = () => {
		const colors = ['#FF69B4', '#FF1493', '#8A2BE2', '#9370DB', '#FF6347'];
		balloons = Array.from({ length: 8 }, (_, i) => ({
			id: i,
			x: Math.random() * (window.innerWidth - 100),
			y: Math.random() * (window.innerHeight - 200) + 100,
			color: colors[i % colors.length],
			isDragging: false,
			dragOffset: { x: 0, y: 0 },
			autoMoveX: (Math.random() - 0.5) * 2,
			autoMoveY: (Math.random() - 0.5) * 1
		}));
	};

	// Auto move balloons
	const animateBalloons = () => {
		if (!mounted) return;

		balloons.forEach((balloon) => {
			if (!balloon.isDragging) {
				balloon.x += balloon.autoMoveX;
				balloon.y += balloon.autoMoveY;

				if (balloon.x <= 0 || balloon.x >= window.innerWidth - 80) {
					balloon.autoMoveX *= -1;
					balloon.x = Math.max(0, Math.min(window.innerWidth - 80, balloon.x));
				}
				if (balloon.y <= 0 || balloon.y >= window.innerHeight - 100) {
					balloon.autoMoveY *= -1;
					balloon.y = Math.max(0, Math.min(window.innerHeight - 100, balloon.y));
				}
			}
		});

		balloons = [...balloons];
		requestAnimationFrame(animateBalloons);
	};

	// Balloon drag functions
	const startDrag = (event: MouseEvent | TouchEvent, balloonId: number) => {
		event.preventDefault();
		const balloon = balloons.find((b) => b.id === balloonId);
		if (!balloon) return;

		const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
		const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

		balloon.isDragging = true;
		balloon.dragOffset = {
			x: clientX - balloon.x,
			y: clientY - balloon.y
		};
	};

	const drag = (event: MouseEvent | TouchEvent) => {
		event.preventDefault();
		const draggingBalloon = balloons.find((b) => b.isDragging);
		if (!draggingBalloon) return;

		const clientX = 'touches' in event ? event.touches[0].clientX : event.clientX;
		const clientY = 'touches' in event ? event.touches[0].clientY : event.clientY;

		draggingBalloon.x = Math.max(
			0,
			Math.min(window.innerWidth - 80, clientX - draggingBalloon.dragOffset.x)
		);
		draggingBalloon.y = Math.max(
			0,
			Math.min(window.innerHeight - 100, clientY - draggingBalloon.dragOffset.y)
		);

		balloons = [...balloons];
	};

	const endDrag = () => {
		balloons.forEach((balloon) => {
			balloon.isDragging = false;
		});
		balloons = [...balloons];
	};

	const handleBalloonKeydown = (event: KeyboardEvent, balloonId: number) => {
		if (event.key === 'Enter' || event.key === ' ') {
			event.preventDefault();
			const balloon = balloons.find((b) => b.id === balloonId);
			if (balloon) {
				balloon.autoMoveX *= 1.5;
				balloon.autoMoveY *= 1.5;
				balloons = [...balloons];
			}
		}
	};

	const openGift = () => {
		showGiftAnimation = true;
	};

	const openWishModal = () => {
		showWishModal = true;
	};

	const submitWish = () => {
		if (wishText.trim()) {
			showWishModal = false;
			showWishConfirmation = true;

			setTimeout(() => {
				showWishConfirmation = false;
				wishText = '';
			}, 4000);
		}
	};

	const toggleMusic = async () => {
		if (audioElement) {
			try {
				if (isPlaying) {
					audioElement.pause();
					isPlaying = false;
				} else {
					await audioElement.play();
					isPlaying = true;
				}
			} catch (error) {
				console.log('Audio play failed:', error);
				// Fallback: try to load and play again
				try {
					audioElement.load();
					await audioElement.play();
					isPlaying = true;
				} catch (retryError) {
					console.log('Audio retry failed:', retryError);
					alert('Tidak dapat memutar musik. Pastikan file /feast.mp3 tersedia atau coba klik tombol musik lagi.');
				}
			}
		}
	};

	onMount(() => {
		mounted = true;
		initBalloons();
		titleScale.set(1);
		setTimeout(() => heartScale.set(1), 500);

		requestAnimationFrame(animateBalloons);

		window.addEventListener('mousemove', drag);
		window.addEventListener('mouseup', endDrag);
		window.addEventListener('touchmove', drag, { passive: false });
		window.addEventListener('touchend', endDrag);

		// Improved audio setup
		audioElement = new Audio();
		audioElement.src = '/feast.mp3';
		audioElement.loop = true;
		audioElement.volume = 0.3;
		audioElement.preload = 'auto';
		
		// Add event listeners for audio
		audioElement.addEventListener('canplaythrough', () => {
			console.log('Audio can play through');
			audioElement.currentTime = 40;
		});
		
		audioElement.addEventListener('error', (e) => {
			console.error('Audio error:', e);
		});

		audioElement.addEventListener('ended', () => {
			isPlaying = false;
		});

		audioElement.addEventListener('pause', () => {
			isPlaying = false;
		});

		audioElement.addEventListener('play', () => {
			isPlaying = true;
		});

		return () => {
			window.removeEventListener('mousemove', drag);
			window.removeEventListener('mouseup', endDrag);
			window.removeEventListener('touchmove', drag);
			window.removeEventListener('touchend', endDrag);
			if (audioElement) {
				audioElement.pause();
			}
		};
	});

	let strawberries = Array.from({ length: 6 }, (_, i) => ({
		id: i,
		delay: i * 0.5
	}));
</script>

<svelte:head>
	<title>Happy 20th Birthday Aya! 🍓</title>
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;500;600;700&family=Poppins:wght@300;400;500;600;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<!-- Background with gradient -->
<div
	class="relative min-h-screen overflow-hidden bg-gradient-to-br from-pink-300 via-purple-300 to-pink-400"
>
	<!-- Music Control Button -->
	<button
		on:click={toggleMusic}
		class="fixed top-4 right-4 z-50 rounded-full bg-white/80 p-3 shadow-lg backdrop-blur-sm transition-all duration-300 hover:bg-white/90 hover:scale-110"
		title={isPlaying ? 'Pause Music' : 'Play Music'}
	>
		{#if isPlaying}
			<span class="text-2xl">🔊</span>
		{:else}
			<span class="text-2xl">🔇</span>
		{/if}
	</button>

	<!-- Floating strawberries background -->
	{#if mounted}
		{#each strawberries as strawberry}
			<div
				class="absolute animate-bounce text-4xl opacity-30"
				style="
          left: {Math.random() * 90}%; 
          top: {Math.random() * 80}%;
          animation-delay: {strawberry.delay}s;
          animation-duration: {2 + Math.random() * 2}s;
        "
			>
				🍓
			</div>
		{/each}
	{/if}

	<!-- Auto-moving Draggable Balloons -->
	{#if mounted}
		{#each balloons as balloon}
			<div
				class="absolute z-10 cursor-grab transition-transform hover:scale-110 focus:ring-2 focus:ring-white/50 focus:ring-offset-2 focus:ring-offset-transparent focus:outline-none active:cursor-grabbing"
				style="left: {balloon.x}px; top: {balloon.y}px; transform: {balloon.isDragging
					? 'scale(1.1)'
					: 'scale(1)'};"
				role="button"
				tabindex="0"
				aria-label="Draggable balloon {balloon.id + 1}"
				on:mousedown={(e) => startDrag(e, balloon.id)}
				on:touchstart={(e) => startDrag(e, balloon.id)}
				on:keydown={(e) => handleBalloonKeydown(e, balloon.id)}
			>
				<div class="relative">
					<div
						class="h-20 w-16 animate-pulse rounded-full shadow-lg"
						style="background: linear-gradient(135deg, {balloon.color}, {balloon.color}88);"
					></div>
					<div class="mx-auto h-12 w-px bg-gray-600"></div>
				</div>
			</div>
		{/each}
	{/if}

	<!-- Main Content -->
	<div class="relative z-20 container mx-auto px-4 py-8">
		<!-- Header with animated title -->
		<div class="mb-12 text-center">
			<div style="transform: scale({$titleScale});">
				<h1 class="font-poppins mb-4 text-4xl font-bold text-white drop-shadow-lg md:text-6xl">
					🎉 Happy 20th Birthday! 🎉
				</h1>
				<h2 class="hero-text mb-4 text-6xl font-bold md:text-9xl lg:text-[12rem]">Ayaa</h2>
				<p class="font-poppins text-lg text-white drop-shadow-md md:text-xl">July 31st, 2025 ✨</p>
			</div>
		</div>

		<!-- Photo Section with Circular Design -->
		<div class="mx-auto mb-12 max-w-sm">
			<div class="photo-container relative">
				<div class="photo-frame">
					<img
						src="/images/ayaa.jpeg"
						alt="Foto Aya yang cantik"
						class="photo-image"
					/>
				</div>
				<!-- Decorative elements around photo -->
				<div class="absolute -top-2 -right-2 text-4xl animate-bounce" style="animation-delay: 0s;">🍓</div>
				<div class="absolute -bottom-2 -left-2 text-4xl animate-bounce" style="animation-delay: 0.5s;">💕</div>
				<div class="absolute -top-2 -left-2 text-4xl animate-bounce" style="animation-delay: 1s;">✨</div>
				<div class="absolute -bottom-2 -right-2 text-4xl animate-bounce" style="animation-delay: 1.5s;">🌟</div>
			</div>
		</div>

		<!-- Gift Box Section -->
		<div class="mb-12 text-center">
			<div class="relative inline-block">
				{#if !showGiftAnimation}
					<button
						on:click={openGift}
						class="transform cursor-pointer transition-all duration-300 hover:scale-110"
					>
						<div class="animate-bounce text-8xl">🎁</div>
						<p class="font-poppins mt-2 text-lg font-bold text-white drop-shadow-md">
							Click to open your gift!
						</p>
					</button>
				{:else}
					<div class="gift-animation">
						<div class="mb-4 animate-ping text-8xl">✨</div>
						<div class="mx-auto max-w-sm rounded-2xl bg-white p-6 shadow-2xl">
							<div class="mb-4 text-4xl">🍓💕</div>
							<h3 class="font-poppins mb-3 text-xl font-bold text-pink-600">
								Special Message for Aya!
							</h3>
							<p class="font-poppins leading-relaxed text-gray-700">
								Happy 20th Birthday, beautiful! 🌟<br />
								You're as sweet as strawberries and as lovely as this purple sunset! May your day be
								filled with joy, laughter, and all your favorite things! 🍓💜
							</p>
							<div class="mt-4 text-2xl">🎂🎈🍓💕✨</div>
						</div>
					</div>
				{/if}
			</div>
		</div>

		<!-- Heart Animation -->
		<div class="mb-8 text-center" style="transform: scale({$heartScale});">
			<div class="inline-flex animate-pulse space-x-2 text-4xl">💕 🍓 💜 🍓 💕</div>
		</div>

		<!-- Interactive Birthday Cake with Wish Modal -->
		<div class="mx-auto mb-12 max-w-xs">
			<div class="rounded-3xl bg-white p-6 text-center shadow-2xl">
				<div class="mb-4 animate-bounce text-6xl">🎂</div>
				<button
					on:click={openWishModal}
					class="font-poppins transform rounded-full bg-gradient-to-r from-pink-500 to-purple-500 px-6 py-3 font-bold text-white shadow-lg transition-all duration-300 hover:scale-105 hover:from-pink-600 hover:to-purple-600"
				>
					Make a Wish! ✨
				</button>
			</div>
		</div>

		<!-- Birthday Message -->
		<div class="mx-auto max-w-2xl rounded-3xl bg-white/90 p-8 shadow-2xl backdrop-blur-sm">
			<h3 class="font-poppins mb-6 text-center text-2xl font-bold text-pink-600">
				💌 Sweet Birthday Wishes 💌
			</h3>
			<div class="font-poppins space-y-4 text-gray-700">
				<p class="text-center text-lg">🍓 Another year of sweetness has begun! 🍓</p>
				<p class="text-center">
					On this special day, I want you to know how incredibly special you are. Like strawberries
					in summer, you bring sweetness to every moment. Your smile lights up the world brighter
					than any candle on a cake!
				</p>
				<p class="text-center">May this new chapter of your life be filled with:</p>
				<div class="mt-6 grid grid-cols-2 gap-4 text-center">
					<div class="rounded-2xl bg-pink-100 p-4">
						<div class="mb-2 text-3xl">🌟</div>
						<p class="font-poppins font-semibold text-pink-600">Amazing Adventures</p>
					</div>
					<div class="rounded-2xl bg-purple-100 p-4">
						<div class="mb-2 text-3xl">💕</div>
						<p class="font-poppins font-semibold text-purple-600">Endless Love</p>
					</div>
					<div class="rounded-2xl bg-pink-100 p-4">
						<div class="mb-2 text-3xl">🍓</div>
						<p class="font-poppins font-semibold text-pink-600">Sweet Moments</p>
					</div>
					<div class="rounded-2xl bg-purple-100 p-4">
						<div class="mb-2 text-3xl">🌈</div>
						<p class="font-poppins font-semibold text-purple-600">Colorful Dreams</p>
					</div>
				</div>
				<p class="font-poppins mt-6 text-center text-lg font-bold text-pink-600">
					Happy 20th Birthday, Aya! 🎉🍓💜
				</p>
			</div>
		</div>

		<!-- Footer with floating hearts -->
		<div class="relative mt-12 text-center">
			<div class="inline-block">
				{#each Array(5) as _, i}
					<span
						class="absolute animate-ping text-2xl opacity-60"
						style="
              left: {(i - 2) * 30}px; 
              animation-delay: {i * 0.3}s;
              animation-duration: 2s;
            "
					>
						💕
					</span>
				{/each}
				<p class="font-poppins pt-8 text-lg font-bold text-white drop-shadow-lg">
					With all my love 💕
				</p>
			</div>
		</div>
	</div>
</div>

<!-- Wish Modal -->
{#if showWishModal}
	<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4 backdrop-blur-sm">
		<div class="w-full max-w-md transform animate-pulse rounded-3xl bg-white p-8 shadow-2xl">
			<div class="mb-6 text-center">
				<div class="mb-4 text-6xl">🕯️</div>
				<h3 class="font-poppins mb-2 text-2xl font-bold text-pink-600">Make Your Birthday Wish</h3>
				<p class="font-poppins text-gray-600">Close your eyes and make a special wish! ✨</p>
			</div>

			<textarea
				bind:value={wishText}
				placeholder="Write your wish here..."
				class="font-poppins h-32 w-full resize-none rounded-2xl border-2 border-pink-300 p-4 focus:border-purple-400 focus:outline-none"
			></textarea>

			<div class="mt-6 flex gap-4">
				<button
					on:click={() => (showWishModal = false)}
					class="font-poppins flex-1 rounded-full bg-gray-300 py-3 font-bold text-gray-700 transition-colors hover:bg-gray-400"
				>
					Cancel
				</button>
				<button
					on:click={submitWish}
					class="font-poppins flex-1 rounded-full bg-gradient-to-r from-pink-500 to-purple-500 py-3 font-bold text-white transition-all hover:from-pink-600 hover:to-purple-600"
				>
					🌟 Submit Wish
				</button>
			</div>
		</div>
	</div>
{/if}

<!-- Wish Confirmation -->
{#if showWishConfirmation}
	<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4 backdrop-blur-sm">
		<div class="wish-confirmation w-full max-w-md rounded-3xl bg-white p-8 text-center shadow-2xl">
			<div class="mb-4 animate-bounce text-8xl">✨</div>
			<h3 class="font-poppins mb-4 text-2xl font-bold text-purple-600">Wish Received!</h3>
			<p class="font-poppins mb-4 text-lg text-gray-700">
				🌟 <strong>May all your dreams come true!</strong> 🌟
			</p>
			<p class="font-poppins text-gray-600">
				Your wish has been sent to the universe on this magical day! 💫
			</p>
			<div class="mt-6 text-4xl">🎂🎈🍓💕✨</div>
		</div>
	</div>
{/if}

<style>
	:global(body) {
		user-select: none;
		-webkit-user-select: none;
		-moz-user-select: none;
		-ms-user-select: none;
	}

	.font-poppins {
		font-family: 'Poppins', sans-serif;
	}

	.hero-text {
		font-family: 'Dancing Script', cursive;
		background: linear-gradient(45deg, #ffffff, #ffd700, #ff69b4, #ffffff, #8a2be2);
		background-size: 400% 400%;
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
		animation: gradientShift 3s ease-in-out infinite;
		text-shadow: 
			3px 3px 0px #ff1493,
			6px 6px 0px #8a2be2,
			0 0 30px rgba(255, 255, 255, 0.8),
			0 0 40px rgba(255, 105, 180, 0.6);
		filter: drop-shadow(2px 2px 8px rgba(0, 0, 0, 0.3));
		position: relative;
	}

	.hero-text::before {
		content: 'Ayaa';
		position: absolute;
		top: 0;
		left: 0;
		background: linear-gradient(45deg, #ff1493, #8a2be2, #ff69b4, #9370db);
		-webkit-background-clip: text;
		background-clip: text;
		-webkit-text-fill-color: transparent;
		z-index: -1;
		filter: blur(3px);
		opacity: 0.7;
	}

	@keyframes gradientShift {
		0%, 100% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
	}

	/* Photo styling - Circular design */
	.photo-container {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 20px;
	}

	.photo-frame {
		position: relative;
		width: 280px;
		height: 280px;
		border-radius: 50%;
		background: conic-gradient(from 0deg, #ff69b4, #ff1493, #8a2be2, #9370db, #ff6347, #ff69b4);
		padding: 8px;
		animation: rotate 10s linear infinite;
		box-shadow: 
			0 0 30px rgba(255, 105, 180, 0.6),
			0 0 60px rgba(138, 43, 226, 0.4),
			inset 0 0 20px rgba(255, 255, 255, 0.2);
	}

	.photo-frame::before {
		content: '';
		position: absolute;
		top: -15px;
		left: -15px;
		right: -15px;
		bottom: -15px;
		border-radius: 50%;
		background: conic-gradient(from 180deg, #ff69b4, #ff1493, #8a2be2, #9370db, #ff6347, #ff69b4);
		animation: rotate 8s linear infinite reverse;
		z-index: -1;
		filter: blur(20px);
		opacity: 0.8;
	}

	.photo-image {
		width: 100%;
		height: 100%;
		border-radius: 50%;
		object-fit: cover;
		border: 4px solid rgba(255, 255, 255, 0.9);
		box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.1);
	}

	@keyframes rotate {
		from {
			transform: rotate(0deg);
		}
		to {
			transform: rotate(360deg);
		}
	}

	.gift-animation {
		animation: giftPop 0.8s cubic-bezier(0.68, -0.55, 0.265, 1.55);
	}

	.wish-confirmation {
		animation: wishPop 0.8s cubic-bezier(0.68, -0.55, 0.265, 1.55);
	}

	@keyframes giftPop {
		0% {
			opacity: 0;
			transform: scale(0.3) rotate(-10deg);
		}
		50% {
			transform: scale(1.1) rotate(5deg);
		}
		100% {
			opacity: 1;
			transform: scale(1) rotate(0deg);
		}
	}

	@keyframes wishPop {
		0% {
			opacity: 0;
			transform: scale(0.5);
		}
		70% {
			transform: scale(1.1);
		}
		100% {
			opacity: 1;
			transform: scale(1);
		}
	}

	/* Mobile responsiveness */
	@media (max-width: 768px) {
		.container {
			padding-left: 1rem;
			padding-right: 1rem;
		}

		.hero-text {
			font-size: 3rem;
		}

		.photo-frame {
			width: 220px;
			height: 220px;
		}
	}

	@media (max-width: 480px) {
		.hero-text {
			font-size: 2.5rem;
		}

		.photo-frame {
			width: 200px;
			height: 200px;
		}
	}
</style>