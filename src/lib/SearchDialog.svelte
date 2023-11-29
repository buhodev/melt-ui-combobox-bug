<script lang="ts">
	import { createDialog, createCombobox, melt } from '@melt-ui/svelte';

	const {
		elements: { trigger, overlay, content, /*title, description,*/ close, portalled },
		states: { open }
	} = createDialog({
		forceVisible: true,
		preventScroll: true,
		closeOnOutsideClick: true,
		closeOnEscape: true
	});

	const {
		elements: { menu: menuCombobox, input: inputCombobox, option: optionCombobox, label: labelCombobox },
		states: { open: openCombobox, inputValue: inputValueCombobox, touchedInput: touchedInputCombobox, selected: selectedCombobox }
	} = createCombobox({
		forceVisible: true,
		portal: '#scrollablePanel',
		positioning: {
			placement: 'bottom',
			overflowPadding: 0,
			sameWidth: false,
			fitViewport: true,
			flip: false,
			gutter: 10
		},
		closeOnEscape: true,
		closeOnOutsideClick: false
	});

	type Manga = {
		author: string;
		title: string;
		disabled: boolean;
	};

	let mangas: Manga[] = [
		{
			author: 'Kentaro Miura',
			title: 'Berserk',
			disabled: false
		},
		{
			author: 'ONE',
			title: 'Mob Psycho 100',
			disabled: false
		},
		{
			author: 'Hajime Isayama',
			title: 'Attack on Titan',
			disabled: false
		},
		{
			author: 'Junji Ito',
			title: 'Uzumaki',
			disabled: false
		},
		{
			author: 'Yomi Sarachi',
			title: 'Steins Gate',
			disabled: false
		},
		{
			author: 'Tite Kubo',
			title: 'Bleach',
			disabled: false
		},
		{
			author: 'Masashi Kishimoto',
			title: 'Naruto',
			disabled: true
		},
		{
			author: 'Katsura Hoshino',
			title: 'D.Gray Man',
			disabled: false
		},
		{
			author: 'Tsugumi Ohba',
			title: 'Death Note',
			disabled: false
		},
		{
			author: 'Hiromu Arakawa',
			title: 'Fullmetal Alchemist',
			disabled: false
		}
	];

	let debounceTimer: ReturnType<typeof setTimeout>;

	const debounce = (callback: () => void) => {
		clearTimeout(debounceTimer);
		debounceTimer = setTimeout(callback, 450);
	};

	$: if (!$open) {
		$inputValueCombobox = $selectedCombobox?.label ?? '';
	}

	let filteredMangas = mangas;
	$: {
		if ($touchedInputCombobox) {
			debounce(() => {
				filteredMangas = mangas.filter(({ title, author }) => {
					const normalizedInput = $inputValueCombobox.toLowerCase();
					return title.toLowerCase().includes(normalizedInput) || author.toLowerCase().includes(normalizedInput);
				});
			});
		} else {
			filteredMangas = mangas;
		}
	}
</script>

<button use:melt={$trigger} class:hidden={$open} class="relative rounded bg-blue-500 items-center justify-center px-2 text-white">open</button>

<button use:melt={$close} class:hidden={!$open} class="relative rounded bg-blue-500 items-center justify-center px-2 text-white">close</button>

<div use:melt={$portalled} class="isolate flex h-full w-full flex-col items-center justify-end">
	{#if $open}
		<div use:melt={$overlay} class="fixed inset-0 bg-slate-500/25 backdrop-blur-sm backdrop-filter dark:bg-slate-900/50" />

		<div
			class="fixed bottom-0 w-full max-w-[475px] rounded-t-3xl border bg-slate-50 pb-6 shadow-lg dark:border-slate-700 dark:bg-slate-800 dark:shadow-highlight"
			use:melt={$content}
			style="height: 100dvh;"
		>
			<form
				class="group absolute inset-x-0 top-4 mx-4 h-8 w-auto overflow-hidden rounded-lg bg-slate-200/50 text-slate-400 focus-within:bg-slate-100 disabled:opacity-25 dark:bg-slate-700 dark:text-slate-200 dark:focus-within:bg-slate-700 dark:focus-within:text-slate-100 sm:justify-start"
			>
				<label for="search-mobile" class="sr-only">Search</label>

				<input
					use:melt={$inputCombobox}
					autocomplete="off"
					id="search-mobile"
					type="text"
					class="h-8 w-full border-0 bg-transparent py-1 text-gray-900 placeholder:text-gray-400 px-2 dark:text-white dark:placeholder:text-gray-500"
					placeholder="Search..."
				/>
			</form>
			<div id="scrollablePanel" class:overflow-y-auto={!$openCombobox} class="relative mt-16 h-full px-6 pb-20 pt-4">
				<div class="pb-16">
					{#if $openCombobox}
						<ul class="z-10 flex max-h-[300px] flex-col overflow-hidden rounded-lg" use:melt={$menuCombobox}>
							<div class="flex max-h-full flex-col gap-0 overflow-y-auto bg-white px-2 py-2 text-black">
								{#each filteredMangas as book, index (index)}
									<li
										use:melt={$optionCombobox({
											value: book,
											label: book.title,
											disabled: book.disabled
										})}
										class="relative cursor-pointer scroll-my-2 rounded-md py-2 pl-4 pr-4 data-[highlighted]:bg-blue-200 data-[highlighted]:text-blue-900 data-[disabled]:opacity-50"
									>
										<div class="pl-4">
											<span class="font-medium">{book.title}</span>
											<span class="block text-sm opacity-75">{book.author}</span>
										</div>
									</li>
								{:else}
									<li
										class="relative cursor-pointer rounded-md py-1 pl-8 pr-4 data-[highlighted]:bg-blue-100 data-[highlighted]:text-blue-700"
									>
										No results found
									</li>
								{/each}
							</div>
						</ul>
					{/if}
				</div>
			</div>
		</div>
	{/if}
</div>
