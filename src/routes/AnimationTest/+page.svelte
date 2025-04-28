<script>
    import MainTitle from "../../lib/Title.svelte";
    import StaticBelt from "../../lib/StaticBelt.svelte";
    import Step1 from "../../lib/Step1.svelte";
    import Step2 from "../../lib/Step2.svelte";
    import Step3 from "../../lib/Step3.svelte";
    import Step4 from "../../lib/Step4.svelte";
    import Step5 from "../../lib/Step5.svelte";
    import Step6 from "../../lib/Step6.svelte";
    
    // Directly test if the components work
    let testMainTitle = true;
    let testStep1 = false;
    
    // For StaticBelt
    let frames = [
      {
        source: MainTitle,
        width: 800,
        height: 800,
        style: "transform: translateY(-100px);" // Move Step1 frame 150px higher
      },
      {
        source: Step1,
        width: 1300,
        height: 650,
        style: "transform: translateY(-50px);" // Move Step1 frame 150px higher
      },

      {
        source: Step2,
        width: 1400,
        height: 700,
        style: "transform: translateY(-50px);" // Move Step1 frame 150px higher
      },

      {
        source: Step3,
        width: 1400,
        height: 700,
        style: "transform: translateY(-70px);" // Move Step1 frame 150px higher
      },

      {
        source: Step4,
        width: 1400,
        height: 700,
        style: "transform: translateY(-70px);" // Move Step1 frame 150px higher
      },

      {
        source: Step5,
        width: 1400,
        height: 700,
        style: "transform: translateY(-70px);" // Move Step1 frame 150px higher
      },

      {
        source: Step6,
        width: 1400,
        height: 700,
        style: "transform: translateY(-70px);" // Move Step1 frame 150px higher
      }
    ];
   </script>
   
   <!-- Use StaticBelt with debugging -->
   <StaticBelt frames={frames} debug={false}></StaticBelt>