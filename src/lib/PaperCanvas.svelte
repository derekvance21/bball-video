<script lang="ts">
    import paper from 'paper/dist/paper-core';
    import { onMount, createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();

    let canvas: HTMLCanvasElement;

    onMount(() => {
        paper.view.onMouseDown = (e: paper.MouseEvent) => { dispatch('onmousedown', e); };
        paper.view.onMouseDrag = (e: paper.MouseEvent) => { dispatch('onmousedrag', e); };
        paper.view.onMouseUp = (e: paper.MouseEvent) => { dispatch('onmouseup', e); };
        paper.view.onMouseMove = (e: paper.MouseEvent) => { dispatch('onmousemove', e); };
    });

</script>

<canvas bind:this={canvas} use:paper.setup></canvas>

<style>
    canvas {
        width: 100%;
        height: 100%;
    }
</style>
