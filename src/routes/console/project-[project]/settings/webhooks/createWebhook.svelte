<script lang="ts">
    import { Wizard } from '$lib/layout';
    import { onDestroy } from 'svelte';
    import { addNotification } from '$lib/stores/notifications';
    import { wizard } from '$lib/stores/wizard';
    import { createWebhook } from './wizard/store';
    import { sdkForConsole } from '$lib/stores/sdk';
    import { page } from '$app/stores';
    import Step1 from './wizard/step1.svelte';
    import Step2 from './wizard/step2.svelte';
    import Step3 from './wizard/step3.svelte';
    import type { WizardStepsType } from '$lib/layout/wizard.svelte';
    import { invalidate } from '$app/navigation';
    import { Dependencies } from '$lib/constants';
    import { trackEvent } from '$lib/actions/analytics';

    const projectId = $page.params.project;
    const create = async () => {
        try {
            await sdkForConsole.projects.createWebhook(
                projectId,
                $createWebhook.name,
                $createWebhook.events,
                $createWebhook.url,
                $createWebhook.security,
                $createWebhook.httpUser,
                $createWebhook.httpPass
            );
            invalidate(Dependencies.WEBHOOKS);
            addNotification({
                message: 'Webhook has been created',
                type: 'success'
            });
            wizard.hide();
            trackEvent('submit_webhook_create');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        }
    };

    onDestroy(() => {
        $createWebhook = {
            name: null,
            url: null,
            events: [],
            security: false,
            httpUser: null,
            httpPass: null
        };
    });

    const stepsComponents: WizardStepsType = new Map();
    stepsComponents.set(1, {
        label: 'Add your webhook',
        component: Step1
    });
    stepsComponents.set(2, {
        label: 'Webhook events',
        component: Step2
    });
    stepsComponents.set(3, {
        label: 'Security',
        component: Step3,
        optional: true
    });
</script>

<Wizard confirmExit title="Create Webhook" steps={stepsComponents} on:finish={create}>
    <svelte:fragment slot="exit">webhook creation</svelte:fragment>
</Wizard>
