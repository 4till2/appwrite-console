<script lang="ts">
    import { Empty, Heading } from '$lib/components';
    import { Button } from '$lib/elements/forms';
    import {
        Table,
        TableBody,
        TableCellHead,
        TableCellText,
        TableHeader,
        TableRowLink
    } from '$lib/elements/table';
    import { toLocaleDateTime } from '$lib/helpers/date';
    import { wizard } from '$lib/stores/wizard';
    import type { PageData } from './$types';
    import Wizard from './wizard.svelte';

    export let data: PageData;

    function create() {
        wizard.start(Wizard);
    }
</script>

<div class="common-section u-flex u-gap-12">
    <Heading tag="h3" size="7">API Keys</Heading>
    <span class="u-margin-inline-start-auto">
        <Button on:click={create}>
            <span class="icon-plus" aria-hidden="true" />
            <span class="text">Create API Key</span>
        </Button>
    </span>
</div>

{#if data.keys.total}
    <Table>
        <TableHeader>
            <TableCellHead>Name</TableCellHead>
            <TableCellHead>last accessed</TableCellHead>
            <TableCellHead>expiration date</TableCellHead>
            <TableCellHead>scopes</TableCellHead>
        </TableHeader>
        <TableBody>
            {#each data.keys.keys as key}
                <TableRowLink href={`keys/${key.$id}`}>
                    <TableCellText title="Name">
                        {key.name}
                    </TableCellText>
                    <TableCellText title="Last Accessed">
                        {key.accessedAt ? toLocaleDateTime(key.accessedAt) : 'never'}
                    </TableCellText>
                    <TableCellText title="Expiration Date">
                        {key.expire ? toLocaleDateTime(key.expire) : 'never'}
                    </TableCellText>
                    <TableCellText title="Expiration Date">
                        {key.scopes.length} Scopes
                    </TableCellText>
                </TableRowLink>
            {/each}
        </TableBody>
    </Table>
{:else}
    <Empty single href="https://appwrite.io/docs/keys" target="API Key" on:click={create} />
{/if}
