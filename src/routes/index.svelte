<script lang="ts">
	import beepSoundFile from '$lib/assets/beep-07a.mp3';
	import { onMount } from 'svelte';

	let timeString = { minutes: '25', seconds: '00' };
	let time = 25 * 60;
	let lastInitTime = 25 * 60;
	let mode = 0;
	let title = 'Time to focus';
	let background = '';
	const faviconList = [
		[
			'/countdown-ready-pomodoro.png',
			'/countdown-ready-shortbreak.png',
			'/countdown-ready-longbreak.png'
		],
		[
			'/countdown-done-pomodoro.png',
			'/countdown-done-shortbreak.png',
			'/countdown-done-longbreak.png'
		]
	];
	enum faviconPicker {
		ready,
		done
	}
	let faviconLink = faviconList[faviconPicker.ready][mode];
	let skews = initSkews();
	let continuous = { counter: 0, toggle: false };
	const continuousOrderList = [25 * 60, 5 * 60, 25 * 60, 5 * 60, 25 * 60, 5 * 60, 25 * 60, 15 * 60];
	let continuosPos = 0;

	let countdownIsRun = false;
	let infiniteBeepIsPlaying = false;

	let countdown: NodeJS.Timer;

	let beepSound: HTMLAudioElement;

	onMount(() => (beepSound = new Audio(beepSoundFile)));

	function setCountdown(t: number) {
		resetSkews();
		lastInitTime = t;
		time = t;
	}

	function pickCountdown(t: number) {
		stopCountdown();
		stopInfiniteBeep();
		setCountdown(t);
	}

	function runCountdown() {
		if (time === 0) {
			return;
		}

		countdownIsRun = true;
		beepSound.play();
		countdown = setInterval(() => {
			--time;
			if (!countdownIsRun || time === 0) {
				setInfiniteBeep();
				if (continuous.toggle) {
					if (continuosPos >= continuousOrderList.length - 1) {
						continuosPos = 0;
					} else {
						++continuosPos;
					}
					setCountdown(continuousOrderList[continuosPos]);
				} else {
					stopCountdown();
				}
			}
		}, 1000);
	}

	function stopCountdown() {
		clearInterval(countdown);
		countdownIsRun = false;
	}

	function resetCountdown() {
		stopCountdown();
		stopInfiniteBeep();
		time = lastInitTime;
		resetSkews();
	}

	function setInfiniteBeep() {
		infiniteBeepIsPlaying = true;
		beepSound.addEventListener('ended', playInfiniteBeep);
		beepSound.play();
	}

	function stopInfiniteBeep() {
		beepSound.removeEventListener('ended', playInfiniteBeep);
		infiniteBeepIsPlaying = false;
	}

	function playInfiniteBeep() {
		setTimeout(
			() => {
				if (!infiniteBeepIsPlaying) {
					return;
				}
				if (continuous.toggle) {
					++continuous.counter;
					switch (mode) {
						case 1:
							if (continuous.counter === 5) {
								continuous.counter = 0;
								return;
							}
							break;
						case 2:
							if (continuous.counter === 2) {
								continuous.counter = 0;
								return;
							}
							break;
						case 3:
							if (continuous.counter === 4) {
								continuous.counter = 0;
								return;
							}
							break;
						default:
							if (continuous.counter === 2) {
								continuous.counter = 0;
								return;
							}
					}
				}
				beepSound.currentTime = 0;
				beepSound.play();
			},
			continuous.toggle ? 700 : 1000
		);
	}

	function resetSkews() {
		skews = initSkews();
	}

	function initSkews() {
		return { a: 90, b: 90, c: 90, d: 90 };
	}

	$: {
		if (time < 10 * 60) {
			timeString.minutes = `0${Math.floor(time / 60)}`;
		} else {
			timeString.minutes = `${Math.floor(time / 60)}`;
		}

		if (time % 60 < 10) {
			timeString.seconds = `0${time % 60}`;
		} else {
			timeString.seconds = `${time % 60}`;
		}
	}

	$: {
		if (time >= (3 / 4) * lastInitTime) {
			skews.a = 90 - ((lastInitTime - time) / ((1 / 4) * lastInitTime)) * 90;
		} else {
			skews.a = 0;

			if (time >= (2 / 4) * lastInitTime) {
				skews.b = 90 - (((3 / 4) * lastInitTime - time) / ((1 / 4) * lastInitTime)) * 90;
			} else {
				skews.b = 0;

				if (time >= (1 / 3) * lastInitTime) {
					skews.c = 90 - (((2 / 4) * lastInitTime - time) / ((1 / 4) * lastInitTime)) * 90;
				} else {
					skews.c = 0;

					if (time < (1 / 3) * lastInitTime) {
						skews.d = 90 - (((1 / 4) * lastInitTime - time) / ((1 / 4) * lastInitTime)) * 90;
					} else {
						skews.d = 0;
					}
				}
			}
		}
	}

	$: {
		faviconLink = faviconList[countdownIsRun ? faviconPicker.ready : faviconPicker.done][mode];
		title = timeString.minutes + ':' + timeString.seconds + ' - ';
		switch (mode) {
			case 0:
				title += 'Time to focus!';
				background = 'bg-[#d95550]';
				break;
			case 1:
				title += 'Time for a short break';
				background = 'bg-[#4c9195]';
				break;
			case 2:
				title += 'Time for a long break';
				background = 'bg-[#457ca3]';
				break;
			default:
				title += 'Testing, maybe';
				background = 'bg-black';
		}
	}

	$: switch (continuosPos) {
		case 1:
		case 3:
		case 5:
			mode = 1;
			break;
		case 7:
			mode = 2;
			break;
		default:
			mode = 0;
	}
