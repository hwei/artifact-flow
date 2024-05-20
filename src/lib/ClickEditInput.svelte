<script lang="ts">
    export let text = "text";

    let editing = false;
    let inputElement: HTMLInputElement | undefined;

    const handleClick = () => {
        editing = true;
        setTimeout(() => {
            inputElement?.focus();
            inputElement?.select();
        }, 0);
    };
</script>

<button on:click={handleClick}>
    {#if editing}
        <input
            bind:this={inputElement}
            class="nodrag"
            type="string"
            on:input={(evt) => {
                text = evt.currentTarget?.value;
            }}
            on:keydown={(evt) => {
                if (evt.key === "Enter") {
                    editing = false;
                }
            }}
            on:blur={() => {
                editing = false;
            }}
            bind:value={text}
        />
    {:else}
        <strong>{text}</strong>
    {/if}
</button>

<style>
    button {
        background: none;
        border: none;
        font: inherit;
        cursor: pointer;
    }
</style>
