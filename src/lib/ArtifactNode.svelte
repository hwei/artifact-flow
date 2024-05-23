<script lang="ts">
    import {
        Handle,
        Position,
        useSvelteFlow,
        type NodeProps,
        type IsValidConnection,
        useHandleConnections,
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

    export let id: $$Props['id'];
    const targetConnections = useHandleConnections({ nodeId: id, type: 'target' });
    const sourceConnections = useHandleConnections({ nodeId: id, type: 'source' });

    $: isBegin = $targetConnections.length === 0 && $sourceConnections.length > 0;
    $: isEnd = $targetConnections.length > 0 && $sourceConnections.length === 0;

    $$restProps;
</script>

<div class="artifact-node" class:begin={isBegin} class:end={isEnd}>
    <Handle type="target" position={Position.Top} />
    <ClickEditInput bind:text={data.name} />
    <Handle type="source" position={Position.Bottom} {isValidConnection} />
</div>

<style>
    .artifact-node {
        padding: 1rem;
        background: #fff;
        font-size: 0.7rem;
        border-radius: 50%;
        border: #ddd solid 0.125rem;
    }
    .artifact-node.begin {
        background: #ffa;
    }
    .artifact-node.end {
        background: #af8;
    }
    :global(.svelte-flow__node.selected) > .artifact-node {
        border: #444 solid 0.125rem;
    }
</style>
