<script lang="ts">
    import { onMount } from 'svelte';
    import { FormItem, Helper } from '.';

    export let label: string;
    export let showLabel = true;
    export let id: string;
    export let value: number = null;
    export let placeholder = '';
    export let required = false;
    export let disabled = false;
    export let readonly = false;
    export let autofocus = false;
    export let min: number = null;
    export let max: number = null;
    export let step: number | 'any' = 1;

    let element: HTMLInputElement;
    let error: string;

    onMount(() => {
        if (element && autofocus) {
            element.focus();
        }
    });

    const handleInvalid = (event: Event) => {
        event.preventDefault();

        if (element.validity.valueMissing) {
            error = 'This field is required';
            return;
        }

        if (element.validity.rangeOverflow) {
            error = `The value must be less than or equal to ${max}`;
            return;
        }

        if (element.validity.rangeUnderflow) {
            error = `The value must be greater than or equal to ${min}`;
            return;
        }

        error = element.validationMessage;
    };

    $: if (value) {
        error = null;
    }
</script>

<FormItem>
    <label class:u-hide={!showLabel} class="label" for={id}>{label}</label>
    <div class="input-text-wrapper">
        <input
            {id}
            {placeholder}
            {disabled}
            {required}
            {min}
            {max}
            {readonly}
            {step}
            type="number"
            class="input-text"
            bind:value
            bind:this={element}
            on:invalid={handleInvalid} />
    </div>
    {#if error}
        <Helper type="warning">{error}</Helper>
    {/if}
</FormItem>
