<script lang="ts">
    import { Button, InputSearch } from '$lib/elements/forms';
    import { createEventDispatcher } from 'svelte';
    import { sdkForProject } from '$lib/stores/sdk';
    import { Query, type Models } from '@aw-labs/appwrite-console';
    import { AvatarInitials, EmptySearch, Modal, PaginationInline } from '..';
    import type { Writable } from 'svelte/store';
    import type { Permission } from './permissions.svelte';

    export let show: boolean;
    export let groups: Writable<Map<string, Permission>>;

    const dispatch = createEventDispatcher();

    let search = '';
    let offset = 0;
    let results: Models.TeamList;
    let selected: Set<string> = new Set();
    let hasSelection = false;

    function reset() {
        offset = 0;
        search = '';
        selected.clear();
    }

    function create() {
        dispatch('create', Array.from(selected));
        reset();
    }

    async function request() {
        if (!show) return;
        results = await sdkForProject.teams.list([Query.limit(5), Query.offset(offset)], search);
    }

    function onSelection(event: Event, role: string) {
        const { checked } = event.currentTarget as HTMLInputElement;

        if (checked) {
            selected.add(role);
        } else {
            selected.delete(role);
        }

        hasSelection = selected.size > 0;
    }

    $: if (show) {
        request();
    }
    $: if (offset !== null) {
        request();
    }
    $: if (search !== null) {
        offset = 0;
        request();
    }
</script>

<Modal bind:show on:submit={create} on:close={reset} size="big">
    <svelte:fragment slot="header">Select teams</svelte:fragment>
    <p class="text">
        Grant access to any member of a specific team. To grant access to team members with specific
        roles, you will need to set a <button
            type="button"
            class="link"
            on:click={() => dispatch('custom')}>custom permission</button
        >.
    </p>
    <InputSearch
        autofocus
        disabled={!results?.teams?.length && !search}
        placeholder="Search by name or ID"
        bind:value={search} />
    {#if results?.teams?.length}
        <div class="table-wrapper">
            <table class="table is-table-layout-auto is-remove-outer-styles">
                <tbody class="table-tbody">
                    {#each results.teams as team (team.$id)}
                        {@const role = `team:${team.$id}`}
                        {@const exists = $groups.has(role)}
                        <tr class="table-row">
                            <td class="table-col" data-title="Enabled" style="--p-col-width:40">
                                <input
                                    id={team.$id}
                                    type="checkbox"
                                    class="icon-check"
                                    aria-label="Create"
                                    checked={exists || selected.has(role)}
                                    disabled={exists}
                                    on:change={(event) => onSelection(event, role)} />
                            </td>
                            <td class="table-col" data-title="Team">
                                <label class="u-flex u-cross-center u-gap-8" for={team.$id}>
                                    <AvatarInitials size={32} name={team.name} />
                                    <div class="u-line-height-1-5">
                                        <div class="body-text-2">{team.name}</div>
                                        <div class="u-x-small">{team.$id}</div>
                                    </div>
                                </label>
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        </div>
        <div class="u-flex u-margin-block-start-32 u-main-space-between">
            <p class="text">Total results: {results?.total}</p>
            <PaginationInline limit={5} bind:offset sum={results?.total} hidePages />
        </div>
    {:else if search}
        <EmptySearch hidePages>
            <div class="common-section">
                <div class="u-text-center common-section">
                    <b class="body-text-2">Sorry we couldn't find "{search}"</b>
                    <p>There are no teams that match your search.</p>
                </div>
                <div class="u-flex u-gap-16 common-section u-main-center">
                    <Button external href="https://appwrite.io/docs/client/teams" text
                        >Documentation</Button>
                    <Button secondary on:click={() => (search = '')}>Clear search</Button>
                </div>
            </div>
        </EmptySearch>
    {:else}
        <EmptySearch hidePages>
            <div class="common-section">
                <div class="u-text-center common-section">
                    <p class="text u-line-height-1-5">
                        You have no teams. Create a team to see them here.
                    </p>
                    <p class="text u-line-height-1-5">
                        Need a hand? Check out our <a
                            href="https://appwrite.io/docs/client/teams"
                            target="_blank"
                            rel="noopener noreferrer">
                            documentation</a
                        >.
                    </p>
                </div>
            </div>
        </EmptySearch>
    {/if}

    <svelte:fragment slot="footer">
        <Button submit disabled={!hasSelection}>Create</Button>
    </svelte:fragment>
</Modal>
