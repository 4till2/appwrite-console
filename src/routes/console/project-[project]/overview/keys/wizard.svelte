<script lang="ts">
    import { Wizard } from '$lib/layout';
    import { beforeNavigate, goto, invalidate } from '$app/navigation';
    import { wizard } from '$lib/stores/wizard';
    import type { WizardStepsType } from '$lib/layout/wizard.svelte';
    import Step1 from './wizard/step1.svelte';
    import Step2 from './wizard/step2.svelte';
    import { key } from './wizard/store';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { page } from '$app/stores';
    import { addNotification } from '$lib/stores/notifications';
    import { onDestroy } from 'svelte';
    import { onboarding } from '../../store';
    import { Dependencies } from '$lib/constants';
    import { trackEvent } from '$lib/actions/analytics';

    async function onFinish() {
        try {
            const { $id } = await sdkForConsole.projects.createKey(
                $page.params.project,
                $key.name,
                $key.scopes,
                $key.expire ?? undefined
            );
            if ($onboarding) {
                invalidate(Dependencies.PROJECT);
            }
            trackEvent('submit_key_create');
            goto(`/console/project-${$page.params.project}/overview/keys/${$id}`);
        } catch (error) {
            addNotification({
                type: 'error',
                message: error.message
            });
        }
    }

    onDestroy(() => {
        key.reset();
    });

    beforeNavigate(() => {
        wizard.hide();
    });

    const stepsComponents: WizardStepsType = new Map();
    stepsComponents.set(1, {
        label: 'Details',
        component: Step1
    });
    stepsComponents.set(2, {
        label: 'Scopes',
        component: Step2
    });
</script>

<Wizard title="Create an API Key" steps={stepsComponents} on:finish={onFinish} />
