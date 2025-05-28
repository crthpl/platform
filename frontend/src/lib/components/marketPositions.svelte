<script lang="ts">
	import { accountName, serverState } from '$lib/api.svelte';
	import FlexNumber from '$lib/components/flexNumber.svelte';
	import * as Table from '$lib/components/ui/table';
	import type { websocket_api } from 'schema-js';
	import { cn } from '$lib/utils';

	// incoming trades
	export let trades: websocket_api.ITrade[] = [];

	// compute net positions whenever trades changes
	$: positions = (() => {
		const map = new Map<number, number>();
		for (const t of trades) {
			const size = t.size ?? 0;
			if (t.buyerId != null) {
				map.set(t.buyerId, (map.get(t.buyerId) ?? 0) + size);
			}
			if (t.sellerId != null) {
				map.set(t.sellerId, (map.get(t.sellerId) ?? 0) - size);
			}
		}
		return Array.from(map.entries())
			.map(([accountId, position]) => ({ accountId, position }))
			.sort((a, b) => b.position - a.position);
	})();

	const getShortUserName = (id: number | null | undefined) => accountName(id).split(' ')[0];
</script>

<Table.Root>
	<Table.Header>
		<Table.Row class="market-positions-cols grid h-full justify-center">
			<Table.Head class="flex items-center justify-center text-center">Trader</Table.Head>
			<Table.Head class="flex items-center justify-center text-center">Position</Table.Head>
		</Table.Row>
	</Table.Header>

	<Table.Body>
		{#each positions as { accountId, position } (accountId)}
			<Table.Row class="market-positions-cols grid justify-center">
				<Table.Cell
					class={cn(
						'flex items-center justify-center truncate px-2 py-1 text-center',
						accountId === serverState.actingAs ? 'border-2 border-primary' : ''
					)}
				>
					{getShortUserName(accountId)}
				</Table.Cell>
				<Table.Cell class="flex items-center justify-center px-2 py-1 text-center">
					<FlexNumber value={position.toFixed(1)} />
				</Table.Cell>
			</Table.Row>
		{/each}
	</Table.Body>
</Table.Root>

<style>
	:global(.market-positions-cols) {
		/* adjust min/max widths to taste */
		grid-template-columns: minmax(5rem, 1fr) minmax(4rem, 1fr);
	}
</style>
