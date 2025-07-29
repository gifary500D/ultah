<script lang="ts">
	import { onMount } from 'svelte';

	let giftOpened = false;
	let showStrawberries = false;
	let strawberries: Array<{ id: number; x: number; delay: number }> = [];
	let currentSection = 0;

	// Sample photos - ganti dengan foto Ayaa yang sebenarnya
	const photos = [
		'https://via.placeholder.com/300x300/FF69B4/FFFFFF?text=Foto+1',
		'https://via.placeholder.com/300x300/9370DB/FFFFFF?text=Foto+2',
		'https://via.placeholder.com/300x300/FF1493/FFFFFF?text=Foto+3',
		'https://via.placeholder.com/300x300/DA70D6/FFFFFF?text=Foto+4'
	];

	const messages = [
		'Selamat Ulang Tahun yang ke-20, Ayaa! 🎉',
		'Semoga hari spesialmu dipenuhi kebahagiaan',
		'Dan strawberry favoritmu! 🍓',
		'Love you always! 💕'
	];

	function openGift() {
		giftOpened = true;
		setTimeout(() => {
			showStrawberries = true;
			createStrawberries();
		}, 500);
	}

	function createStrawberries() {
		strawberries = [];
		for (let i = 0; i < 20; i++) {
			strawberries.push({
				id: i,
				x: Math.random() * 100,
				delay: Math.random() * 2
			});
		}
	}

	function nextSection() {
		if (currentSection < 3) {
			currentSection++;
		}
	}

	function prevSection() {
		if (currentSection > 0) {
			currentSection--;
		}
	}

	function createMoreStrawberries() {
		const newStrawberries = [];
		for (let i = 0; i < 25; i++) {
			newStrawberries.push({
				id: Date.now() + i,
				x: Math.random() * 100,
				delay: Math.random() * 1.5
			});
		}
		strawberries = [...strawberries, ...newStrawberries];

		// Clear strawberries after animation
		setTimeout(() => {
			strawberries = [];
		}, 4000);
	}

	onMount(() => {
		// Auto scroll through sections - diperlambat jadi 8 detik
		const interval = setInterval(() => {
			if (currentSection < 3 && giftOpened) {
				currentSection++;
			}
		}, 8000);

		return () => clearInterval(interval);
	});
</script>

<main
	class="min-h-screen overflow-hidden bg-gradient-to-br from-pink-300 via-purple-300 to-pink-400"
