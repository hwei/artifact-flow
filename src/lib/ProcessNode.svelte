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
        border-radius: 0.125rem;
        font-size: 0.7rem;
    }
</style>
