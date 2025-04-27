<script>
    import ConveyorBelt from "../../lib/ConveyorBelt.svelte";
    import TestPage from "../../lib/TestPage.svelte";

    let page = {source: TestPage,
                width: 400,
                height: 200,
                fadeInTime: -2000,
                fadeOutTime: -1000
    }
    let frames = [page];
</script>


<ConveyorBelt frames = {frames}></ConveyorBelt>