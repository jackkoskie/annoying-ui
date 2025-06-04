<script lang="ts">
	import { buttonVariants, Button } from '$lib/components/ui/button';
	import * as Select from '$lib/components/ui/select';
	import { data } from './state.svelte';
	import codes from '$lib/countryCodes.json';
	import { onMount } from 'svelte';
	let interval: ReturnType<typeof setInterval>;

	onMount(() => {
		data.countryCode = codes[Math.floor(Math.random() * codes.length)];

		interval = setInterval(() => {
			codes.sort(() => Math.random() - 0.5);
		}, 500);

		return () => {
			clearInterval(interval);
		};
	});
</script>

<main class="flex min-h-screen flex-col items-center justify-center gap-3">
	<h1>Please select a section of your phone number to edit</h1>
	<div class="flex flex-row items-center gap-3">
		<Select.Root type="single" bind:value={data.countryCode}>
			<Select.Trigger class="">+{data.countryCode}</Select.Trigger>
			<Select.Content>
				{#each codes as code}
					<Select.Item value={code}>+{code}</Select.Item>
				{/each}
			</Select.Content>
		</Select.Root>

		<a href="/area" class={buttonVariants({ variant: 'default' })}
			>{data.areaCode.toString().padStart(3, '0')}</a
		>
		<a href="/middle" class={buttonVariants({ variant: 'default' })}
			>{data.middle.toString().padStart(3, '0')}</a
		>
		<a href="/last" class={buttonVariants({ variant: 'default' })}
			>{data.last.toString().padStart(4, '0')}</a
		>
	</div>
	<Button
		variant="ghost"
		class="fixed bottom-5 cursor-pointer"
		onclick={() => {
			// Reset the state
			data.areaCode = '000';
			data.middle = 0;
			data.last = 0;

			console.log('State reset');
		}}>Reset</Button
	>
</main>
