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
	let isMobile = false;

	const downloadVoucher = () => {
		const canvas = document.createElement('canvas');
		const ctx = canvas.getContext('2d');

		if (!ctx) {
			console.error('Canvas context not supported.');
			return;
		}

		canvas.width = 400;
		canvas.height = 300;

		const gradient = ctx.createLinearGradient(0, 0, 400, 300);
		gradient.addColorStop(0, '#f3e8ff');
		gradient.addColorStop(1, '#fce7f3');
		ctx.fillStyle = gradient;
		ctx.fillRect(0, 0, 400, 300);

		ctx.strokeStyle = '#8b5cf6';
		ctx.lineWidth = 3;
		ctx.setLineDash([5, 5]);
		ctx.strokeRect(10, 10, 380, 280);

		ctx.fillStyle = '#7c3aed';
		ctx.font = 'bold 24px Arial';
		ctx.textAlign = 'center';
		ctx.fillText('🎁 GIFT VOUCHER 🎁', 200, 60);

		ctx.font = '16px Arial';
		ctx.fillStyle = '#6b21a8';
		ctx.fillText('Happy 20th Birthday!', 200, 100);

		ctx.font = 'bold 18px Arial';
		ctx.fillStyle = '#be185d';
		ctx.fillText('For: Beautiful Aya 💕', 200, 140);

		ctx.font = '14px Arial';
		ctx.fillStyle = '#6b7280';
		ctx.fillText('Valid for: One Special Surprise', 200, 170);
		ctx.fillText('July 31st, 2025', 200, 190);

		ctx.font = '20px Arial';
		ctx.fillText('🍓🎂🎈', 200, 230);

		ctx.font = '12px Arial';
		ctx.fillText('Show this voucher to claim your gift!', 200, 260);

		canvas.toBlob((blob) => {
			if (!blob) return;
			const url = URL.createObjectURL(blob);
			const a = document.createElement('a');
			a.href = url;
			a.download = 'aya-birthday-gift-voucher.png';
			document.body.appendChild(a);
			a.click();
			document.body.removeChild(a);
			URL.revokeObjectURL(url);
		});
	};

	// Simplified balloons - only auto-moving, no drag
	let balloons: Array<{
		id: number;
		x: number;
		y: number;
		color: string;
		autoMoveX: number;
		autoMoveY: number;
	}> = [];

	// Animation stores
	const titleScale = tweened(0, { duration: 800, easing: cubicOut });
	const heartScale = tweened(0, { duration: 600, easing: cubicOut });

	// Check if device is mobile
	const checkMobile = () => {
		isMobile =
			window.innerWidth < 768 ||
			/Android|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent);
	};

	// Initialize balloons - auto-moving only
	const initBalloons = () => {
		const colors = ['#FF69B4', '#FF1493', '#8A2BE2', '#9370DB', '#FF6347'];
		const balloonCount = isMobile ? 5 : 8; // Increased balloon count

		balloons = Array.from({ length: balloonCount }, (_, i) => ({
			id: i,
			x: Math.random() * (window.innerWidth - 100),
			y: Math.random() * (window.innerHeight - 200) + 100,
			color: colors[i % colors.length],
			autoMoveX: (Math.random() - 0.5) * (isMobile ? 0.8 : 1.5),
			autoMoveY: (Math.random() - 0.5) * (isMobile ? 0.4 : 0.8)
		}));
	};

	let animationId: number;

	// Simplified balloon animation - auto-move only
	const animateBalloons = () => {
		if (!mounted) return;

		balloons.forEach((balloon) => {
			balloon.x += balloon.autoMoveX;
			balloon.y += balloon.autoMoveY;

			// Bounce off edges
			if (balloon.x <= 0 || balloon.x >= window.innerWidth - 80) {
				balloon.autoMoveX *= -1;
				balloon.x = Math.max(0, Math.min(window.innerWidth - 80, balloon.x));
			}
			if (balloon.y <= 0 || balloon.y >= window.innerHeight - 100) {
				balloon.autoMoveY *= -1;
				balloon.y = Math.max(0, Math.min(window.innerHeight - 100, balloon.y));
			}
		});

		balloons = [...balloons];

		// Reduce animation frame rate for mobile
		const delay = isMobile ? 50 : 16;
		setTimeout(() => {
			animationId = requestAnimationFrame(animateBalloons);
		}, delay);
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
					if (isMobile) {
						audioElement.volume = 0.1;
					}
					await audioElement.play();
					isPlaying = true;
				}
			} catch (error) {
				console.log('Audio play failed:', error);
				if (isMobile) {
					alert('Tap tombol musik sekali lagi untuk memutar audio');
				} else {
					alert('Tidak dapat memutar musik. Coba klik tombol musik lagi.');
				}
			}
		}
	};

	onMount(() => {
		mounted = true;
		checkMobile();
		initBalloons();
		titleScale.set(1);
		setTimeout(() => heartScale.set(1), 300);

		// Start balloon animation
		animationId = requestAnimationFrame(animateBalloons);

		// Audio setup
		audioElement = new Audio();
		audioElement.src = '/feast.mp3';
		audioElement.loop = true;
		audioElement.volume = isMobile ? 0.1 : 0.3;
		audioElement.preload = isMobile ? 'none' : 'auto';

		audioElement.addEventListener('canplaythrough', () => {
			if (!isMobile) {
				audioElement.currentTime = 40;
			}
		});

		audioElement.addEventListener('error', (e) => {
			console.error('Audio error:', e);
		});

		// Handle resize for mobile orientation changes
		const handleResize = () => {
			checkMobile();
			if (balloons.length > 0) {
				setTimeout(() => {
					initBalloons();
				}, 100);
			}
		};

		window.addEventListener('resize', handleResize, { passive: true });

		return () => {
			if (animationId) {
				cancelAnimationFrame(animationId);
			}
			window.removeEventListener('resize', handleResize);
			if (audioElement) {
				audioElement.pause();
			}
		};
	});

	// Reduced strawberries for mobile
	let strawberries = Array.from({ length: isMobile ? 3 : 4 }, (_, i) => ({
		id: i,
		delay: i * 1
	}));
