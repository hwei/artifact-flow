<script lang="ts">
    import {
        type NodeProps,
    } from "@xyflow/svelte";

    import ClickEditInput from "./ClickEditInput.svelte";

    export let data = {
        text: "",
    };

    type $$Props = NodeProps & {
        data: typeof data;
    };

</script>

<div class="note-node">
    <ClickEditInput bind:text={data.text} multiline={true} />
</div>

<style>
    .note-node {
        padding: 1rem;
        background: #ffe;
        font-size: 0.7rem;
        border: #ddd solid 0.125rem;
    }
    :global(.svelte-flow__node.selected) > .note-node {
        border: #444 solid 0.125rem;
    }
</style>
