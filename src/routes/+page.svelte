<script lang="ts">
import { onMount } from 'svelte';
import { Application } from '@splinetool/runtime';
import { gsap } from 'gsap';

let ScrollTrigger;

import('gsap/ScrollTrigger').then(module => {
   ScrollTrigger = module.ScrollTrigger;
   gsap.registerPlugin(ScrollTrigger);


   // Any code relying on ScrollTrigger should be here
   gsap.to("#yourElement", {
      scrollTrigger: {
         // ... your ScrollTrigger properties here
      },
      x: 100,
      duration: 1
   });
});

let canvas: HTMLCanvasElement | null = null;

onMount(() => {
  if (canvas) {
    const app = new Application(canvas);
    app.load('https://prod.spline.design/FivctkfnUg7HaUsz/scene.splinecode').then(() => {
      const keyboard = app.findObjectByName("keyboard")!;
      
      gsap.set(keyboard.scale, { x: 1, y: 1, z: 1 });
      gsap.set(keyboard.position, { x: 110, y: 50 });

      let rotateKeyboard = gsap.to(keyboard.rotation, {
        y: Math.PI * 2 + keyboard.rotation.y,
        x: 0,
        z: 0,
        duration: 10,
        repeat: -1,
        ease: "none"
      });

      let rotationProgress = 0;
      let interval: NodeJS.Timeout | null = null;

      gsap.timeline({
        scrollTrigger: {
          trigger: "#part1",
          start: "top 60%",
          end: "bottom bottom",
          scrub: true,
          onEnter: () => {
            rotationProgress = rotateKeyboard.progress();

            interval = setInterval(() => {
              app.emitEvent("keyDown", "keyboard");
            }, 1500);

            rotateKeyboard.pause();
            gsap.to(keyboard.rotation, {
              y: Math.PI / 12,
              duration: 1
            });
          },
          onLeaveBack: () => {
            const newProgress = keyboard.rotation.y / (Math.PI * 2);
            rotateKeyboard.progress(newProgress).resume();
            if (interval) {
              clearInterval(interval);
            }
          }
        }
      })
      .to(keyboard.rotation, { x: -Math.PI / 14, z: Math.PI / 36 }, 0)
      .to(keyboard.position, { x: -500, y: -200 }, 0)
      .to(keyboard.scale, { x: 3, y: 3, z: 3 }, 0);

      gsap.timeline({
        onComplete: () => {
          if (interval) {
            clearInterval(interval);
          }
          app.emitEvent("mouseDown", "keyboard");
        },
        scrollTrigger: {
          trigger: "#part2",
          start: "top bottom",
          end: "center bottom",
          scrub: true
        }
      })
      .to(keyboard.rotation, { x: Math.PI / 36, y: -Math.PI / 10 }, 0)
      .to(keyboard.position, { x: 150, y: 50 }, 0)
      .to(keyboard.scale, { x: 0.8, y: 0.8, z: 0.8 }, 0);

      gsap.timeline({
        scrollTrigger: {
          trigger: "#part3",
          start: "top bottom",
          end: "bottom bottom",
          scrub: true
        }
      })
      .to(keyboard.position, { x: 0, y: 0 }, 0);
    });

    // Background progress bar
    function animateBar(triggerElement: string, onEnterWidth: string, onLeaveBackWidth: string) {
      gsap.to(".bar", {
        scrollTrigger: {
          trigger: triggerElement,
          start: "top center",
          end: "bottom bottom",
          scrub: true,
          onEnter: () => {
            gsap.to(".bar", {
              width: onEnterWidth,
              duration: 0.2,
              ease: "none"
            });
          },
          onLeaveBack: () => {
            gsap.to(".bar", {
              width: onLeaveBackWidth,
              duration: 0.2,
              ease: "none"
            });
          }
        }
      });
    }
    
    animateBar("#part1", "35%", "0%");
    animateBar("#part2", "65%", "35%");
    animateBar("#part3", "100%", "65%");

    // Keyboard text effect
    const keys = document.querySelectorAll<HTMLElement>(".key");
    function pressRandomKey() {
      const randomKey = keys[Math.floor(Math.random() * keys.length)];

      randomKey.style.animation = "pressDown 0.2s ease-in-out";

      randomKey.onanimationend = () => {
        randomKey.style.animation = "";
        setTimeout(pressRandomKey, 100 + Math.random() * 300);
      };
    }

    pressRandomKey();
  }
});
</script>

