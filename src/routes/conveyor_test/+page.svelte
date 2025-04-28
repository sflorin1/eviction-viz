<script>
    import ConveyorBelt from "../../lib/ConveyorBelt.svelte";
    import TestPage from "../../lib/TestPage.svelte";
    import Step1 from "../../lib/Step1.svelte"
    import EvictorMap from "../../lib/EvictorMap.svelte"


    let page1 = {source: Step1,
                width: 800,
                height: 800,
                fadeInTime: -2000,
                fadeOutTime: -1000
    }

    /**let page2 = {source: EvictorMap,
                width: 600,
                height: 600,
                fadeInTime: -3500,
                fadeOutTime: -2500
    }**/
    let frames = [page1];
</script>


<ConveyorBelt frames = {frames}></ConveyorBelt>