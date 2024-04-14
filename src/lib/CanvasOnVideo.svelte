<script lang="ts">
    import vod from "../assets/vod.mp4";
    import paper from 'paper/dist/paper-core';
    import { onMount, tick } from "svelte";
    import PaperCanvas from "./PaperCanvas.svelte";

    let video: HTMLVideoElement;

    let path: paper.Path;
    let paused: boolean;
    let currentTime: number;
    let duration: number;

    let layerIds: Set<number> = new Set();

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

        // set time of frame
        paper.project.activeLayer.data.time = currentTime;
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
        }
    }

    function checkSaveActiveLayer(newLayer = true) {
        const activeLayer = paper.project.activeLayer;
        if (!activeLayer.isEmpty()) {
            layerIds.add(activeLayer.id);
            layerIds = layerIds; // trigger Svelte reactivity
            activeLayer.visible = false;
            if (newLayer) {
                new paper.Layer(); // makes new Layer active layer
            }
        }
    }

    function jumpToLayer(id: number) {
        return () => {
            paused = true;

            checkSaveActiveLayer(false);
            
            const frameLayer = paper.project.getItem({
                id,
                class: paper.Layer,
            }) as paper.Layer;
            
            const time = frameLayer.data.time;
                
            // confirmed this type of error does not occur in Chrome, but does in Firefox
            // https://github.com/sveltejs/svelte/issues/3524
            // even if you video.pause() right here, and then set the currentTime with
            // `currentTime = time;`, the video will pause but not jump to the time.
            // If you call this again via a click, it will jump the currentTime
            // so the tick() is a Svelte workaround mentioned in the above issue to get
            // this to work in Firefox
            tick().then(() => currentTime = time);

            frameLayer.activate();
            frameLayer.visible = true;
        }
    }

    function handleSlider() {
        paused = true;
        checkSaveActiveLayer();
    }

    function handlePlay() {
        checkSaveActiveLayer();
    }
</script>

<section>
    <div class="container">
        <div class="video-canvas">
            <video
                bind:this={video}
                bind:paused
                bind:currentTime
                bind:duration
                on:play={handlePlay}
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

        <input
            type="range"
            min={0}
            max={duration}
            step={1}
            bind:value={currentTime}
            on:input={handleSlider}
        />
        
        <div class="flex">
            {#each layerIds as id}
                <button
                    type="button"
                    on:click={jumpToLayer(id)}
                >{id}</button>
            {/each}
        </div>
    </div>

</section>

<style>
    .flex {
        display: flex;
        gap: 1rem;
        flex-wrap: wrap;
    }

    .container {
        width: 1024px;
    }
    
    .video-canvas {
        width: 100%;
        height: 576px;
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
