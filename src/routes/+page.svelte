<script lang="ts">
    
    import { SvelteFlowProvider } from '@xyflow/svelte';
    import Flow from '$lib/Flow.svelte';
</script>

<main>
    <SvelteFlowProvider>
        <Flow />
    </SvelteFlowProvider>
</main>
