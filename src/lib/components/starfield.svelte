<script>
  // This is where the fun begins!
  import { onMount } from "svelte";

  const star = { size: 1.5, speed: 2, tail: 100 }; // Shooting star properties
  const timeout = { min: 1000, max: 3000 }; // Timeout range for the shooting star

  /**
   * @type {HTMLCanvasElement}
   */
  let canvas;

  class ShootingStar {
    /**
     * @param {HTMLCanvasElement} canvas
     */
    constructor(canvas) {
      this.canvas = canvas;
      this.color = `hsl(45, 95%, 85%)`; // Cozy and warm yellowish color for the shooting star
      this.x = Math.random() * this.canvas.width;
      this.y = -star.size;
      this.angle = Math.PI * (0.25 + Math.random() * 0.5); // Random angle between 45° and 135°
    }

    /**
     * @param {any} ctx
     */
    draw(ctx) {
      this.x += star.speed * Math.cos(this.angle);
      this.y += star.speed * Math.sin(this.angle);
      this.drawStar(ctx);
      this.drawTail(ctx);
    }

    /**
     * @param {{ beginPath: () => void; arc: (arg0: number, arg1: number, arg2: number, arg3: number, arg4: number) => void; fillStyle: string; fill: () => void; }} ctx
     */
    drawStar(ctx) {
      ctx.beginPath();
      ctx.arc(this.x, this.y, star.size, 0, 2 * Math.PI);
      ctx.fillStyle = this.color;
      ctx.fill();
    }

    /**
     * @param {{ beginPath: () => void; moveTo: (arg0: number, arg1: number) => void; lineTo: (arg0: number, arg1: number) => void; createLinearGradient: (arg0: number, arg1: number, arg2: number, arg3: number) => any; strokeStyle: any; lineWidth: number; stroke: () => void; }} ctx
     */
    drawTail(ctx) {
      ctx.beginPath();
      ctx.moveTo(this.x, this.y);
      ctx.lineTo(
        this.x - star.tail * Math.cos(this.angle),
        this.y - star.tail * Math.sin(this.angle)
      );

      const gradient = ctx.createLinearGradient(
        this.x,
        this.y,
        this.x - star.tail * Math.cos(this.angle),
        this.y - star.tail * Math.sin(this.angle)
      );
      gradient.addColorStop(0, this.color);
      gradient.addColorStop(1, "transparent");

      ctx.strokeStyle = gradient;
      ctx.lineWidth = star.size * 2;
      ctx.stroke();
    }

    // Check if the shooting star is out of the canvas bounds
    isOutOfBounds() {
      return this.y > this.canvas.height + star.tail;
    }
  }

  // Initialize and run the animation when component mounts
  onMount(() => {
    const ctx = canvas.getContext("2d");
    canvas.width = window.innerWidth;
    canvas.height = window.innerHeight;

    /**
     * @type {any[]}
     */
    const shootingStar = [];

    // Generate a new shooting star and schedule the next one
    function createShootingStar() {
      if (shootingStar.length === 0) {
        // Ensure we don't oversaturate the canvas
        shootingStar.push(new ShootingStar(canvas)); // Add a new shooting star
        setTimeout(
          createShootingStar, // Schedule the next shooting star creation
          Math.floor(Math.random() * (timeout.max - timeout.min)) + timeout.min // Random timeout
        );
      }
    }

    // Update position and draw the next shooting star
    function updateShootingStar() {
      for (let i = shootingStar.length - 1; i >= 0; i--) {
        // Loop in reverse
        const star = shootingStar[i];
        star.draw(ctx);
        if (star.isOutOfBounds()) {
          shootingStar.pop(); // Remove the shooting star from the array
        }
      }
    }

    // Main animation loop
    function animate() {
      if (!ctx) return; // Check if ctx is null

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      createShootingStar();
      updateShootingStar();
      requestAnimationFrame(animate); // Request the next animation frame
    }

    animate(); // Start the animation loop
  });
</script>

<canvas
  bind:this={canvas}
  id="starfield"
  style="display:block;position:fixed;top:0;left:0;width:100%;height:100%;z-index:-1;pointer-events:none;background:black;"
/>