</script>

<svelte:head>
	<title>{title}</title>
	<link rel="icon" href={faviconLink} />
</svelte:head>

<div
	class="min-w-screen min-h-screen p-8 flex flex-col gap-10 items-center justify-center select-none transition-colors duration-300 {background}"
>
	<div class="relative z-10 flex flex-wrap justify-center gap-3 sm:gap-4">
		<button
			class="py-1.5 px-4 bg-white rounded-lg"
			on:click={() => {
				pickCountdown(25 * 60);
				mode = 0;
			}}
		>
			<p class="text-sm sm:text-base">Pomodoro</p>
			<p class="text-xs sm:text-sm">25 min</p>
		</button>
		<button
			class="py-1.5 px-4 bg-white rounded-lg"
			on:click={() => {
				pickCountdown(5 * 60);
				mode = 1;
			}}
		>
			<p class="text-sm sm:text-base">Short Break</p>
			<p class="text-xs sm:text-sm">5 min</p>
		</button>
		<button
			class="py-1.5 px-4 bg-white rounded-lg"
			on:click={() => {
				pickCountdown(15 * 60);
				mode = 2;
			}}
		>
			<p class="text-sm sm:text-base">Long Break</p>
			<p class="text-xs sm:text-sm">15 min</p>
		</button>
		<!-- <button
			class="py-1.5 px-4 bg-white rounded-lg"
			on:click={() => {
				pickCountdown(2);
				mode = 3;
			}}
		>
			<p class="text-sm sm:text-base">Test</p>
			<p class="text-xs sm:text-sm">2 secs</p>
		</button> -->
	</div>
	<div
		class="w-80 h-80 sm:w-96 sm:h-96 relative rounded-full cursor-pointer overflow-hidden"
		on:click={countdownIsRun || infiniteBeepIsPlaying
			? () => {
					stopCountdown();
					stopInfiniteBeep();
			  }
			: runCountdown}
	>
		<div
			class="relative w-full h-full flex items-center justify-center rounded-full border-8 border-white text-white text-8xl"
		>
			<div class="w-full absolute z-20 flex">
				<span class="flex-1 text-right">{timeString.minutes}</span>
				<span class="flex-none">:</span>
				<span class="flex-1 text-left">{timeString.seconds}</span>
			</div>
		</div>
		<div
			class="w-full h-full sm:w-96 sm:h-96 absolute top-0 z-10 rounded-full border-8 border-white/30"
		/>
		<div
			class="w-screen h-screen absolute bottom-1/2 right-1/2 transition-colors duration-300 {background} origin-bottom-right"
			style="transform: skewY({skews.a}deg)"
		/>
		<div
			class="w-screen h-screen absolute bottom-1/2 right-1/2 transition-colors duration-300 {background} origin-bottom-right"
			style="transform: rotate(-90deg) skewY({skews.b}deg)"
		/>
		<div
			class="w-screen h-screen absolute bottom-1/2 right-1/2 transition-colors duration-300 {background} origin-bottom-right"
			style="transform: rotate(-180deg) skewY({skews.c}deg)"
		/>
		<div
			class="w-screen h-screen absolute bottom-1/2 right-1/2 transition-colors duration-300 {background} origin-bottom-right"
			style="transform: rotate(-270deg) skewY({skews.d}deg)"
		/>
	</div>
	<div class="relative z-10 flex justify-center flex-wrap gap-3 sm:gap-4">
		<button
			class="py-1 sm:py-2 px-2 sm:px-4 bg-white rounded-lg"
			on:click={countdownIsRun || infiniteBeepIsPlaying
				? () => {
						stopCountdown();
						stopInfiniteBeep();
				  }
				: runCountdown}>{countdownIsRun || infiniteBeepIsPlaying ? 'STOP' : 'START'}</button
		>
		<button class="py-1 sm:py-2 px-2 sm:px-4 bg-white rounded-lg" on:click={resetCountdown}
			>RESET</button
		>
		<label
			for="continuous"
			class="py-1 sm:py-2 px-2 sm:px-4 flex gap-1 items-center bg-white rounded-lg whitespace-nowrap"
		>
			<input
				id="continuous"
				type="checkbox"
				bind:checked={continuous.toggle}
				on:change={() => {
					if (continuous.toggle) {
						continuosPos = 0;
						pickCountdown(25 * 60);
						mode = 0;
					}
				}}
			/>
			<span>Continuous</span>
		</label>
	</div>
</div>
