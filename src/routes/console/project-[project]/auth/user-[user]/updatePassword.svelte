<script lang="ts">
    import { trackEvent } from '$lib/actions/analytics';
    import { CardGrid, Heading } from '$lib/components';
    import { Button, Form, InputPassword } from '$lib/elements/forms';
    import { addNotification } from '$lib/stores/notifications';
    import { sdkForProject } from '$lib/stores/sdk';
    import { user } from './store';

    let newPassword: string = null;

    async function updatePassword() {
        try {
            await sdkForProject.users.updatePassword($user.$id, newPassword);
            newPassword = null;
            addNotification({
                message: 'Password has been updated',
                type: 'success'
            });
            trackEvent('submit_user_update_password');
        } catch (error) {
            addNotification({
                message: error.message,
                type: 'error'
            });
        }
    }
</script>

<Form on:submit={updatePassword}>
    <CardGrid>
        <div>
            <Heading tag="h6" size="7">Update Password</Heading>
        </div>

        <p>
            Enter a new password. A password must contain <b>at least 8 characters.</b>
        </p>
        <svelte:fragment slot="aside">
            <ul>
                <InputPassword
                    id="newPassword"
                    label="New Password"
                    placeholder="Enter new password"
                    autocomplete={false}
                    meter={false}
                    showPasswordButton={true}
                    bind:value={newPassword} />
            </ul>
        </svelte:fragment>

        <svelte:fragment slot="actions">
            <Button disabled={!newPassword} submit>Update</Button>
        </svelte:fragment>
    </CardGrid>
</Form>
