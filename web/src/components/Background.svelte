
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

    const polygon = [
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

    function explode(line, distance) {
        const mid = {
            x: line.start.x + (line.end.x - line.start.x) / 2,
            y: line.start.y + (line.end.y - line.start.y) / 2,
            z: line.start.z + (line.end.z - line.start.z) / 2,
        };

        const proj_mid = projectPoint(mid);

        const dx = proj_mid.x - mousePos.x;
        const dy = proj_mid.y - mousePos.y;

        const dist = Math.sqrt(dx * dx + dy * dy);

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
            updateShape();
            drawShape(ctx);
        });
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

    onMount(() => {
        const background = document.querySelector("canvas.background");
        const ctx = background.getContext('2d');

        const width = background.clientWidth;
        const height = background.clientHeight;

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