<script lang="ts">
    import { writable } from "svelte/store";
    import {
        Handle,
        Position,
        useSvelteFlow,
        type NodeProps,
        type IsValidConnection,
    } from "@xyflow/svelte";

    import ClickEditInput from "./ClickEditInput.svelte";

    export let data = {
        name: "name",
    };

    type $$Props = NodeProps & {
        data: typeof data;
    };

    const { getNode } = useSvelteFlow();

    const isValidConnection: IsValidConnection = (connection) => {
        const targetNode = getNode(connection.target);
        return targetNode?.type !== "artifact";
    };
</script>

<div class="artifact-node">
    <Handle type="target" position={Position.Top} />
    <ClickEditInput bind:text={data.name} />
    <Handle type="source" position={Position.Bottom} {isValidConnection} />
</div>

<style>
    .artifact-node {
        padding: 1rem;
        background: #eee;
        font-size: 0.7rem;
        border-radius: 50%;
        border: #ddd solid 0.125rem;
    }
    :global(.svelte-flow__node.selected) > .artifact-node {
        border: #444 solid 0.125rem;
    }
</style>
