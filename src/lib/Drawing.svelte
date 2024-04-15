<script lang="ts">
    import { onMount, createEventDispatcher, tick } from "svelte";
    import PaperCanvas from "./PaperCanvas.svelte";
    import { PathStyle } from "./drawing";

    let path: paper.Path;
    let paperCanvas: PaperCanvas;
    let paper: Pick<paper.PaperScope, "version" | "settings" | "project" | "projects" | "view" | "tool" | "tools" | "Color" | "CompoundPath" | "Curve" | "CurveLocation" | "Event" | "Gradient" | "GradientStop" | "Group" | "HitResult" | "Item" | "Key" | "KeyEvent" | "Layer" | "Matrix" | "MouseEvent" | "PaperScope" | "Path" | "PathItem" | "Point" | "PointText" | "Project" | "Raster" | "Rectangle" | "Segment" | "Shape" | "Size" | "Style" | "SymbolDefinition" | "SymbolItem" | "TextItem" | "Tool" | "ToolEvent" | "Tween" | "View" | "execute" | "install" | "setup" | "activate">;

    export let pathStyle: number;

    const dispatch = createEventDispatcher();

    onMount(() => {
        paper = paperCanvas.paperScope;
        paper.project.currentStyle.strokeColor = new paper.Color("salmon");
        paper.project.currentStyle.strokeWidth = 2;
    });

    function onMouseDown(customEvent: CustomEvent) {
        const event: paper.MouseEvent = customEvent.detail;
        path = new paper.Path({
            segments: [event.point],
        });
        
    }

    function onMouseDrag(customEvent: CustomEvent) {
        const event: paper.MouseEvent = customEvent.detail;

        if (path.segments.length === 1) {
            dispatch('begindraw', {layerId: paper.project.activeLayer.id});
        }
        path.add(event.point);
    }

    function arrowize(path: paper.Path) {
        const endpoint = path.lastSegment.point;
        const endVector = path.getTangentAt(path.length);
        if (!endVector) {
            return;
        }
        
        const tip = endVector.normalize(10);

        new paper.Group({
            children: [
                path,
                new paper.Path([
                    endpoint.add(tip.rotate(-145, [0, 0])),
                    endpoint,
                    endpoint.add(tip.rotate(145, [0, 0])),
                ])
            ],
        });
    }

    function screenize(path: paper.Path) {
        var endpoint = path.lastSegment.point;
        var endNormal = path.getNormalAt(path.length);
        if (!endNormal) {
            return path;
        };
        var tip = endNormal.normalize(10);

        return new paper.Group({
            children: [
                path,
                new paper.Path([
                    endpoint.add(tip),
                    endpoint,
                    endpoint.add(tip.rotate(180, [0, 0])),
                ])
            ],
        });
    }

    function squiggly(path: paper.Path) {
        const step = 8;

        const start = path.getPointAt(0);
        const squiggle = new paper.Path({
            segments: [start],
        });

        const curveOffset = 3;
        
        for (let offset = step, direction = false; offset < path.length - step; offset += step, direction = !direction) {
            const from = squiggle.lastSegment.point;
            const to = path.getPointAt(offset);
            const tangent = to.subtract(from);
            const midpoint = from.add(tangent.divide(2));
            const normal = tangent.normalize(curveOffset).rotate(direction ? 90 : -90, [0, 0]);
            
            squiggle.curveTo(
                midpoint.add(normal),
                to
            );
        }
        // const finalSegment = new paper.Segment(path.lastSegment.point, path.lastSegment.handleOut)
        // squiggle.addSegments([finalSegment]);
        squiggle.lineTo(path.lastSegment.point);
        path.replaceWith(squiggle);
    }

    function onMouseUp(customEvent: CustomEvent) {
        if (path.segments.length === 1) {
            path.remove();
            dispatch('click');
            return;
        }
        
        switch (pathStyle) {
            case PathStyle.Cut:
                path.simplify();
                arrowize(path);
                break;
            case PathStyle.Pass:
                path.simplify();
                path.dashArray = [8, 10];
                arrowize(path);
                break;
            case PathStyle.Screen:
                path.simplify();
                screenize(path);
                break;
            case PathStyle.Dribble:
                path.simplify();
                arrowize(path);
                squiggly(path);
                break;
        }
        // path.selected = true;
    }

    export function unsetActiveLayer(newLayer = true) {
        const activeLayer = paper.project.activeLayer;
        if (activeLayer.isEmpty()) {
            return;
        }
        
        dispatch('newlayer', {layerId: activeLayer.id});
        activeLayer.visible = false;
        if (newLayer) {
            new paper.Layer(); // makes new Layer active layer
        }
    }

    export function setActiveLayer(id: number) {
        unsetActiveLayer(false);
            
        const frameLayer = paper.project.getItem({
            id,
            class: paper.Layer,
        }) as paper.Layer;

        if (!frameLayer) {
            throw new Error(`layer with id: ${id} does not exist`);
        }
        
        frameLayer.activate();
        frameLayer.visible = true;
    }
</script>

<PaperCanvas
    bind:this={paperCanvas}
    on:onmousedown={onMouseDown}
    on:onmousedrag={onMouseDrag}
    on:onmouseup={onMouseUp}
/>

<style>
</style>
