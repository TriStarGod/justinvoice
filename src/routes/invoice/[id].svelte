<script lang="ts" context="module">
	/** @type {import('./__types/[slug]').Load} */
	export async function load({ params }) {
		return {
			status: 200,
			props: {
				id: +params.id
			}
		};
	}
</script>

<script lang="ts">
	import { goto } from '$app/navigation';
	import Item from '$components/Item.svelte';

	import { addressbook, invoices, selectedInvoice } from '$stores/app';
	import { addNewItemModal } from '$stores/modals';
	import { exportToJsonFile } from '$utils';

	import { removeInvoice, type Invoice } from '$utils/invoice';
	import { onMount } from 'svelte';

	/// STATE ///
	export let id: Invoice['id'];
	$: invoice = $invoices.find((i) => i.id === id);
	$: $selectedInvoice = invoice;
	let loading: boolean = true;
	$: dateString = invoice
		? new Date(invoice.date_of_issue).toLocaleDateString(undefined, {
				dateStyle: 'full'
		  })
		: undefined;

	$: recipient = $addressbook.find((address) => address.id == invoice?.recipientID);
	$: sender = $addressbook.find((address) => address.id == invoice?.senderID);

	/// METHODS ///
	async function deleteInvoice() {
		await goto('/');
		removeInvoice(id);
	}

	function addItem() {
		$addNewItemModal = true;
	}

	function exportInvoice() {
		if (invoice) {
			exportToJsonFile(invoice, `invoice-${invoice.id}.json`);
		}
	}

	function onPaidChange() {
		$invoices = $invoices;
	}

	function print() {
		if (!invoice) return;
		if (invoice.items.length === 0) {
			alert('add atleast on item');
			return;
		}
		window.open(`/pdf/${id}`, '_blank');
	}

	/// LIFECYCLE HOOKS ///
	onMount(() => {
		loading = false;
	});
</script>

<div class="p-5">
	{#if loading}
		<p>Loading</p>
	{:else if invoice}
		<div class="card w-full bg-base-100 shadow-xl card-compact border border-gray-600">
			<div class="card-body">
				<h2 class="card-title">
					<span>{invoice.title}</span>
					<div class="badge badge-outline">ID: {invoice.id}</div>
					<span class="grow" />
					<div class="form-control">
						<label class="label cursor-pointer gap-3">
							<input
								type="checkbox"
								class="toggle toggle-sm"
								bind:checked={invoice.paid}
								on:change={onPaidChange}
							/>
							<span class="label-text">Paid</span>
						</label>
					</div>
				</h2>
				<p class="flex gap-1 items-center">
					<span>Date of Issue:</span><span class="font-bold">
						{dateString}
					</span>
				</p>
				<p class="flex gap-1 items-center">
					<span>Sender:</span><span class="font-bold">
						{sender.name}, {sender.address}
					</span>
				</p>
				<p class="flex gap-1 items-center">
					<span>Recipient:</span><span class="font-bold">
						{recipient.name}, {recipient.address}
					</span>
				</p>
				<div class="card-actions items-center">
					<span class="grow" />
					<button class="btn btn-outline btn-sm" on:click={print}>Print</button>
					<button class="btn btn-outline btn-sm" on:click={exportInvoice}>Export</button>
					<button class="btn btn-sm btn-primary" on:click={addItem}>Add Item</button>
					<button class="btn btn-sm btn-error" on:click={deleteInvoice}>Delete</button>
				</div>
			</div>
		</div>
		<div class="py-5 grid gap-5" id="item-grid">
			{#each invoice.items as item}
				<Item {...item} />
			{/each}
		</div>
	{:else}
		<p class="text-2xl">
			Invoice with id {id} not found :(
		</p>
	{/if}
</div>

<style>
	#item-grid {
		grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
	}
</style>
