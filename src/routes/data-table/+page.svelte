<script lang="ts">
	import { Input } from '$lib/components/ui/input';
	import { Button } from '$lib/components/ui/button';
	import Loader2 from 'lucide-svelte/icons/loader-2';
    import { Checkbox } from "$lib/components/ui/checkbox";
    import { Label } from "$lib/components/ui/label";
	import * as Table from '$lib/components/ui/table';
	import * as HoverCard from '$lib/components/ui/hover-card';
    import * as Popover from "$lib/components/ui/popover";
	import csvtojson from 'csvtojson';
	import Icon from '@iconify/svelte';

	let loading = false;
	let columnNames: any[] = [];
	let tuples: any[] = [];
    let selectedColumns: any[] = [];

	// load sample data
	const loadData = async () => {
		loading = true;

		await fetch(
			`https://raw.githubusercontent.com/CourseMaterial/DataWrangling/main/flowerdataset.csv`,
			{
				method: 'GET'
			}
		)
			.then(async (response) => {
				let data = await response.text();

				// const filename = 'dataset.csv';
				// const blob = new Blob([data], { type: 'text/csv' });
				// const CSV = new File([blob], filename, { type: blob.type });

				const jsonArray = await csvtojson().fromString(data as string);

				columnNames = Object.keys(jsonArray[0]);
				tuples = jsonArray;

                selectedColumns = []
                columnNames.forEach(() => {
                    selectedColumns.push(true);
                });

			})
			.catch((error) => {
				console.log(error);
				alert('Error loading data');
			})
			.finally(() => {
				loading = false;
			});
	};

	// load user data
	const handleInput = async (e: any) => {
		loading = true;

		const file = e.target.files[0];

		// helper function
		function readFile(file: Blob) {
			return new Promise((resolve, reject) => {
				const reader = new FileReader();
				reader.onload = () => resolve(reader.result);
				reader.onerror = reject;
				reader.readAsText(file);
			});
		}

		// main function
		try {
			const csvData = await readFile(file);
			const jsonArray = await csvtojson().fromString(csvData as string);

			columnNames = Object.keys(jsonArray[0]);
			tuples = jsonArray;

            selectedColumns = []
            columnNames.forEach(() => {
                selectedColumns.push(true);
            });

		} catch (err) {
			console.error(err);
		} finally {
			loading = false;
		}
	};
</script>

<section>
    <!-- <Svrollbar/> -->
	<div class="flex w-full flex-col items-center justify-center">
		<div class="flex max-w-96 flex-row items-center justify-center gap-4">
			<a href="/">
				<Icon icon="bx:arrow-back" style="color: #020817 dark: white" class="size-8" />
			</a>
			<Input type="file" on:input={handleInput} />
			{#if loading}
				<Button variant="destructive" disabled class="w-40">
					<Loader2 class="mr-2 h-4 w-4 animate-spin" />
					Loading
				</Button>
			{:else}
				<Button variant="destructive" class="w-40" on:click={loadData}>Sample data</Button>
			{/if}
		</div>
		<!-- Table -->
		{#if columnNames.length > 0}

            <div class="mt-8">
                <Popover.Root>
                    <Popover.Trigger>
                        <Button variant="secondary" class="w-40">choose columns</Button>
                    </Popover.Trigger>
                    <Popover.Content class="w-fit mt-2">
                        <div class="flex flex-col gap-4 max-h-96 max-w-96 w-full overflow-y-auto overflow-visible">
                            {#each columnNames as column , i (i)}
                                <div class="flex flex-row gap-4">
                                    <Checkbox id ={column} checked={selectedColumns[i]} on:click={() => {selectedColumns[i] = !selectedColumns[i]}} />
                                    <Label for={column} class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70 peer-data-[disabled=true]:cursor-not-allowed peer-data-[disabled=true]:opacity-70" >
                                        {column}
                                    </Label>
                                </div>
                            {/each}
                        </div>
                    </Popover.Content>
                  </Popover.Root>
            </div>

			<div
				class="mt-10 max-h-[70vh] w-full max-w-[85%] overflow-hidden overflow-y-auto rounded-lg border"
			>
				<Table.Root>
					{#if tuples.length == 0}
						<Table.Caption class="my-2">Your list is empty</Table.Caption>
					{:else}
						<Table.Caption class="my-2">End of list</Table.Caption>
					{/if}
					<Table.Header>
						<Table.Row>
							{#each columnNames as column , i (i)}
                                {#if selectedColumns[i]}
                                    <Table.Head class="text-center text-base font-semibold text-black dark:text-white"
                                        >{column}</Table.Head
                                    >
                                {/if}
							{/each}
						</Table.Row>
					</Table.Header>
					<Table.Body>
						{#each tuples as tuple, i (i)}
							<Table.Row>
								{#each columnNames as column , i (i)}
                                    {#if selectedColumns[i]}
                                        <Table.Cell class="text-center font-normal">
                                            <HoverCard.Root>
                                                <HoverCard.Trigger>
                                                    <div class="w-full h-full">
                                                        {tuple[column]}
                                                    </div>
                                                </HoverCard.Trigger>
                                                <HoverCard.Content>
                                                    <div class="max-h-96 max-w-96 w-full overflow-y-auto overflow-visible flex flex-row">
                                                        <div class="flex flex-col basis-1/2">
                                                            <h1 class="font-bold">Index</h1>
                                                            {#each columnNames as column}
                                                                <h1 class="font-semibold text-nowrap">{column}</h1>
                                                            {/each}
                                                        </div>
                                                        <div class="flex flex-col basis-1/2">
                                                            <h1> - {i+1}</h1>
                                                            {#each columnNames as column}
                                                                <h1 class="text-nowrap"> - {tuple[column]}</h1>
                                                            {/each}
                                                        </div>
                                                    </div>
                                                </HoverCard.Content>
                                            </HoverCard.Root>
                                        </Table.Cell>
                                    {/if}
								{/each}
							</Table.Row>
						{/each}
					</Table.Body>
				</Table.Root>
			</div>
		{/if}
	</div>
</section>
