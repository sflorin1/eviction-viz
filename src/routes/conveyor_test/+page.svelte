<script>
    import ConveyorBelt from "../../lib/ConveyorBelt.svelte";
    import TestPage from "../../lib/TestPage.svelte";
    import Step1 from "../../lib/Step1.svelte"


    let page = {source: Step1,
                width: 800,
                height: 800,
                fadeInTime: -2000,
                fadeOutTime: -1000
    }
    let frames = [page];
</script>


<ConveyorBelt frames = {frames}></ConveyorBelt>