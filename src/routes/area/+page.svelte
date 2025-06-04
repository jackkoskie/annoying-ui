<script lang="ts">
	import { Button, buttonVariants } from '$lib/components/ui/button';
	import type { PageData } from './$types';

	import { data } from '../state.svelte';

	let active = $state([false, false, false]);

	setInterval(() => {
		if (active[0]) {
			data.areaCode = ((Number(data.areaCode[0]) + 1) % 10).toString() + data.areaCode.slice(1);
		}
		if (active[1]) {
			data.areaCode =
				data.areaCode[0] +
				((Number(data.areaCode[1]) + 1) % 10).toString() +
				data.areaCode.slice(2);
		}
		if (active[2]) {
			data.areaCode = data.areaCode.slice(0, 2) + ((Number(data.areaCode[2]) + 1) % 10).toString();
		}
	}, 50);
</script>

<section class="flex min-h-screen flex-col items-center justify-center gap-3">
	<h1>Please attempt to select your area code</h1>
	<div class="flex flex-row gap-3">
		<Button
			onclick={() => {
				active[0] = !active[0];
			}}>{data.areaCode[0]}</Button
		>
		<Button
			onclick={() => {
				active[1] = !active[1];
			}}>{data.areaCode[1]}</Button
		>
		<Button
			onclick={() => {
				active[2] = !active[2];
			}}>{data.areaCode[2]}</Button
		>
	</div>
	<a href="/" class={buttonVariants({ class: 'fixed bottom-5' })}>Back</a>
</section>
