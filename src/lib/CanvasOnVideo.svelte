<script lang="ts">
    import vod from "../assets/vod.mp4";
    import paper from 'paper/dist/paper-core';
    import { onMount } from "svelte";
    import { writable, type Writable } from "svelte/store";
    import PaperCanvas from "./PaperCanvas.svelte";

    let video: HTMLVideoElement;

    let path: paper.Path;
    let paused: boolean;
    let currentTime: number;
    let duration: number;

    interface Frame {
        currentTime: number;
        jsonData: string;
    }

    let frameStore: Writable<Map<number, Frame>> = writable(new Map());

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

    function saveActiveLayer() {
        const activeLayer = paper.project.activeLayer;
        if (!activeLayer.isEmpty()) {
            frameStore.update(frameMap => {
                const id = activeLayer.id;
                
                if (!frameMap.has(id)) {
                    console.log('new frame');
                }
                
                const json = activeLayer.exportJSON();

                console.log(json)

                frameMap.set(id, {
                    jsonData: json,
                    currentTime: currentTime,
                });
                return frameMap;
            });
        }
    }
    
    function onPlay() {
        if (!paper.project.activeLayer.isEmpty()) {
            saveActiveLayer();
        }
        const items = paper.project.activeLayer.removeChildren();
        new paper.Layer(); // makes new Layer active layer
        // TODO: do something with these. Save them?
    }

    function pause() {
        video.pause();
    }

    function jumpToFrame(frame: Frame) {
        return () => {
            if (!paused) {
                return;
            }

            console.log(currentTime, '->', frame.currentTime);
            currentTime = frame.currentTime;

            const item = paper.project.importJSON(frame.jsonData);
            console.log(item);
            // paper.project.importJSON(frame.jsonData);
        }
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

    <input
        type="range"
        min={0}
        max={duration}
        step={1}
        bind:value={currentTime}
        on:input={pause}
    />

    <div>
        {#each $frameStore as [id, frame]}
		    <p on:click={jumpToFrame(frame)}>{frame.currentTime}: {frame.jsonData.substring(0, 20)}...</p>
	    {/each}
    </div>

</section>

<style>
    .container {
        width: 1024px;
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
