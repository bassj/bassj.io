<style>
    canvas.background {
        position: fixed;
        width: 100vw;
        height: 100vh;
        opacity: 0.5;
        z-index: 0;
    }
</style>

<script>
    import { onMount } from "svelte";

    export let scroll_container;

    let polygon = [
        {
            start: point(-1, -1, -1),
            end: point(1, -1, -1)
        },
        {
            start: point(-1, -1, 1),
            end: point(1, -1, 1)
        },
        {
            start: point(-1, -1, 1),
            end: point(-1, -1, -1)
        },
        {
            start: point(1, -1, 1),
            end: point(1, -1, -1)
        },
        {
            start: point(-1, 1, -1),
            end: point(1, 1, -1)
        },
        {
            start: point(-1, 1, 1),
            end: point(1, 1, 1)
        },
        {
            start: point(-1, 1, 1),
            end: point(-1, 1, -1)
        },
        {
            start: point(1, 1, 1),
            end: point(1, 1, -1)
        },
        {
            start: point(1, 1, 1),
            end: point(1, -1, 1)
        },
        { 
            start: point(-1, 1, 1),
            end: point(-1, -1, 1)
        },
        {
            start: point(1, 1, -1),
            end: point(1, -1, -1)
        },
        {
            start: point(-1, 1, -1),
            end: point(-1, -1, -1)
        }
    ]

    let rotation = 0;
    let startRotation = 0;
    let explosion = 1;
    let scroll_transition = 0;
    let lastTime = new Date().getTime();

    let isMouseDown = false;
    let mouseStart = {
        x: 0,
        y: 0,
    };

    let mousePos = {
        x: 0,
        y: 0
    };

    let viewport =  {
        x: 0,
        y: 0,
        width: 500,
        height: 500
    };

    function loadObj(source) {
        const lines = source.split('\n');

        let vertices = [];

        let polygon = [];

        lines.forEach(function (line) {
            const split = line.split(' ');
            if (split[0] == 'v') {
                const x = parseFloat(split[1]);
                const y = parseFloat(split[2]);
                const z = parseFloat(split[3]);
                vertices.push({ x, y, z});
            } else if (split[0] == 'f') {
                for (let i = 2; i < split.length; i++) {
                    const startIndex = parseInt(split[i - 1]) - 1;
                    const endIndex = parseInt(split[i]) - 1;

                    const start = vertices[startIndex];
                    const end = vertices[endIndex];

                    polygon.push({
                        start,
                        end
                    });
                }
            }
        });

        return polygon;
    }

    function point(x, y, z) {
        return {x, y, z};
    }

    function magnitude({x, y, z}) {
        return Math.sqrt(x * x + y * y + z * z);
    }

    function dot(a, b) {
        return a.x * b.x + a.y * b.y + a.z * b.z;
    }

    function scale(vec, scalar) {
        return {
            x: vec.x * scalar,
            y: vec.y * scalar,
            z: vec.z * scalar
        };
    }

    function subtract(a, b) {
        return {
            x: a.x - b.x,
            y: a.y - b.y,
            z: a.z - b.z
        };
    }

    function add(a, b) {
        return {
            x: a.x + b.x,
            y: a.y + b.y,
            z: a.z + b.z
        };
    }

    function rotateY(point, angle) {
        const cos = Math.cos(angle * (Math.PI / 180));
        const sin = Math.sin(angle * (Math.PI / 180));
        return {
            x: cos * point.x - sin * point.z,
            y: point.y,
            z: sin * point.x + cos * point.z
        }
    }

    function projectPoint({x, y, z}) {

        const fovY = 60;

        const s = 1.5;

        const lookDir = {
            x: 0,
            y: 0,
            z: 1
        };

        const cameraPos = {
            x: 0,
            y: 0,
            z: -10,
        }

        x *= s; y *= s; z *= s;
        x -= cameraPos.x; y -= cameraPos.y; z -= cameraPos.z;

        // Projection of our vector onto the normal of the plane.
        const proj_normal = scale(lookDir, dot({x, y, z}, lookDir) / 1);
        const proj_point = subtract({x, y, z}, proj_normal);

        const depth = magnitude(proj_normal);

        const plane_width = Math.tan((fovY / 2) * (Math.PI / 180)) * (2 * depth);

        const sx = viewport.width * (1 / plane_width);
        const sy = viewport.width * (1 / plane_width);

    
        return {
            x: proj_point.x * sx + viewport.width / 2 + viewport.x,
            y: proj_point.y * sy + viewport.height / 2 + viewport.y
        }
    }

    function normalize(vec) {
        const mag = magnitude(vec);

        const norm = {
            x: vec.x / mag,
            y: vec.y / mag,
            z: vec.z / mag
        };

        return norm;
    }

    function sqr(x) { return x * x }
    function dist2(v, w) { return sqr(v.x - w.x) + sqr(v.y - w.y) }

    function min_dist(point, line) {
        const l2 = dist2(line.start, line.end);
        if (l2 == 0) return dist2(point, line.start);

        let t = ((point.x - line.start.x) * (line.end.x - line.start.x) + (point.y - line.start.y) * (line.end.y - line.start.y)) / l2;
        t = Math.max(0, Math.min(1, t));
        
        const dist_sqr = dist2(point, {
            x: line.start.x + t * (line.end.x - line.start.x),
            y: line.start.y + t * (line.end.y - line.start.y)
        });

        return Math.sqrt(dist_sqr);
    }

    function explode(line, distance) {
        const mid = {
            x: line.start.x + (line.end.x - line.start.x) / 2,
            y: line.start.y + (line.end.y - line.start.y) / 2,
            z: line.start.z + (line.end.z - line.start.z) / 2,
        };

        const proj_line = {
            start: projectPoint(line.start),
            end: projectPoint(line.end)
        };

        const dist = min_dist(mousePos, proj_line);

        let explosion_bias = 1.0;

        const explosion_start = 300;

        explosion_bias = (explosion_start - dist) / explosion_start;

        explosion_bias /= 2;

        if (explosion_bias < 0) {
            explosion_bias = 0;
        }

        const explosion_dir = normalize(mid);

        const exploded_line = {
            start: add(line.start, scale(explosion_dir, distance + explosion_bias)),
            end: add(line.end, scale(explosion_dir, distance + explosion_bias))
        }

        return exploded_line;
    }

    function drawLine(ctx, start, end) {
        const proj_start = projectPoint(start);
        const proj_end = projectPoint(end);

        ctx.beginPath();
        ctx.moveTo(proj_start.x, proj_start.y);
        ctx.lineTo(proj_end.x, proj_end.y);
        ctx.stroke();
    }

    function updateShape() {
        const time = new Date().getTime();
        const dt = (time - lastTime) / 1000;
        lastTime = time;
        
        const centerX = viewport.width / 2;
        const centerY = viewport.height / 2;

        const dx = mousePos.x - centerX;
        const dy = mousePos.y - centerY;

        const dist = Math.sqrt(dx * dx + dy * dy);

        const explosion_dist = 250;
        const explosion_start = 600;

        explosion = (explosion_dist - Math.min(dist, explosion_start)) / (2 * explosion_dist);
        explosion = Math.max(explosion, scroll_transition * 10);

        if (explosion < 0) {
            explosion = 0;
        }

        if (isMouseDown) {
            rotation = startRotation + (mousePos.x - mouseStart.x) / 10;
        } else {
            rotation += 25 * dt;
        }
    }

    function drawShape(ctx) {
        ctx.clearRect(viewport.x, viewport.y, viewport.width, viewport.height);
        
        if (scroll_transition < 0.7) {
            ctx.strokeStyle = "#86836D";
            ctx.lineWidth = 1;

            polygon.forEach(function (line) {
                const rotated_line = {
                    start: rotateY(line.start, rotation),
                    end: rotateY(line.end, rotation)
                };

                const exploded_line = explode(rotated_line, explosion);
                
                drawLine(ctx, exploded_line.start, exploded_line.end);
            });

            requestAnimationFrame(function () {
                if (scroll_container) {
                    scroll_transition = scroll_container.scrollTop / viewport.height;
                }
                updateShape();
                drawShape(ctx);
            });
        }

        

    }

    function mouseMove(e) {
        mousePos.x = e.clientX;
        mousePos.y = e.clientY;
    }

    function mouseDown(e) {
        isMouseDown = true;
        mouseStart.x = e.clientX;
        mouseStart.y = e.clientY;
        startRotation = rotation;
    }

    function mouseUp(e) {
        isMouseDown = false;
    }

    onMount(async () => {
        const background = document.querySelector("canvas.background");
        const ctx = background.getContext('2d');

        const width = background.clientWidth;
        const height = background.clientHeight;

        const modelSource = `v 0 0.618034 1.61803
v 0 -0.618034 1.61803
v 0 -0.618034 -1.61803
v 0 0.618034 -1.61803
v 1.61803 0 0.618034
v -1.61803 0 0.618034
v -1.61803 0 -0.618034
v 1.61803 0 -0.618034
v 0.618034 1.61803 0
v -0.618034 1.61803 0
v -0.618034 -1.61803 0
v 0.618034 -1.61803 0
v 1 1 1
v -1 1 1
v -1 -1 1
v 1 -1 1
v 1 -1 -1
v 1 1 -1
v -1 1 -1
v -1 -1 -1
# 20 vertices

f 1 2 16 5 13
f 1 13 9 10 14
f 1 14 6 15 2
f 2 15 11 12 16
f 3 4 18 8 17
f 3 17 12 11 20
f 3 20 7 19 4
f 19 10 9 18 4
f 16 12 17 8 5
f 5 8 18 9 13
f 14 10 19 7 6
f 6 7 20 11 15
# 12 faces`;

        polygon = loadObj(modelSource);

        window.addEventListener("resize", function (e) {
            background.width = background.clientWidth;
            background.height = background.clientHeight;
            viewport.width = background.clientWidth;
            viewport.height = background.clientHeight;
        });

        window.addEventListener("mousemove", mouseMove);

        background.width = width;
        background.height = height;
        viewport.width = width;
        viewport.height = height;

        requestAnimationFrame(function () {
            updateShape();
            drawShape(ctx);
        });
    });
</script>

<canvas class="background" on:mousedown={mouseDown} on:mouseup={mouseUp}>
</canvas>
