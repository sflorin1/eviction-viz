<script>
    import { onMount } from 'svelte';
    /**onMount(() => {
       document.body.style.backgroundColor = "#FFFFFF";   
    });*/

    export let width = 100;  // default value if none passed
    export let height = 100;
    let scaleX = 1;
    let scaleY = 1;

    onMount(() => {
        // Optional: if you want white background
        // document.body.style.backgroundColor = "#FFFFFF";   
    });

    // Calculate scaling based on 1000x1000 base size (or whatever your design base is)
    $: {
        const baseWidth = 1000;
        const baseHeight = 1000;
        scaleX = width / baseWidth;
        scaleY = height / baseHeight;
    }
</script>
<div class="resized-container" style="transform: scale({scaleX}, {scaleY}); transform-origin: top left;">
<p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Dolores amet iusto placeat maiores quia! Nostrum ad vitae ratione rem nisi exercitationem quasi? Eaque mollitia ea, esse nobis odio hic tempora.
</p>
<img src = 'house.svg'/>
</div>
<style>
    .resized-container {
        width: 1000px; /* base width */
        height: 1000px; /* base height */
        overflow: hidden; /* optional: hide anything overflowing */
    }
    img {
        max-width: 100%;
        height: auto;
    }
    </style>