</script>

<svelte:head>
	<title>Happy 20th Birthday Ayaa! 🍓</title>
	<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes" />
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" />
	<link
		href="https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;500;600;700&family=Poppins:wght@300;400;500;600;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<!-- Background with gradient -->
<div
	class="relative min-h-screen overflow-x-hidden bg-gradient-to-br from-pink-300 via-purple-300 to-pink-400"
>
	<!-- Music Control Button - Hidden since we have music player below -->
	<div class="hidden">
		<button
			on:click={toggleMusic}
			class="fixed top-4 right-4 z-50 rounded-full bg-white/80 p-3 shadow-lg backdrop-blur-sm transition-all duration-300 hover:scale-110 hover:bg-white/90"
			title={isPlaying ? 'Pause Music' : 'Play Music'}
		>
			{#if isPlaying}
				<span class="text-2xl">🔊</span>
			{:else}
				<span class="text-2xl">🔇</span>
			{/if}
		</button>
	</div>

	<!-- Floating strawberries background - reduced for mobile -->
	{#if mounted && !isMobile}
		{#each strawberries as strawberry}
			<div
				class="pointer-events-none absolute animate-bounce text-4xl opacity-20"
				style="
          left: {Math.random() * 90}%; 
          top: {Math.random() * 80}%;
          animation-delay: {strawberry.delay}s;
          animation-duration: {3 + Math.random() * 2}s;
        "
			>
				🍓
			</div>
		{/each}
	{/if}

	<!-- Auto-moving Balloons - no drag functionality -->
	{#if mounted}
		{#each balloons as balloon}
			<div
				class="pointer-events-none absolute z-10"
				style="left: {balloon.x}px; top: {balloon.y}px;"
			>
				<div class="relative">
					<div
						class="h-16 w-12 rounded-full shadow-lg {isMobile ? '' : 'animate-pulse'}"
						style="background: linear-gradient(135deg, {balloon.color}, {balloon.color}88);"
					></div>
					<div class="mx-auto h-8 w-px bg-gray-600"></div>
				</div>
			</div>
		{/each}
	{/if}

	<!-- Main Content -->
	<div class="relative z-20 container mx-auto px-4 py-6">
		<!-- Header with animated title -->
		<div class="mb-8 text-center">
			<div style="transform: scale({$titleScale});">
				<h1 class="font-poppins mb-4 text-3xl font-bold text-white drop-shadow-lg md:text-5xl">
					🎉 Happy 20th Birthday! 🎉
				</h1>
				<h2 class="hero-text mb-4 text-8xl font-bold md:text-8xl lg:text-9xl">Ayaa</h2>
				<p class="font-poppins text-lg text-white drop-shadow-md">✨ July 31st, 2005 ✨</p>
			</div>
		</div>

		<!-- Photo Section with Circular Design - simplified for mobile -->
		<div class="mx-auto mb-8 max-w-sm">
			<div class="photo-container relative">
				<div class="photo-frame {isMobile ? 'mobile-photo' : ''}">
					<img
						src="/images/ayaa.jpeg"
						alt="Foto Aya yang cantik"
						class="photo-image"
						loading="lazy"
					/>
				</div>
				<!-- Reduced decorative elements for mobile -->
				{#if !isMobile}
					<div
						class="pointer-events-none absolute -top-2 -right-2 animate-bounce text-3xl"
						style="animation-delay: 0s;"
					>
						🍓
					</div>
					<div
						class="pointer-events-none absolute -bottom-2 -left-2 animate-bounce text-3xl"
						style="animation-delay: 0.5s;"
					>
						💕
					</div>
					<div
						class="pointer-events-none absolute -top-2 -left-2 animate-bounce text-3xl"
						style="animation-delay: 1s;"
					>
						✨
					</div>
					<div
						class="pointer-events-none absolute -right-2 -bottom-2 animate-bounce text-3xl"
						style="animation-delay: 1.5s;"
					>
						🌟
					</div>
				{/if}
			</div>
		</div>

		<!-- Music Player Section -->
		<div class="mx-auto mb-8 max-w-xs">
			<div class="rounded-3xl bg-white/90 p-6 text-center shadow-2xl backdrop-blur-sm">
				<div class="mb-3 text-4xl">🎵</div>
				<h3 class="font-poppins mb-3 text-lg font-bold text-purple-600">🎶 Birthday Song 🎶</h3>
				<p class="font-poppins mb-4 text-sm text-gray-600">
					"Feast" - Special song for your special day!
				</p>
				<button
					on:click={toggleMusic}
					class="font-poppins transform rounded-full bg-gradient-to-r from-purple-500 to-pink-500 px-8 py-3 text-sm font-bold text-white shadow-lg transition-all duration-300 hover:scale-105 hover:from-purple-600 hover:to-pink-600"
				>
					<span class="inline-flex items-center gap-2">
						{#if isPlaying}
							<span>⏸️</span>
							<span>Pause Music</span>
						{:else}
							<span>▶️</span>
							<span>Play Music</span>
						{/if}
					</span>
				</button>
				{#if isPlaying}
					<div class="mt-3 flex items-center justify-center space-x-1">
						<div class="h-3 w-1 animate-pulse rounded-full bg-purple-500"></div>
						<div
							class="h-4 w-1 animate-pulse rounded-full bg-pink-500"
							style="animation-delay: 0.1s;"
						></div>
						<div
							class="h-5 w-1 animate-pulse rounded-full bg-purple-500"
							style="animation-delay: 0.2s;"
						></div>
						<div
							class="h-3 w-1 animate-pulse rounded-full bg-pink-500"
							style="animation-delay: 0.3s;"
						></div>
						<div
							class="h-4 w-1 animate-pulse rounded-full bg-purple-500"
							style="animation-delay: 0.4s;"
						></div>
					</div>
				{/if}
			</div>
		</div>

		<!-- Gift Box Section -->
		<div class="mb-8 text-center">
			<div class="relative inline-block">
				{#if !showGiftAnimation}
					<button
						on:click={openGift}
						class="transform cursor-pointer transition-all duration-300 hover:scale-105"
					>
						<div class="text-6xl {isMobile ? '' : 'animate-bounce'}">🎁</div>
						<p class="font-poppins mt-2 text-base font-bold text-white drop-shadow-md">
							Click to open your gift!
						</p>
					</button>
				{:else}
					<div class="gift-animation">
						<div class="mb-4 text-6xl {isMobile ? '' : 'animate-ping'}">✨</div>
						<div class="mx-auto max-w-sm rounded-2xl bg-white p-6 shadow-2xl">
							<div class="mb-4 text-4xl">🍓💕</div>
							<h3 class="font-poppins mb-3 text-lg font-bold text-pink-600">
								Special Message for Aya!
							</h3>
							<p class="font-poppins text-sm leading-relaxed text-gray-700">
								Happy 20th Birthday, beautiful! 🌟<br />
								You're as sweet as strawberries and as lovely as this sunset! May your day be filled
								with joy, laughter, and all your favorite things! 🍓💜
							</p>
							<div class="mt-4 text-xl">🎂🎈🍓💕✨</div>
						</div>

						<!-- Gift Card Section -->
						<div
							class="mx-auto mt-6 max-w-sm rounded-2xl bg-gradient-to-br from-purple-100 to-pink-100 p-6 shadow-2xl"
						>
							<div class="mb-3 text-3xl">🎁</div>
							<h4 class="font-poppins mb-2 text-base font-bold text-purple-700">
								🎀 Special Gift Card 🎀
							</h4>
							<p class="font-poppins mb-4 text-xs leading-relaxed text-gray-700">
								This card can be exchanged for a real surprise gift! Just show this to me when we
								meet 😊
							</p>
							<div
								class="rounded-xl border-2 border-dashed border-purple-300 bg-white p-4 shadow-lg"
							>
								<div class="text-center">
									<div class="mb-2 text-2xl">🌟</div>
									<p class="font-poppins text-xs font-bold text-purple-600">GIFT VOUCHER</p>
									<p class="font-poppins mt-1 text-xs text-gray-600">
										Valid for: One Special Surprise
									</p>
									<p class="font-poppins text-xs text-gray-600">For: Beautiful Aya 💕</p>
									<div class="mt-2 text-lg">🍓🎂🎈</div>
								</div>
							</div>
							<button
								on:click={downloadVoucher}
								class="font-poppins mt-4 w-full transform rounded-full bg-gradient-to-r from-green-500 to-blue-500 px-4 py-2 text-xs font-bold text-white shadow-lg transition-all duration-300 hover:scale-105 hover:from-green-600 hover:to-blue-600"
							>
								📥 Download Voucher
							</button>
						</div>
					</div>
				{/if}
			</div>
		</div>

		<!-- Heart Animation - simplified for mobile -->
		<div class="mb-6 text-center" style="transform: scale({$heartScale});">
			<div class="inline-flex space-x-2 text-3xl {isMobile ? '' : 'animate-pulse'}">
				💕 🍓 💜 🍓 💕
			</div>
		</div>

		<!-- Interactive Birthday Cake -->
		<div class="mx-auto mb-8 max-w-xs">
			<div class="rounded-3xl bg-white p-6 text-center shadow-2xl">
				<div class="mb-4 text-5xl {isMobile ? '' : 'animate-bounce'}">🎂</div>
				<button
					on:click={openWishModal}
					class="font-poppins transform rounded-full bg-gradient-to-r from-pink-500 to-purple-500 px-6 py-3 text-sm font-bold text-white shadow-lg transition-all duration-300 hover:scale-105 hover:from-pink-600 hover:to-purple-600"
				>
					Make a Wish! ✨
				</button>
			</div>
		</div>

		<!-- Birthday Message - simplified for mobile -->
		<div class="mx-auto max-w-2xl rounded-3xl bg-white/90 p-6 shadow-2xl backdrop-blur-sm">
			<h3 class="font-poppins mb-4 text-center text-xl font-bold text-pink-600">
				💌 Sweet Birthday Wishes 💌
			</h3>
			<div class="font-poppins space-y-3 text-gray-700">
				<p class="text-center text-base">🍓 Another year of sweetness has begun! 🍓</p>
				<p class="text-center text-sm">
					On this special day, I want you to know how incredibly special you are. Like strawberries
					in summer, you bring sweetness to every moment. Your smile lights up the world brighter
					than any candle on a cake!
				</p>
				<p class="text-center text-sm">May this new chapter of your life be filled with:</p>
				<div class="mt-4 grid grid-cols-2 gap-3 text-center">
					<div class="rounded-2xl bg-pink-100 p-3">
						<div class="mb-1 text-2xl">🌟</div>
						<p class="font-poppins text-xs font-semibold text-pink-600">Amazing Life</p>
					</div>
					<div class="rounded-2xl bg-purple-100 p-3">
						<div class="mb-1 text-2xl">💕</div>
						<p class="font-poppins text-xs font-semibold text-purple-600">Endless Love</p>
					</div>
					<div class="rounded-2xl bg-pink-100 p-3">
						<div class="mb-1 text-2xl">🍓</div>
						<p class="font-poppins text-xs font-semibold text-pink-600">Sweet Moments</p>
					</div>
					<div class="rounded-2xl bg-purple-100 p-3">
						<div class="mb-1 text-2xl">🌈</div>
						<p class="font-poppins text-xs font-semibold text-purple-600">Colorful Dreams</p>
					</div>
				</div>
				<p class="font-poppins mt-4 text-center text-base font-bold text-pink-600">
					Happy 20th Birthday, Ayaakuu! 🎉🍓💜
				</p>
			</div>
		</div>

		<!-- Footer - simplified -->
		<div class="relative mt-8 text-center">
			<div class="inline-block">
				{#if !isMobile}
					{#each Array(3) as _, i}
						<span
							class="pointer-events-none absolute animate-ping text-xl opacity-60"
							style="
              left: {(i - 1) * 25}px; 
              animation-delay: {i * 0.5}s;
              animation-duration: 2s;
            "
						>
							💕
						</span>
					{/each}
				{/if}
				<p class="font-poppins pt-6 text-base font-bold text-white drop-shadow-lg">
					With all my love 💕
				</p>
			</div>
		</div>
	</div>
</div>

<!-- Wish Modal -->
{#if showWishModal}
	<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 p-4 backdrop-blur-sm">
		<div
			class="w-full max-w-md transform rounded-3xl bg-white p-6 shadow-2xl {isMobile
				? ''
				: 'animate-pulse'}"
		>
			<div class="mb-4 text-center">
				<div class="mb-3 text-5xl">🕯️</div>
				<h3 class="font-poppins mb-2 text-xl font-bold text-pink-600">Make Your Birthday Wish</h3>
				<p class="font-poppins text-sm text-gray-600">
					Close your eyes and make a special wish! ✨
				</p>
			</div>

			<textarea
				bind:value={wishText}
				placeholder="Write your wish here..."
				class="font-poppins h-24 w-full resize-none rounded-2xl border-2 border-pink-300 p-3 text-sm focus:border-purple-400 focus:outline-none"
			></textarea>

			<div class="mt-4 flex gap-3">
				<button
					on:click={() => (showWishModal = false)}
					class="font-poppins flex-1 rounded-full bg-gray-300 py-3 text-sm font-bold text-gray-700 transition-colors hover:bg-gray-400"
				>
					Cancel
				</button>
				<button
					on:click={submitWish}
					class="font-poppins flex-1 rounded-full bg-gradient-to-r from-pink-500 to-purple-500 py-3 text-sm font-bold text-white transition-all hover:from-pink-600 hover:to-purple-600"
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
		<div class="wish-confirmation w-full max-w-md rounded-3xl bg-white p-6 text-center shadow-2xl">
			<div class="mb-3 text-6xl {isMobile ? '' : 'animate-bounce'}">✨</div>
			<h3 class="font-poppins mb-3 text-xl font-bold text-purple-600">Wish Received!</h3>
			<p class="font-poppins mb-3 text-base text-gray-700">
				🌟 <strong>May all your dreams come true!</strong> 🌟
			</p>
			<p class="font-poppins text-sm text-gray-600">
				Your wish has been sent to the universe on this magical day! 💫
			</p>
			<div class="mt-4 text-3xl">🎂🎈🍓💕✨</div>
		</div>
	</div>
{/if}

<style>
	:global(html) {
		overflow-x: hidden;
	}

	:global(body) {
		overflow-x: hidden;
		/* Enable scrolling for mobile */
		-webkit-overflow-scrolling: touch;
		/* Allow normal touch behaviors */
		touch-action: auto;
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
		animation: gradientShift 4s ease-in-out infinite;
		text-shadow:
			2px 2px 0px #ff1493,
			4px 4px 0px #8a2be2,
			0 0 20px rgba(255, 255, 255, 0.8);
		filter: drop-shadow(2px 2px 6px rgba(0, 0, 0, 0.3));
	}

	@keyframes gradientShift {
		0%,
		100% {
			background-position: 0% 50%;
		}
		50% {
			background-position: 100% 50%;
		}
	}

	/* Photo styling - optimized */
	.photo-container {
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 15px;
	}

	.photo-frame {
		position: relative;
		width: 240px;
		height: 240px;
		border-radius: 50%;
		background: conic-gradient(from 0deg, #ff69b4, #ff1493, #8a2be2, #9370db, #ff6347, #ff69b4);
		padding: 6px;
		animation: rotate 15s linear infinite;
		box-shadow:
			0 0 20px rgba(255, 105, 180, 0.4),
			0 0 40px rgba(138, 43, 226, 0.3);
	}

	.mobile-photo {
		width: 220px;
		height: 220px;
		box-shadow: 0 0 15px rgba(255, 105, 180, 0.4);
	}

	.photo-frame::before {
		content: '';
		position: absolute;
		top: -10px;
		left: -10px;
		right: -10px;
		bottom: -10px;
		border-radius: 50%;
		background: conic-gradient(from 180deg, #ff69b4, #ff1493, #8a2be2, #9370db, #ff6347, #ff69b4);
		animation: rotate 12s linear infinite reverse;
		z-index: -1;
		filter: blur(15px);
		opacity: 0.6;
	}

	.mobile-photo::before {
		animation: none; /* Disable rotation on mobile */
		filter: blur(10px);
	}

	.photo-image {
		width: 100%;
		height: 100%;
		border-radius: 50%;
		object-fit: cover;
		border: 3px solid rgba(255, 255, 255, 0.9);
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
		animation: giftPop 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
	}

	.wish-confirmation {
		animation: wishPop 0.6s cubic-bezier(0.68, -0.55, 0.265, 1.55);
	}

	@keyframes giftPop {
		0% {
			opacity: 0;
			transform: scale(0.5);
		}
		100% {
			opacity: 1;
			transform: scale(1);
		}
	}

	@keyframes wishPop {
		0% {
			opacity: 0;
			transform: scale(0.7);
		}
		100% {
			opacity: 1;
			transform: scale(1);
		}
	}

	/* Mobile optimizations */
	@media (max-width: 768px) {
		.container {
			padding-left: 1rem;
			padding-right: 1rem;
		}

		.hero-text {
			font-size: 3.5rem; /* Increased size for mobile */
			animation-duration: 6s; /* Slower animation */
		}

		/* Disable heavy animations on mobile */
		.mobile-photo {
			animation: none !important;
		}

		.mobile-photo::before {
			animation: none !important;
		}
	}

	@media (max-width: 480px) {
		.hero-text {
			font-size: 3rem; /* Still larger than before */
		}
	}

	/* Performance optimizations */
	@media (prefers-reduced-motion: reduce) {
		* {
			animation-duration: 0.01ms !important;
			animation-iteration-count: 1 !important;
			transition-duration: 0.01ms !important;
		}
	}
</style>
