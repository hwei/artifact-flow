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
        return targetNode?.type !== "process";
    };

    $$restProps;
</script>

<div class="process-node">
    <Handle type="target" position={Position.Top} />
    <ClickEditInput bind:text={data.name} />
    <Handle type="source" position={Position.Bottom} {isValidConnection} />
</div>

<style>
    .process-node {
        padding: 1rem;
        background: #eee;
        font-size: 0.7rem;
        border: #ddd solid 0.125rem;
    }
    :global(.svelte-flow__node.selected) > .process-node {
        border: #444 solid 0.125rem;
    }
</style>
