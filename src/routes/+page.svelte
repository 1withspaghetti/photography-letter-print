<script lang="ts">
	import { PersistedState } from 'runed';

	const fonts = [
		{
			label: 'Audiowide',
			value: 'Audiowide'
		},
		{
			label: 'Press Start 2P',
			value: 'PressStart2P'
		},
		{
			label: 'Seven Segment',
			value: 'SevenSegment'
		},
		{
			label: 'Special Elite',
			value: 'SpecialElite'
		},
		{
			label: 'Story Script',
			value: 'StoryScript'
		}
	];

	const text = new PersistedState('text', '');
	const font = new PersistedState('font', 'SevenSegment');
	const fontSize = new PersistedState('fontSize', 256);
	const color = new PersistedState('color', '#ffffff');
	const timeOn = new PersistedState('timeOn', 10);
	const timeOff = new PersistedState('timeOff', 25);
	const padStart = new PersistedState('padStart', 500);
	const padEnd = new PersistedState('padEnd', 500);

	const totalTime = $derived(
		padStart.current + (timeOn.current + timeOff.current) * text.current.length + padEnd.current
	);

	function wait(ms: number): Promise<void> {
		return new Promise((res) => {
			setTimeout(res, ms);
		});
	}

	let running = $state(false);
	let currentLetter = $state('');

    function fullscreen() {
        if (document.fullscreenElement == null) document.documentElement.requestFullscreen();
        else document.exitFullscreen();
    }

	async function run() {
		currentLetter = '';
		running = true;
		await wait(padStart.current);
		for (let i = 0; i < text.current.length; i++) {
			currentLetter = text.current.charAt(i);
			await wait(timeOn.current);
			currentLetter = '';
			await wait(timeOff.current);
		}
		await wait(padEnd.current);
		running = false;
	}
</script>

<main>
	<div class="mx-auto flex max-w-md flex-col gap-4 p-8">
		<label class="block">
			<span class="text-sm text-gray-700">Text</span>
			<input
				type="text"
				bind:value={text.current}
				class="mt-1 form-input block w-full"
				style:font-family={font.current}
				placeholder="Hello, World"
			/>
		</label>
		<div class="flex gap-4">
			<label class="block w-full">
				<span class="text-sm text-gray-700">Font</span>
				<select class="mt-1 block w-full" bind:value={font.current}>
					{#each fonts as f (f.value)}
						<option value={f.value} style:font-family={f.value}>{f.label}</option>
					{/each}
				</select>
			</label>
			<label class="block w-32">
				<span class="text-sm text-gray-700">Font Size</span>
				<input type="number" bind:value={fontSize.current} class="mt-1 form-input block w-full" />
			</label>
			<label class="block w-32">
				<span class="text-sm text-gray-700">Color</span>
				<input
					type="color"
					bind:value={color.current}
					class="mt-1 form-input block h-10.5 w-full"
				/>
			</label>
		</div>
		<div class="flex gap-4">
			<label class="block">
				<span class="text-sm text-gray-700">Per Letter (ms)</span>
				<input type="number" bind:value={timeOn.current} class="mt-1 form-input block w-full" />
			</label>
			<label class="block">
				<span class="text-sm text-gray-700">Between Letters (ms)</span>
				<input type="number" bind:value={timeOff.current} class="mt-1 form-input block w-full" />
			</label>
		</div>
		<div class="flex gap-4">
			<label class="block">
				<span class="text-sm text-gray-700">Pad Start (ms)</span>
				<input type="number" bind:value={padStart.current} class="mt-1 form-input block w-full" />
			</label>
			<label class="block">
				<span class="text-sm text-gray-700">Pad End (ms)</span>
				<input type="number" bind:value={padEnd.current} class="mt-1 form-input block w-full" />
			</label>
		</div>
        <button
			class="bg-gray-300 px-4 py-2 font-bold transition hover:bg-gray-400"
			onclick={fullscreen}>Fullscreen</button
		>
		<button
			class="bg-blue-400 px-4 py-6 font-bold text-white transition hover:bg-blue-500"
			onclick={run}>Run</button
		>
		<div class="text-center text-sm">
			Total Time: {totalTime}ms
		</div>
	</div>
	<div class="mt-8 flex h-screen w-screen items-center justify-center bg-black px-8 text-white">
		<div
			style:font-family={font.current}
			style:font-size="{fontSize.current}px"
			style:color={color.current}
		>
			{text.current.charAt(0) || 'A'}
		</div>
	</div>
</main>

{#if running}
	<div
		class="fixed top-0 left-0 flex h-full w-full items-center justify-center bg-black text-white"
	>
		<div
			style:font-family={font.current}
			style:font-size="{fontSize.current}px"
			style:color={color.current}
		>
			{currentLetter}
		</div>
	</div>
{/if}
