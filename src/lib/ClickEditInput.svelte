<script lang="ts">
    export let text = "text";

    export let multiline = false;

    let editing = false;
    let inputElement: HTMLInputElement | undefined;
    let textareaElement: HTMLTextAreaElement | undefined;

    const handleClick = () => {
        editing = true;

        setTimeout(() => {
            if (multiline) {
                textareaElement?.focus();
                textareaElement?.select();
            } else {
                inputElement?.focus();
                inputElement?.select();
            }
        }, 0);
    };

    const handleInput = (evt: Event) => {
        text = (evt.currentTarget as HTMLInputElement)?.value;
    };

    const handleKeyDown = (evt: KeyboardEvent) => {
        if (!multiline && evt.key === "Enter") {
            editing = false;
        }
    };

    const handleBlur = () => {
        editing = false;
    };
</script>

{#if editing}
    {#if multiline}
        <textarea
            bind:this={textareaElement}
            class="nodrag"
            on:input={handleInput}
            on:keydown={handleKeyDown}
            on:blur={handleBlur}
            bind:value={text}
        />
    {:else}
        <input
            bind:this={inputElement}
            class="nodrag"
            type="string"
            on:input={handleInput}
            on:keydown={handleKeyDown}
            on:blur={handleBlur}
            bind:value={text}
        />
    {/if}
{:else}
    <button on:click={handleClick}>{text}</button>
{/if}

<style>
    button {
        background: none;
        border: none;
        font: inherit;
        cursor: pointer;
        white-space: pre-line;
    }
</style>
