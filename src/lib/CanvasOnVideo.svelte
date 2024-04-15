<script lang="ts">
    import vod from "../assets/vod.mp4";
    import { onMount, tick } from "svelte";
    import Drawing from "./Drawing.svelte";
    import { PathStyle } from "./drawing";

    let video: HTMLVideoElement;

    let drawing: Drawing;

    let paused: boolean;
    let currentTime: number;
    let duration: number;

    let pathStyle: number = PathStyle.Normal;

    let layerMap: Map<number, number> = new Map();

    onMount(() => {
        currentTime = 60 * 15 + 40; // tipoff
    });

    function onBeginDraw(customEvent: CustomEvent) {
        paused = true;
        layerMap.set(customEvent.detail.layerId, currentTime);
        layerMap = layerMap;
    }

    function jumpToLayer(id: number, time: number) {
        return () => {
            paused = true;
            drawing.setActiveLayer(id);
            
            // confirmed this type of error does not occur in Chrome, but does in Firefox
            // https://github.com/sveltejs/svelte/issues/3524
            // even if you video.pause() right here, and then set the currentTime with
            // `currentTime = time;`, the video will pause but not jump to the time.
            // If you call this again via a click, it will jump the currentTime
            // so the tick() is a Svelte workaround mentioned in the above issue to get
            // this to work in Firefox
            tick().then(() => {
                currentTime = time;
            });
        }
    }

    function onSliderInput() {
        paused = true;
        drawing.unsetActiveLayer();
    }

    function onPlay() {
        drawing.unsetActiveLayer();
    }

    function printSeconds(seconds: number) {
        const h = (seconds / 3600) >> 0;
        const m = (seconds % 3600 / 60) >> 0;
        const s = (seconds % 60) >> 0;
        const minTwoDigits = (i: number) => {
            return i.toString().padStart(2, "0");
        }
        return `${h}:${minTwoDigits(m)}:${minTwoDigits(s)}`
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
                on:play={onPlay}
                autoplay
                muted
                disablepictureinpicture={true}
            >
                <source src={vod} type="video/mp4" />
                <track kind="captions" />
            </video>
            <div class="drawing">
                <Drawing
                    bind:this={drawing}
                    on:begindraw={onBeginDraw}
                    on:click={() => paused = !paused}
                    bind:pathStyle={pathStyle}
                />
            </div>
            <div class="settings">
                <label>
                    <input
                        type="radio"
                        value={PathStyle.Normal}
                        bind:group={pathStyle}
                    />
                    Normal
                </label>
                <label>
                    <input
                        type="radio"
                        checked
                        value={PathStyle.Cut}
                        bind:group={pathStyle}
                    />
                    Cut
                </label>
                <label>
                    <input
                        type="radio"
                        value={PathStyle.Pass}
                        bind:group={pathStyle}
                    />
                    Pass
                </label>
                <label>
                    <input
                        type="radio"
                        value={PathStyle.Screen}
                        bind:group={pathStyle}
                    />
                    Screen
                </label>
                <label>
                    <input
                        type="radio"
                        value={PathStyle.Dribble}
                        bind:group={pathStyle}
                    />
                    Dribble
                </label>
            </div>
        </div>

        <input
            type="range"
            min={0}
            max={duration}
            step={1}
            bind:value={currentTime}
            on:input={onSliderInput}
        />
        
        <div class="flex">
            {#each layerMap as [id, time]}
                <button
                    type="button"
                    on:click={jumpToLayer(id, time)}
                >{printSeconds(time)}</button>
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

    label {
        display: flex;
    }

    .container {
        width: 896px;
    }
    
    .video-canvas {
        width: 100%;
        height: 504px;
        position: relative;
    }

    video {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    .drawing {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
    }

    .settings {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        position: absolute;
        left: 100%;
    }

    input[type=range] {
        width: 100%;
    }
</style>