<style>
    :global(body) {
    background-color: rgb(17, 16, 29);

    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    position: relative;
    margin: 0;
    padding: 20px;
    overflow-x: hidden;
    }

    * {
    font-family: "Bricolage Grotesque", sans-serif;
    }

    .flex {
    display: flex;
    }

    .flex.row {
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    }
    
    .flex.column {
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    }

    .canvas-cont {
    position: fixed;
    z-index: -1;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    pointer-events: none;
    }

    #canvas3d {
    width: 100%;
    height: 100%;
    }

    nav {
    width: 100%;
    }

    p {
    margin: 0;
    font-size: 18px;
    font-weight: 200;
    color: white;
    }

    .bar {
    height: 100%;
    width: 0%;
    position: absolute;
    right: 0;
    top: 0;
    background-color: black;
    z-index: -5;
    }

    h1 {
    font-size: 10vw;
    position: absolute;
    top: calc(100vh - 50vw);
    font-weight: 800;
    left: 20px;
    line-height: 1;
    letter-spacing: -5px;
    color: white;
    }

    h2 {
    font-size: 40px;
    line-height: 1;
    color: white;
    }

    #hero {
    height: 100vh;
    }

    #part1 {
    width: 100%;
    height: 150vh;
    }

    .part1-info,
    .part2-info {
    width: 30%;
    gap: 40px;
    padding: 1%;
    }

    button {
    border: 1px solid rgba(255, 255, 255, 0.4);
    background-color: transparent;
    font-size: 18px;
    font-weight: 200;
    width: 100%;
    border-radius: 10px;
    padding: 20px 0;
    cursor: pointer;
    color: white;
    }

    button:hover {
    border: 1px solid white;
    }

    #part2 {
    height: 150vh;
    width: 100%;
    }

    #part3 {
    height: 120vh;
    justify-content: flex-end !important;
    align-items: center !important;
    margin-bottom: 50px;
    }


    .scroll-icon {
    height: 50px;
    width: 35px;
    border: 1px solid white;
    border-radius: 100px;
    padding: 5px 14px 20px 14px;
    box-sizing: border-box;
    }

    .scroll {
    height: 10px;
    width: 5px;
    border-radius: 10px;
    background-color: white;
    }

    .key {
    display: inline-block;
    letter-spacing: -2vw;
    transition: transform 0.2s;
    }

    @keyframes pressDown {
    0%,
    100% {
        transform: translateY(0);
    }
    50% {
        transform: translateY(10px);
    }
    }

    @media screen and (max-width: 1000px) {
    .canvas-cont {
        width: 200vw;
        transform: scale(0.5);
        left: auto;
        top: 25vh;
    }

    h1 {
        top: 20vw;
        font-size: 15vw;
    }

    .part1-info,
    .part2-info {
        width: 100%;
    }

    h2 {
        font-size: 5vw;
    }
    }
  </style>

<nav class="flex column alighn-center">
    <!--  made-up logo  -->
    <svg width="30" height="30" viewBox="0 0 76 76" fill="none" xmlns="http://www.w3.org/2000/svg">
      <path d="M10.9082 30.6362C11.2726 29.4923 12.3987 28.7689 13.5906 28.9132L48.4252 33.1282C49.5306 33.262 50.4143 34.1107 50.5926 35.2098L54.1189 56.949C54.3837 58.5815 53.0187 60.0167 51.375 59.834L6.33996 54.8289C4.77165 54.6546 3.75507 53.0888 4.23406 51.5853L10.9082 30.6362Z" fill="#D9D9D9" stroke="black" stroke-width="5"/>
      <path d="M64.6523 16.1406C63.6548 11.8386 57.7033 11.3976 56.0826 15.5056L49.4996 32.1914C49.2944 32.7115 49.1879 33.2652 49.1856 33.8243L49.0982 54.9647C49.0786 59.6996 55.4449 61.249 57.6034 57.0347L68.2326 36.2818C68.7171 35.336 68.8512 34.2492 68.6111 33.214L64.6523 16.1406Z" fill="black" stroke="black"/>
      <path d="M32.3651 8.90785L59.7453 12.4201C61.6291 12.6618 62.5661 14.833 61.4496 16.3694L48.8508 33.7069C48.3724 34.3653 47.6034 34.7496 46.7896 34.7369L27.223 34.4331C27.0207 34.43 26.8194 34.4023 26.6237 34.3506L13.5987 30.9119C11.7123 30.4139 11.098 28.0433 12.5051 26.6917L30.3152 9.58454C30.8609 9.06039 31.6146 8.81158 32.3651 8.90785Z" fill="#D9D9D9" stroke="black" stroke-width="5"/>
    </svg>

  </nav>
  
  <div class="bar"></div>

  <div class="canvas-cont">
    <canvas bind:this={canvas} id="canvas3d"></canvas>
  </div>
  
  <div id="hero" class="flex row">
    <h1>Introducing<br>
      <div class="Ergokeys">
        <span class="key">E</span>
        <span class="key">R</span>
        <span class="key">G</span>
        <span class="key">O</span>
        <span class="key">1</span>
      </div>
    </h1>
  </div>
  
  <div id="part1" class="flex row">
    <div></div>
    <div class="part1-info flex column">
      <h2>PLAY LIKE A PRO.</h2>
      <p>With these keyboards, you'll get proper bounce, a bit of *click*, and lots of satisfaction.</p>
      <button>Make a keyboard - It's easy!</button>
    </div>
  </div>
  
  <div id="part2" class="flex row">
    <div class="part2-info flex column">
      <h2>CUSTOMIZE ALL THE WAY.</h2>
      <p>It's all yours! Change the colors as you like. Make them purple, green, red, anything.</p>
      <button>Customize a Keyboard</button>
    </div>
    <div></div>
  </div>
  
  <div id="part3" class="flex column">
    <div class="scroll-icon">
      <div class="scroll"></div>
    </div>
  </div>