<script lang="ts">
    import { Container } from '$lib/layout';
    import { Card, DropListLink, DropTabs, Heading } from '$lib/components';
    import { last } from '$lib/layout/usage.svelte';
    import { BarChart } from '$lib/charts';
    import { page } from '$app/stores';
    import type { PageData } from './$types';

    export let data: PageData;

    $: count = data.count;
    $: errors = data.errors;
</script>

<Container>
    <div class="u-flex u-main-space-between common-section">
        <Heading tag="h2" size="5">Functions</Heading>
        <DropTabs>
            <DropListLink
                href={`/console/project-${data.project.$id}/functions/function-${data.function.$id}/usage/24h`}
                disabled={($page.params.period ?? '24h') === '24h'}>
                24h
            </DropListLink>
            <DropListLink
                href={`/console/project-${data.project.$id}/functions/function-${data.function.$id}/usage/30d`}
                disabled={$page.params.period === '30d'}>
                30d
            </DropListLink>
            <DropListLink
                href={`/console/project-${data.project.$id}/functions/function-${data.function.$id}/usage/90d`}
                disabled={$page.params.period === '90d'}>
                90d
            </DropListLink>
        </DropTabs>
    </div>
    {#if count}
        <Card>
            <Heading tag="h6" size="6">{last(count).value}</Heading>
            <p>Executions</p>
            <div class="u-margin-block-start-16" />
            <BarChart
                series={[
                    {
                        name: 'Count of function executions over time',
                        data: [...count.map((e) => [e.date, e.value])]
                    }
                ]} />
        </Card>
    {/if}
    {#if errors}
        <Card>
            <Heading tag="h6" size="6">{last(errors).value}</Heading>
            <p>Errors</p>
            <div class="u-margin-block-start-16" />
            <BarChart
                series={[
                    {
                        name: 'Count of function errors over time',
                        data: [...errors.map((e) => [e.date, e.value])]
                    }
                ]} />
        </Card>
    {/if}
</Container>
