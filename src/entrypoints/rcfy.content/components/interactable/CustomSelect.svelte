<script context="module" lang="ts">
	export interface SelectOption {
		id: any;
		title: string;
	}
</script>

<script lang="ts">
	import { onMount } from 'svelte';

	export let options: SelectOption[];
	export let selectedOption: SelectOption;
	export let fullWidth: boolean = false;
	export let callback: (o: SelectOption) => void;

	let entryWidth: number;
	let buttonWidth: number = 0;
	let mounted = false;
	let isActive = false;

	function setSelectedOption(option: any) {
		selectedOption = option;
		isActive = false;
		if (callback) {
			callback(option);
		}
	}

	function clickOutside(node: HTMLElement) {
		const handleClick = (e: Event) => {
			const target = e.target as HTMLElement;
			if (node && !node.contains(target) && !e.defaultPrevented) {
				isActive = false;
			}
		};
		document.addEventListener('click', handleClick, true);
		return {
			destroy() {
				document.removeEventListener('click', handleClick, true);
			},
		};
	}

	function setSite(width: number, ready: boolean) {
		if (buttonWidth !== 0 || !ready) return;
		buttonWidth = width - 9;
	}

	$: setSite(entryWidth, mounted);

	onMount(() => {
		mounted = true;
	});
</script>

<div
	class="select"
	class:fill-available={fullWidth}
	class:fit={!fullWidth}
	use:clickOutside
>
	<button
		class="selector"
		class:active={isActive}
		on:click={() => {
			isActive = !isActive;
		}}
	>
		<span style="width: {buttonWidth}px;">{selectedOption.title}</span>
		<div class="arrow">╲╱</div>
	</button>
	<div
		class="options"
		class:hidden={!isActive}
		class:fill-available={buttonWidth}
		bind:clientWidth={entryWidth}
	>
		{#each options as o (o.id)}
			<button on:click={() => setSelectedOption(o)}>
				{o.title}
			</button>
		{/each}
	</div>
</div>

<style lang="postcss">
	button {
		@apply plain-button;
	}

	.select {
		@apply relative text-standard cursor-pointer;

		* {
			@apply text-primary overflow-hidden whitespace-pre text-ellipsis;
		}
	}

	.fit {
		@apply max-w-fit;
	}

	.selector {
		@apply box-content flex items-center justify-between p-[5px] gap-[5px] rounded-small bg-interactable border-[1px] border-interactable fill-available border-solid;

		&.active {
			@apply rounded-b-none;
		}
	}

	.arrow {
		@apply flex-shrink-0 text-arrow;
	}

	.options {
		@apply bg-select absolute top-full z-10 border-interactable border-solid box-content border-t-2 flex flex-col rounded-b-small max-h-[40vh] overflow-auto;

		button {
			@apply bg-option p-[5px] border-t-0 flex-shrink-0 last:border-b-0 hover:bg-hover;
		}

		&.hidden {
			@apply invisible;
		}
	}
</style>
