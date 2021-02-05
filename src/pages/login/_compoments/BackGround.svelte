<script>
  import { onMount } from "svelte";
  let canvas;

  onMount(() => {
    const STAR_COUNT = (window.innerWidth + window.innerHeight) / 8;
    const STAR_SIZE = 3;
    const STAR_MIN_SCALE = 0.2;
    const OVERFLOW_THRESHOLD = 50;
    let scale = 1;
    let width = 0;
    let height = 0;
    let stars = [];
    let pointerX = 0;
    let pointerY = 0;
    let velocity = { x: 0, y: 0, tx: 0, ty: 0, z: 0.0005 };
    let touchInput = false;
    let context = canvas.getContext("2d");
    window.onresize = resize;
    canvas.onmousemove = onMouseMove;
    canvas.ontouchmove = onTouchMove;
    canvas.ontouchend = onMouseLeave;
    document.onmouseleave = onMouseLeave;
    generate();
    resize();
    step();
    function generate() {
      for (var i = 0; i < STAR_COUNT; i++) {
        stars.push({
          x: 0,
          y: 0,
          z: STAR_MIN_SCALE + Math.random() * (1 - STAR_MIN_SCALE),
        });
      }
    }

    function placeStar(star) {
      star.x = Math.random() * width;
      star.y = Math.random() * height;
    }

    function recycleStar(star) {
      var direction = "z";

      var vx = Math.abs(velocity.x),
        vy = Math.abs(velocity.y);

      if (vx > 1 || vy > 1) {
        var axis = void 0;

        if (vx > vy) {
          axis = Math.random() < vx / (vx + vy) ? "h" : "v";
        } else {
          axis = Math.random() < vy / (vx + vy) ? "v" : "h";
        }

        if (axis === "h") {
          direction = velocity.x > 0 ? "l" : "r";
        } else {
          direction = velocity.y > 0 ? "t" : "b";
        }
      }

      star.z = STAR_MIN_SCALE + Math.random() * (1 - STAR_MIN_SCALE);

      if (direction === "z") {
        star.z = 0.1;
        star.x = Math.random() * width;
        star.y = Math.random() * height;
      } else if (direction === "l") {
        star.x = -OVERFLOW_THRESHOLD;
        star.y = height * Math.random();
      } else if (direction === "r") {
        star.x = width + OVERFLOW_THRESHOLD;
        star.y = height * Math.random();
      } else if (direction === "t") {
        star.x = width * Math.random();
        star.y = -OVERFLOW_THRESHOLD;
      } else if (direction === "b") {
        star.x = width * Math.random();
        star.y = height + OVERFLOW_THRESHOLD;
      }
    }

    function resize() {
      scale = window.devicePixelRatio || 1;

      width = window.innerWidth * scale;
      height = window.innerHeight * scale;

      canvas.width = width;
      canvas.height = height;

      stars.forEach(placeStar);
    }

    function step() {
      context.clearRect(0, 0, width, height);

      update();
      render();

      requestAnimationFrame(step);
    }

    function update() {
      velocity.tx *= 0.96;
      velocity.ty *= 0.96;

      velocity.x += (velocity.tx - velocity.x) * 0.8;
      velocity.y += (velocity.ty - velocity.y) * 0.8;

      stars.forEach(function (star) {
        star.x += velocity.x * star.z;
        star.y += velocity.y * star.z;

        star.x += (star.x - width / 2) * velocity.z * star.z;
        star.y += (star.y - height / 2) * velocity.z * star.z;
        star.z += velocity.z;

        // recycle when out of bounds
        if (
          star.x < -OVERFLOW_THRESHOLD ||
          star.x > width + OVERFLOW_THRESHOLD ||
          star.y < -OVERFLOW_THRESHOLD ||
          star.y > height + OVERFLOW_THRESHOLD
        ) {
          recycleStar(star);
        }
      });
    }

    function render() {
      stars.forEach(function (star) {
        context.beginPath();
        context.lineCap = "round";
        context.lineWidth = STAR_SIZE * star.z * scale;
        context.strokeStyle =
          "rgba(255,255,255," + (0.5 + 0.5 * Math.random()) + ")";

        context.beginPath();
        context.moveTo(star.x, star.y);

        var tailX = velocity.x * 2,
          tailY = velocity.y * 2;
        context.lineTo(star.x + tailX, star.y + tailY);

        context.stroke();
      });
    }

    function movePointer(x, y) {
      if (typeof pointerX === "number" && typeof pointerY === "number") {
        var ox = x - pointerX,
          oy = y - pointerY;

        velocity.tx = velocity.tx + (ox / 8) * scale * (touchInput ? 1 : -1);
        velocity.ty = velocity.ty + (oy / 8) * scale * (touchInput ? 1 : -1);
      }

      pointerX = x;
      pointerY = y;
    }

    function onMouseMove(event) {
      touchInput = false;

      movePointer(event.clientX, event.clientY);
    }

    function onTouchMove(event) {
      touchInput = true;

      movePointer(event.touches[0].clientX, event.touches[0].clientY, true);

      event.preventDefault();
    }

    function onMouseLeave() {
      pointerX = null;
      pointerY = null;
    }
  });
</script>

<main>
  <a href="https://github.com/sovlookup">by SOVLOOKUP</a>
  <canvas bind:this={canvas} />
</main>

<style>
  main {
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: rgb(0, 0, 0);
    background-image: radial-gradient(
        circle at center,
        rgba(81, 68, 154, 0.13),
        transparent
      ),
      radial-gradient(circle at center, rgba(41, 196, 255, 0.13), transparent);
  }

  canvas {
    position: fixed;
    width: 100%;
    height: 100%;
  }

  a {
    position: absolute;
    bottom: 2vmin;
    right: 2vmin;
    color: rgba(255, 255, 255, 0.4);
    text-decoration: none;
  }

  a:hover {
    color: #fff;
  }
</style>

