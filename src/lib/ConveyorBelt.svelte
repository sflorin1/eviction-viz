<!--<script>
    let image = '/Users/samflorin/Documents/GitHub/eviction-viz/static/2b7d18b3f076964096123da411bbdefc.png'
    let cycle = true;
</script>
<img src =  '2b7d18b3f076964096123da411bbdefc.png' alt = 'test'>-->


<script>
    /**
     * TODO:
     *  Make more flexible (so gap and height and number of repeats and whatever are easily configurable)
    *     Make it so objects can come in on the conveyer belt (given fixed position)
    *     Make it so objects can fall down onto the conveyer belt (given fixed position)
    *     Make it so objects can appear and disappear (given fixed appear/disappear position)
    *     Maybe make it so objects can come up on conveyor belt, rise up to center of screen, 
    *          and then fall back down to conveyor belt and disappear
    */
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';
    import { fade } from 'svelte/transition';
  
    export let conveyorImage = 'conveyorline.svg';
    export let count = 100;
    export let gap = -180;
    export let slidingImage = 'house.svg'
    export let slidingImageStartingPos = 3000;
    export let fadingImage = 'nwbsu2jdeh6gemcm9xo0vhl62pyh.png';
    export let fadeInPos = -1000;
    export let fadeOutPos = -2000;
    export let frames = [];
    let fadeInOpacity = 0;
    let showImage;
    const scrollY = writable(0);
  
    onMount(() => {
      window.scrollTo(0, 0)
      const handleScroll = () => {
        scrollY.set(window.scrollY);
      };
  
      window.addEventListener('scroll', handleScroll);
      document.body.style.backgroundColor = "#FCE9E0";
      return () => window.removeEventListener('scroll', handleScroll);
    });
  
    let x = 0;
  
    scrollY.subscribe(value => {
      x = value * -0.5;
    });
    $:{
        showImage = x > fadeOutPos && x < fadeInPos;
        console.log(showImage, x);
        if (x > fadeInPos) {
        fadeInOpacity = 0;
    } else if (x < fadeOutPos) {
        fadeInOpacity = 0;
    } else {
        // Linear interpolation between fadeStart and fadeEnd
        fadeInOpacity = 1;
    }
    console.log(x);
    console.log(fadeInOpacity);
    }
  </script>
  

  <img class = 'background-image' src = {'box.svg'}/>
  {#each frames as frame}
    {#if (x>frame.fadeInTime) & (x<frame.fadeOutTime)}
      {console.log('image importing')}
      <div class = "fade-in" transition:fade={{ duration: 400 }}>
        <svelte:component 
        this={frame.source} 
        width={frame.width} 
        height={frame.height}
        />
     </div>
    {/if}
  {/each}
  <!--
  {#if showImage}
  <div class = "fade-in" style = "opacity: {fadeInOpacity}" transition:fade={{ duration: 400 }}>
    <img src = {fadingImage}/>
  
  </div>
  {/if}-->
  <div class="image-container">
    <div
      class="image-strip"
      style="transform: translateX({x}px); --gap: {gap}px"
    >
      {#each Array(count) as _, i}
        <img src={conveyorImage} alt="Repeated Image {i}" />
      {/each}
    </div>
   <!-- <div
    class="sliding-image"
    style="transform: translateX({x+slidingImageStartingPos}px);"
  >
  </div>-->
    </div>
    
  


  <style>
    .background-image{
        position: fixed;
        top: 583px;
        left: 700px;
        width: 100px;
        height: 100px;
        z-index: 1;
    }
    .fade-in{
        position: fixed;
        top: 0px;
        left: 400px;
    }
    .fade-in img{
        height: 300px;
        width: 300px;
        transition: opacity 0.3s ease, transform 0.05s ease-out;
    }
    .image-container {
    position: relative;
    height: 3000vh;
  }

  .image-strip {
    position: fixed;
    top: 650px;
    left: -100px;
    display: flex;
    
    transition: transform 0.05s ease-out;
    will-change: transform;
  }

  .image-strip img {
    height: auto;
    width: 5000px;
    margin-left: var(--gap);
  }
  .image-strip img:first-child {
    margin-left: 0;
    }
    .sliding-image {
  position: fixed;
  top: 320px; 
  width: 100px;
  height: auto;
  z-index: 2;
  transition: transform 0.05s ease-out;
}
.sliding-image img{
  width: 250px;
}
  </style>
  