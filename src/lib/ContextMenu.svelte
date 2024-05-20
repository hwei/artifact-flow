<script lang="ts">

import {createEventDispatcher} from 'svelte';

const dispatch = createEventDispatcher();

export let data = { x: 0, y: 0, menuItems: ['item0'] };

function onClick(n: number) {
    dispatch('menuClick', n);
}

</script>

<div class="context-menu" style="top: {data.y}px; left: {data.x}px;">
    {#each data.menuItems as item, i }
        <button on:click={() => onClick(i)}>{item}</button>
    {/each}
</div>

<style>
    .context-menu {
        position: absolute;
        z-index: 10;
        background: #fff;
        border: 1px solid #000;
        padding: 0.5rem;
        border-radius: 0.25rem;
        box-shadow: 0 0 0.5rem rgba(0, 0, 0, 0.5);
    }

    .context-menu button {
        display: block;
        width: 100%;
        padding: 0.25rem 0.5rem;
        border: none;
        background: none;
        text-align: left;
    }

    .context-menu button:hover {
        background-color: #f2f2f2;
    }

</style>