<script>
  import Siema from "siema";
  import { onMount, createEventDispatcher } from "svelte";

  export let perPage = 2;
  export let loop = false;
  export let autoplay = 0;
  export let duration = 200;
  export let easing = "ease-out";
  export let startIndex = 0;
  export let draggable = true;
  export let multipleDrag = true;
  export let dots = true;
  export let controls = true;
  export let threshold = 20;
  export let rtl = false;
  let currentIndex = startIndex;

  let siema;
  let controller;
  let timer;

  const dispatch = createEventDispatcher();

  $: pips = controller ? controller.innerElements : [];
  $: currentPerPage = controller ? controller.perPage : perPage;
  $: totalDots = controller
    ? Math.ceil(controller.innerElements.length / currentPerPage)
    : [];

  onMount(() => {
    controller = new Siema({
      selector: siema,
      perPage: typeof perPage === "object" ? perPage : Number(perPage),
      loop,
      duration,
      easing,
      startIndex,
      draggable,
      multipleDrag,
      threshold,
      rtl,
      onChange: handleChange,
    });

    if (autoplay) {
      timer = setInterval(right, autoplay);
    }

    return () => {
      autoplay && clearInterval(timer);
      controller.destroy();
    };
  });

  export function isDotActive(currentIndex, dotIndex) {
    if (currentIndex < 0) currentIndex = pips.length + currentIndex;
    return (
      currentIndex >= dotIndex * currentPerPage &&
      currentIndex < dotIndex * currentPerPage + currentPerPage
    );
  }

  export function left() {
    controller.prev();
  }

  export function right() {
    controller.next();
  }

  export function go(index) {
    controller.goTo(index);
  }

  export function pause() {
    clearInterval(timer);
  }

  export function resume() {
    if (autoplay) {
      timer = setInterval(right, autoplay);
    }
  }

  function handleChange(event) {
    currentIndex = controller.currentSlide;

    dispatch("change", {
      currentSlide: controller.currentSlide,
      slideCount: controller.innerElements.length,
    });
  }
</script>

<div class="carousel">
  <div class="slides" bind:this={siema}>
    <slot />
  </div>
  {#if controls}
    <!-- <button class="left" on:click={left} aria-label="left">
			<slot name="left-control" />
		</button> -->
    <button class="right" on:click={right} aria-label="right">
      <slot name="right-control" />
    </button>
  {/if}
  {#if dots}
    <ul>
      {#each { length: totalDots } as _, i}
        <li
          on:click={() => go(i * currentPerPage)}
          class={isDotActive(currentIndex, i) ? "active" : ""}
        />
      {/each}
    </ul>
  {/if}
</div>

<style>
  .carousel {
    position: relative;
    width: 100%;
    justify-content: center;
    align-items: center;
    cursor: grab;
  }

  button {
    position: absolute;
    width: 40px;
    height: 40px;
    top: 50%;
    z-index: 50;
    margin-top: -20px;
    border: none;
    background-color: transparent;
  }

  button:focus {
    outline: none;
  }
  /* 
	.left {
		left: -20px;
	} */

  .right {
    right: -20px;
    position: fixed;
    position: absolute;
    right: 0px;
    height: 100%;
    top: 0;
    display: flex;
    right: 0px;
    padding: 5px;
    background: #192636;
    box-shadow: -46px 4px 88px 0px rgba(0, 0, 0, 1);
    cursor: pointer;
  }

  .right:hover {
    background: #66f;
    transition: 400ms;
  }

  .right :global(.control) {
    height: 100%;
  }

  .right :global(.control svg) {
    border: none;
  }

  ul {
    list-style-type: none;
    position: absolute;
    display: flex;
    justify-content: center;
    width: 100%;
    margin-top: 10px;
    padding: 0;
  }

  ul li {
    margin: 6px;
    border-radius: 100%;
    background-color: rgba(255, 255, 255, 0.5);
    height: 8px;
    width: 8px;
  }

  ul li:hover {
    background-color: rgba(255, 255, 255, 0.85);
  }

  ul li.active {
    background-color: rgba(255, 255, 255, 1);
  }

  .slides {
    width: 100%;
    padding-right: 0px;
    margin-right: 0px;
    /* overflow: visible !important; */
    padding-top: 30px;
    cursor: grab;
  }

  .slides > :global(div) {
    display: grid;
    column-count: 3;
    column-fill: auto;
    height: 60vh;
    padding-left: 10px;
    grid-template-rows: 220px;
    grid-template-columns: none;
    grid-auto-flow: column;
    grid-template-rows: 1fr 1fr;
    grid-auto-flow: column;
    /* grid-template-columns: 1fr 1fr 1fr 1fr 1fr;
		 */
    grid-auto-columns: 11%;
  }
</style>
