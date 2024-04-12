<script lang="ts">
    import vod from "../assets/vod.mp4";
    import paper from 'paper/dist/paper-core';
    import { onMount } from "svelte";
    import PaperCanvas from "./PaperCanvas.svelte";

    let video: HTMLVideoElement;

    let path: paper.Path;
    let paused: boolean;
    let currentTime: number;
    let duration: number;

    onMount(() => {
        currentTime = 60 * 15 + 40; // tipoff
    });

    function onMouseDown(customEvent: CustomEvent) {
        const event: paper.MouseEvent = customEvent.detail;
        path = new paper.Path({
            segments: [event.point],
            strokeColor: "violet",
        });
    }

    function onMouseDrag(customEvent: CustomEvent) {
        const event: paper.MouseEvent = customEvent.detail;
        paused = true;
        if (path) {
            path.add(event.point);
        }
    }

    function onMouseUp(customEvent: CustomEvent) {
        const event: paper.MouseEvent = customEvent.detail;
        const segmentCount = path.segments.length;

        if (segmentCount === 1) {
            if (path) {
                path.remove();
            }
            paused = !paused;
        } else {
            if (path) {
                path.simplify(10);
            }
            // path.closePath(); // close path AFTER simplifying
        }
    }

    function onPlay() {
        const items = paper.project.activeLayer.removeChildren();
        // TODO: do something with these. Save them?
    }
</script>

<section>
    <div class="container">
        <video
            bind:this={video}
            bind:paused
            bind:currentTime
            bind:duration
            on:play={onPlay}
            autoplay
            muted
            disablepictureinpicture={true}
        >
            <source src={vod} type="video/mp4" />
            <track kind="captions" />
        </video>
        <div class="canvas">
            <PaperCanvas
                on:onmousedown={onMouseDown}
                on:onmousedrag={onMouseDrag}
                on:onmouseup={onMouseUp}
            />
        </div>
    </div>

    <input type="range" min={0} max={duration} step={1} bind:value={currentTime} />

</section>

<style>
    .container {
        width: 720px;
        height: 406px;
        position: relative;
    }

    video {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    .canvas {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    input[type=range] {
        width: 100%;
    }
</style>
