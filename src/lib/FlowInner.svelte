<script lang="ts">
    import { writable } from "svelte/store";
    import {
        SvelteFlow,
        useSvelteFlow,
        Background,
        Controls,
        type Node,
        type Edge,
        type XYPosition,
    } from "@xyflow/svelte";

    import "@xyflow/svelte/dist/style.css";

    import ArtifactNode from "./ArtifactNode.svelte";
    import ProcessNode from "./ProcessNode.svelte";
    import ContextMenu from "./ContextMenu.svelte";
    import NoteNode from "./NoteNode.svelte";

    const { screenToFlowPosition, toObject } = useSvelteFlow();

    const nodeTypes = {
        artifact: ArtifactNode,
        process: ProcessNode,
        note: NoteNode,
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

    function saveFileWithBlob(text: string) {
        const blob = new Blob([text], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        a.download = "flow.json";
        a.click();
        URL.revokeObjectURL(url);
        a.remove();
    }
    
    let fileHandle: FileSystemFileHandle | undefined;

    async function saveFileWithFileSystem(fileHandle: FileSystemFileHandle, text: string) {
        let writable: FileSystemWritableFileStream | undefined;
        try {
            writable = await fileHandle.createWritable();
            await writable.write(text);
        } catch (err) {
            console.error(err);
            saveFileWithBlob(text);
        }
        if (writable !== undefined) {
            await writable.close();
        }
    }

    async function handleSave() {
        const data = toObject();
        const jsonString = JSON.stringify(data, null, 2);

        if ('showSaveFilePicker' in window) {
            if (fileHandle === undefined) {
                try {
                    fileHandle = await (window.showSaveFilePicker as any)({
                        suggestedName: 'flow.json',
                        types: [{
                            description: "JSON files",
                            accept: {
                                "application/json": [".json"],
                            },
                        }],
                    });
                } catch (err) {
                    console.error(err);
                }
            }

            if (fileHandle !== undefined) {
                saveFileWithFileSystem(fileHandle, jsonString);
            }
        } else {
            saveFileWithBlob(jsonString);
        }
    }

    function loadFromJsonText(jsonText: string) {
        const data = JSON.parse(jsonText);
        const maxNodeId = data.nodes.reduce((maxId: number, node: Node) => {
            const id = parseInt(node.id);
            return id > maxId ? id : maxId;
        }, 0);
        nextNodeId = maxNodeId + 1;

        nodes.set(data.nodes);
        edges.set(data.edges);
    }

    async function selectFile(startFile?: File) {
        if (!('showOpenFilePicker' in window)) {
            return;
        }
        try {
            [ fileHandle ] = await (window.showOpenFilePicker as any)({
                suggestedName: startFile ? startFile.name : undefined,
                types: [{
                    description: "JSON files",
                    accept: {
                        "application/json": [".json"],
                    },
                }],
            });
            if (fileHandle !== undefined) {
                console.log('fileHandle', fileHandle);
                const file = await fileHandle.getFile();
                const text = await file.text();
                loadFromJsonText(text);
            }
        } catch (err) {
            console.error(err);
        }
    }

    function handleLoad() {
        if ('showOpenFilePicker' in window) {
            selectFile();
            return;
        }
        const input = document.createElement("input");
        input.type = "file";
        input.accept = ".json";
        input.onchange = async (event) => {
            const file = (event.target as HTMLInputElement).files?.[0];
            if (!file) {
                return;
            }
            const text = await file.text();
            loadFromJsonText(text);
            input.remove();
        };
        input.click();
    }

    function handleClose() {
        nodes.set([]);
        edges.set([]);
        nextNodeId = 1;
        fileHandle = undefined;
    }

    function onDragOver(event: DragEvent) {
        event.preventDefault();

        if (event.dataTransfer?.types.includes("application/json")) {
            event.dataTransfer.dropEffect = "copy";
        }
    }

    function onDrop(event: DragEvent) {
        event.preventDefault();

        if (!event.dataTransfer) {
            return;
        }

        const [file] = event.dataTransfer.files;
        if (!file) {
            return;
        }

        if ('showOpenFilePicker' in window) {
            selectFile(file);
        } else {
            const reader = new FileReader();
            reader.onload = async (e) => {
                if (!e.target) {
                    return;
                }
                const text = e.target.result as string;
                loadFromJsonText(text);
            };
            reader.readAsText(file);
        }
    }
</script>


    <SvelteFlow
        {nodeTypes}
        {nodes}
        {edges}
        fitView
        minZoom={0.1}
        panOnDrag={[1]}
        selectionOnDrag={true}
        on:panecontextmenu={(e) => handlePaneContextMenu(e.detail.event)}
        on:paneclick={() => handleContexMenuClick()}
        on:dragover={onDragOver}
        on:drop={onDrop}
    >
        <div class="controls">
            <div>{fileHandle ? fileHandle.name : ''}</div>
            <button on:click={handleSave}>Save</button>
            <button on:click={handleLoad}>Load</button>
            <button on:click={handleClose}>Close</button>
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
