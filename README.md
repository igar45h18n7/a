<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Sparkle Particles</title>
    <script src="https://cdn.jsdelivr.net/npm/p5@1.11.3/lib/p5.min.js"></script>
    <style>
      html,
      body {
        margin: 0;
        padding: 0;
        overflow: hidden;
        background: #05060d;
        cursor: none;
      }
      canvas {
        display: block;
      }
    </style>
  </head>
  <body>
    <script>
      const PARTICLE_COUNT = 180;
      const particles = [];
      let mouseGlow = 0;

      class Particle {
        constructor() {
          this.reset(true);
        }

        reset(scatter) {
          this.pos = scatter
            ? createVector(random(width), random(height))
            : createVector(mouseX + random(-40, 40), mouseY + random(-40, 40));
          this.vel = p5.Vector.random2D().mult(random(0.2, 1.4));
          this.acc = createVector();
          this.size = random(1.2, 4.8);
          this.hue = random([48, 52, 200, 210, 265, 280]);
          this.sat = random(20, 70);
          this.life = random(80, 220);
          this.maxLife = this.life;
          this.twinkle = random(TWO_PI);
          this.orbit = random(18, 90);
          this.orbitSpeed = random(0.018, 0.05) * (random() < 0.5 ? -1 : 1);
          this.angle = random(TWO_PI);
          this.lag = random(0.035, 0.12);
        }

        seekMouse() {
          const target = createVector(
            mouseX + Math.cos(this.angle) * this.orbit,
            mouseY + Math.sin(this.angle) * this.orbit * 0.62
          );
          const desired = p5.Vector.sub(target, this.pos);
          const dist = desired.mag();
          desired.setMag(constrain(map(dist, 0, 280, 0.4, 6.5), 0.4, 6.5));
          const steer = p5.Vector.sub(desired, this.vel);
          steer.limit(0.45);
          this.acc.add(steer);
          this.acc.add(p5.Vector.random2D().mult(0.08));
        }

        update() {
          this.angle += this.orbitSpeed;
          this.seekMouse();
          this.vel.add(this.acc);
          this.vel.mult(0.92);
          this.vel.limit(7.5);
          this.pos.add(this.vel);
          this.acc.mult(0);
          this.life -= 0.55;
          this.twinkle += 0.18;

          if (this.life <= 0 || this.pos.x < -40 || this.pos.x > width + 40 || this.pos.y < -40 || this.pos.y > height + 40) {
            this.reset(false);
          }
        }

        draw() {
          const fade = this.life / this.maxLife;
          const sparkle = 0.55 + 0.45 * Math.abs(Math.sin(this.twinkle));
          const alpha = 255 * fade * sparkle;
          const glow = this.size * (3.4 + sparkle * 2.2);

          noStroke();
          fill(this.hue, this.sat, 100, alpha * 0.12);
          circle(this.pos.x, this.pos.y, glow);

          fill(this.hue, this.sat * 0.4, 100, alpha * 0.55);
          circle(this.pos.x, this.pos.y, this.size * 2.1);

          fill(48, 10, 100, alpha);
          circle(this.pos.x, this.pos.y, this.size * 0.7);

          if (sparkle > 0.92) {
            stroke(48, 8, 100, alpha);
            strokeWeight(0.8);
            const spike = this.size * 4.5;
            line(this.pos.x - spike, this.pos.y, this.pos.x + spike, this.pos.y);
            line(this.pos.x, this.pos.y - spike, this.pos.x, this.pos.y + spike);
          }
        }
      }

      function setup() {
        createCanvas(windowWidth, windowHeight);
        colorMode(HSB, 360, 100, 100, 255);
        for (let i = 0; i < PARTICLE_COUNT; i++) {
          particles.push(new Particle());
        }
      }

      function draw() {
        drawingContext.globalCompositeOperation = "source-over";
        background(230, 40, 4, 38);

        drawingContext.globalCompositeOperation = "lighter";

        const speed = dist(mouseX, mouseY, pmouseX, pmouseY);
        mouseGlow = lerp(mouseGlow, constrain(speed * 2.2, 18, 90), 0.18);

        noStroke();
        fill(48, 25, 100, 18);
        circle(mouseX, mouseY, mouseGlow * 3.4);
        fill(210, 30, 100, 22);
        circle(mouseX, mouseY, mouseGlow * 1.8);
        fill(48, 5, 100, 80);
        circle(mouseX, mouseY, 5);

        for (const p of particles) {
          p.update();
          p.draw();
        }
      }

      function windowResized() {
        resizeCanvas(windowWidth, windowHeight);
      }

      function mousePressed() {
        for (let i = 0; i < 24; i++) {
          const burst = new Particle();
          burst.pos = createVector(mouseX, mouseY);
          burst.vel = p5.Vector.random2D().mult(random(3, 10));
          burst.orbit = random(8, 40);
          particles.push(burst);
          if (particles.length > PARTICLE_COUNT + 80) particles.shift();
        }
      }
    </script>
  </body>
</html>
