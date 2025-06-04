<script lang="ts">
	import { Button, buttonVariants } from '$lib/components/ui/button';
	import { data } from '../state.svelte';

	let low = $state(0);
	let high = $state(1000);
	data.middle = 500;

	const higher = () => {
		low = data.middle;
		data.middle = Math.floor((high + low) / 2);
	};

	const lower = () => {
		high = data.middle;
		data.middle = Math.floor((high + low) / 2);
	};
</script>

<section class="flex min-h-screen flex-col items-center justify-center gap-3">
	<h1>Please attempt to select the middle part of your phone number.</h1>
	<div class="flex flex-row items-center gap-6">
		<Button onclick={lower}>Lower</Button>
		<p>{data.middle}</p>
		<Button onclick={higher}>Higher</Button>
	</div>
	<a href="/" class={buttonVariants({ class: 'fixed bottom-5' })}>Back</a>
</section>
