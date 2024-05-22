<script lang="ts">
    import { writable } from "svelte/store";
    import {
        SvelteFlowProvider,
        SvelteFlow,
        useSvelteFlow,
        Background,
        Controls,
        type Node,
        type Edge,
        type XYPosition,
    } from "@xyflow/svelte";

    import "@xyflow/svelte/dist/style.css";

    import ArtifactNode from "$lib/ArtifactNode.svelte";
    import ProcessNode from "$lib/ProcessNode.svelte";
    import ContextMenu from "$lib/ContextMenu.svelte";

    const { screenToFlowPosition, toObject } = useSvelteFlow();

    const nodeTypes = {
        artifact: ArtifactNode,
        process: ProcessNode,
    };
    const nodeTypeNames = Object.keys(nodeTypes);

    const nodes = writable<Node[]>([]);
    const edges = writable<Edge[]>([]);

    let createNodeMenuData: ContextMenu["$$prop_def"]["data"] | undefined;

    let createPos: XYPosition | undefined;

    function handlePaneContextMenu(event: MouseEvent) {
        event.preventDefault();
  
        createPos = screenToFlowPosition({
            x: event.clientX,
            y: event.clientY,
        });

        createNodeMenuData = {
            x: event.clientX,
            y: event.clientY,
            menuItems: nodeTypeNames,
        };
    }

    let nextNodeId = 1;

    function handleContexMenuClick(n?: number) {
        createNodeMenuData = undefined;
        console.log(n);

        if (n === undefined || createPos === undefined) {
            return;
        }

        const nodeId = nextNodeId;
        const nodeTypeName = nodeTypeNames[n];
        const id = `${nodeId}`;

        const newNode = {
            id,
            type: nodeTypeName,
            position: createPos,
            data: { name: nodeTypeName + "-" + id },
        };
        nodes.update((oldNodes) => [...oldNodes, newNode]);

        nextNodeId++;
    }

    function handleSave() {
        const data = toObject();
        console.log(data);
        const jsonString = JSON.stringify(data, null, 2);
        const blob = new Blob([jsonString], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = "flow.json";
        a.click();
        URL.revokeObjectURL(url);
        a.remove();
    }

    function handleLoad() {
        const input = document.createElement("input");
        input.type = "file";
        input.accept = ".json";
        input.onchange = async (event) => {
            const file = (event.target as HTMLInputElement).files?.[0];
            if (!file) {
                return;
            }
            const text = await file.text();
            const data = JSON.parse(text);
            const maxNodeId = data.nodes.reduce((maxId: number, node: Node) => {
                const id = parseInt(node.id);
                return id > maxId ? id : maxId;
            }, 0);
            nextNodeId = maxNodeId + 1;

            nodes.set(data.nodes);
            edges.set(data.edges);
            input.remove();
        };
        input.click();
    }

    function handleClear() {
        nodes.set([]);
        edges.set([]);
        nextNodeId = 1;
    }
</script>


    <SvelteFlow
        {nodeTypes}
        {nodes}
        {edges}
        fitView
        on:panecontextmenu={(e) => handlePaneContextMenu(e.detail.event)}
        on:paneclick={() => handleContexMenuClick()}
    >
        <div class="controls">
            <button on:click={handleSave}>Save</button>
            <button on:click={handleLoad}>Load</button>
            <button on:click={handleClear}>Clear</button>
        </div>
        <Background />
        <Controls />
        {#if createNodeMenuData}
            <ContextMenu
                data={createNodeMenuData}
                on:menuClick={(e) => handleContexMenuClick(e.detail)}
            />
        {/if}
    </SvelteFlow>


<style>

    .controls {
        position: absolute;
        top: 10px;
        right: 10px;
        padding: 1rem;
        z-index: 4;
    }
</style>
