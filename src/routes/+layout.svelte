<script>
	import '@skeletonlabs/skeleton/themes/theme-skeleton.css';
	import '@skeletonlabs/skeleton/styles/all.css';
	import '../app.postcss';
	import { AppShell, AppBar } from '@skeletonlabs/skeleton';

</script>

<!-- App Shell -->
<AppShell>
	<svelte:fragment slot="header">
		<!-- App Bar -->
		<AppBar>
			<svelte:fragment slot="lead">
				<strong class="text-xl uppercase">WordGuesser</strong>
			</svelte:fragment>
			<svelte:fragment slot="trail">
				<button
					id="give-up"
					class="btn btn-sm variant-ghost-surface"
				>
					Give Up
				</button>
				<button
					class="btn btn-sm variant-ghost-surface"
					on:click={() => window.location.reload()}
				>
					Reset
				</button>
			</svelte:fragment>
		</AppBar>
	</svelte:fragment>
	<!-- Page Route Content -->
	<slot />
</AppShell>