>
	<!-- Floating Hearts Background -->
	<div class="pointer-events-none fixed inset-0">
		{#each Array(15) as _, i}
			<div
				class="absolute animate-bounce text-2xl text-pink-500 opacity-30"
				style="left: {Math.random() * 100}%; top: {Math.random() *
					100}%; animation-delay: {Math.random() * 3}s; animation-duration: {2 +
					Math.random() * 2}s;"
			>
				💕
			</div>
		{/each}
	</div>

	<!-- Strawberry Rain -->
	{#if showStrawberries}
		<div class="pointer-events-none fixed inset-0 z-50">
			{#each strawberries as strawberry}
				<div
					class="animate-strawberry-fall absolute text-4xl"
					style="left: {strawberry.x}%; animation-delay: {strawberry.delay}s; top: -50px;"
				>
					🍓
				</div>
			{/each}
		</div>
	{/if}

	<div class="container mx-auto flex min-h-screen items-center justify-center px-4 py-8">
		{#if !giftOpened}
			<!-- Gift Box Section -->
			<div class="text-center">
				<div class="mb-8 animate-pulse">
					<h1 class="font-pacifico mb-4 text-4xl font-bold text-white drop-shadow-lg md:text-6xl">
						Happy Birthday Ayaa! 🎂
					</h1>
					<p class="font-poppins mb-8 text-xl text-pink-100 md:text-2xl">
						Ada surprise untukmu! ✨
					</p>
				</div>

				<!-- Animated Gift Box -->
				<div
					class="relative inline-block transform cursor-pointer transition-transform duration-300 hover:scale-110"
					on:click={openGift}
				>
					<div
						class="h-32 w-32 animate-bounce rounded-lg bg-gradient-to-br from-pink-400 to-purple-500 shadow-2xl md:h-40 md:w-40"
					>
						<div class="absolute inset-0 rounded-lg border-4 border-yellow-300"></div>
						<div
							class="absolute top-0 left-1/2 h-full w-1 -translate-x-1/2 transform bg-yellow-300"
						></div>
						<div
							class="absolute top-1/2 left-0 h-1 w-full -translate-y-1/2 transform bg-yellow-300"
						></div>
						<div
							class="absolute -top-4 left-1/2 h-8 w-8 -translate-x-1/2 transform rounded-full bg-yellow-300"
						></div>
						<div
							class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 transform text-2xl"
						>
							🎁
						</div>
					</div>
				</div>

				<p class="font-poppins mt-8 animate-pulse text-lg text-pink-100">
					Klik kado untuk membuka! 👆
				</p>
			</div>
		{:else}
			<!-- Main Content After Gift Opened -->
			<div class="mx-auto w-full max-w-4xl">
				<!-- Navigation Dots -->
				<div class="mb-8 flex justify-center space-x-2">
					{#each Array(4) as _, i}
						<button
							class="h-3 w-3 rounded-full transition-all duration-300 {currentSection === i
								? 'scale-125 bg-white'
								: 'bg-pink-200'}"
							on:click={() => (currentSection = i)}
						></button>
					{/each}
				</div>

				<!-- Section 0: Welcome Message -->
				{#if currentSection === 0}
					<div class="animate-fade-in text-center">
						<div
							class="rounded-3xl border border-white/30 bg-white/20 p-8 shadow-2xl backdrop-blur-lg md:p-12"
						>
							<div class="mb-6 animate-bounce text-6xl md:text-8xl">🍓</div>
							<h2
								class="font-pacifico mb-4 text-3xl font-bold text-white drop-shadow-lg md:text-5xl"
							>
								Selamat Ulang Tahun ke-20!
							</h2>
							<h3 class="font-script mb-6 text-2xl text-pink-100 md:text-4xl">Ayaa Tersayang 💕</h3>
							<p class="font-poppins text-lg leading-relaxed text-pink-50 md:text-xl">
								Semoga hari spesialmu dipenuhi dengan kebahagiaan, strawberry yang manis, dan semua
								hal indah yang kamu impikan! ✨
							</p>
						</div>
					</div>
				{/if}

				<!-- Section 1: Birthday Messages -->
				{#if currentSection === 1}
					<div class="animate-fade-in text-center">
						<div
							class="rounded-3xl border border-white/30 bg-white/20 p-8 shadow-2xl backdrop-blur-lg md:p-12"
						>
							<div class="grid gap-6">
								{#each messages as message, i}
									<div
										class="transform transition-all duration-500 hover:scale-105"
										style="animation-delay: {i * 0.3}s"
									>
										<div
											class="rounded-2xl bg-gradient-to-r from-pink-400/50 to-purple-400/50 p-6 backdrop-blur-sm"
										>
											<p class="font-quicksand text-lg font-medium text-white md:text-xl">
												{message}
											</p>
										</div>
									</div>
								{/each}
							</div>
						</div>
					</div>
				{/if}

				<!-- Section 2: Photo Gallery -->
				{#if currentSection === 2}
					<div class="animate-fade-in">
						<div
							class="rounded-3xl border border-white/30 bg-white/20 p-6 shadow-2xl backdrop-blur-lg md:p-8"
						>
							<h3 class="font-pacifico mb-8 text-center text-2xl font-bold text-white md:text-3xl">
								Memories Together 📸
							</h3>
							<div class="grid grid-cols-2 gap-4 md:grid-cols-2 md:gap-6">
								{#each photos as photo, i}
									<div
										class="group relative transform overflow-hidden rounded-2xl shadow-lg transition-all duration-300 hover:scale-105"
										style="animation-delay: {i * 0.2}s"
									>
										<img
											src={photo}
											alt="Memory {i + 1}"
											class="h-48 w-full object-cover md:h-64"
										/>
										<div
											class="absolute inset-0 bg-gradient-to-t from-pink-500/50 to-transparent opacity-0 transition-opacity duration-300 group-hover:opacity-100"
										>
											<div class="absolute bottom-4 left-4">
												<div class="text-2xl">💕</div>
											</div>
										</div>
									</div>
								{/each}
							</div>
						</div>
					</div>
				{/if}

				<!-- Section 3: Final Message -->
				{#if currentSection === 3}
					<div class="animate-fade-in text-center">
						<div
							class="rounded-3xl border border-white/30 bg-white/20 p-8 shadow-2xl backdrop-blur-lg md:p-12"
						>
							<div class="mb-6 animate-pulse text-4xl md:text-6xl">🍓💕🍓</div>
							<h2 class="font-pacifico mb-6 text-2xl font-bold text-white md:text-4xl">
								You're 20 and Amazing! ✨
							</h2>
							<div class="mb-6 rounded-2xl bg-gradient-to-r from-pink-400/30 to-purple-400/30 p-6">
								<p class="font-poppins text-lg leading-relaxed text-pink-50 md:text-xl">
									Terima kasih sudah menjadi bagian terpenting dalam hidupku. Di usia 20 tahun ini,
									semoga kamu semakin bahagia dan semua impianmu terwujud! 🌟
								</p>
							</div>
							<div class="font-script mb-8 animate-bounce text-3xl md:text-4xl">
								I Love You, Ayaa! 💕🍓
							</div>

							<!-- Strawberry Rain Button -->
							<button
								class="group relative transform animate-pulse rounded-full bg-gradient-to-r from-pink-500 to-purple-500 px-8 py-4 text-lg font-bold text-white shadow-2xl transition-all duration-300 hover:scale-105 hover:from-pink-600 hover:to-purple-600"
								on:click={createMoreStrawberries}
							>
								<span class="flex items-center gap-2"> 🍓 Make it Rain Strawberries! 🍓 </span>
								<div
									class="absolute inset-0 rounded-full bg-white/20 opacity-0 transition-opacity duration-300 group-hover:opacity-100"
								></div>
							</button>
						</div>
					</div>
				{/if}

				<!-- Navigation Buttons -->
				<div class="mt-8 flex justify-between">
					<button
						class="font-poppins rounded-full bg-white/20 px-6 py-3 font-medium text-white shadow-lg backdrop-blur-lg transition-all duration-300 hover:bg-white/30 disabled:opacity-50"
						on:click={prevSection}
						disabled={currentSection === 0}
					>
						← Previous
					</button>

					<button
						class="font-poppins rounded-full bg-white/20 px-6 py-3 font-medium text-white shadow-lg backdrop-blur-lg transition-all duration-300 hover:bg-white/30 disabled:opacity-50"
						on:click={nextSection}
						disabled={currentSection === 3}
					>
						Next →
					</button>
				</div>
			</div>
		{/if}
	</div>
</main>

<style>
	@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@400;700&family=Pacifico&family=Quicksand:wght@300;400;500;600;700&family=Poppins:wght@300;400;500;600;700;800&display=swap');

	:global(body) {
		font-family: 'Quicksand', sans-serif;
	}

	.font-script {
		font-family: 'Dancing Script', cursive;
	}

	.font-pacifico {
		font-family: 'Pacifico', cursive;
	}

	.font-poppins {
		font-family: 'Poppins', sans-serif;
	}

	.animate-fade-in {
		animation: fadeIn 1.2s ease-in-out;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: translateY(30px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* Custom strawberry falling animation */
	@keyframes strawberryFall {
		0% {
			transform: translateY(-100vh) rotate(0deg);
			opacity: 1;
		}
		100% {
			transform: translateY(100vh) rotate(720deg);
			opacity: 0.3;
		}
	}

	/* Enhanced strawberry animation */
	.animate-strawberry-fall {
		animation: strawberryFall 4s linear forwards;
	}

	.animate-bounce {
		animation: bounce 2.5s infinite;
	}

	@keyframes bounce {
		0%,
		20%,
		53%,
		80%,
		100% {
			transform: translate3d(0, 0, 0);
		}
		40%,
		43% {
			transform: translate3d(0, -30px, 0);
		}
		70% {
			transform: translate3d(0, -15px, 0);
		}
		90% {
			transform: translate3d(0, -4px, 0);
		}
	}

	/* Floating animation for hearts */
	@keyframes float {
		0%,
		100% {
			transform: translateY(0px) rotate(0deg);
		}
		50% {
			transform: translateY(-20px) rotate(10deg);
		}
	}

	.animate-float {
		animation: float 3s ease-in-out infinite;
	}
</style>
