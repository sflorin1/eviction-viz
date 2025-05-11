<script>
export let width = 100;  // default value if none passed
    export let height = 100;
    let scaleX = 1;
    let scaleY = 1;
    $:{const baseWidth = 1000;
        const baseHeight = 1000;
        scaleX = width / baseWidth;
        scaleY = height / baseHeight;
    }
</script>
<div class="resized-container" style="transform: scale({scaleX}, {scaleY}); transform-origin: top left;">
<a href = 'https://www.google.com/'>Click here to learn more about serial evictors in Boston</a>
</div>
<style>
.resized-container {
        width: 1000px; /* base width */
        height: 1000px; /* base height */
        overflow: scroll; /* optional: hide anything overflowing */
    }
</style>
