<script lang="ts">
  
  import DilutingTouchIcon from "./assets/Diluting_Touch_status_icon.png"
  import ErodingTouchIcon from "./assets/Eroding_Touch_status_icon.png"
  import ParalysingTouchIcon from "./assets/Paralysing_Touch_status_icon.png"
  import WastingTouchIcon from "./assets/Wasting_Touch_status_icon.png"
  import DebuffBorder from "./assets/Debuff_frame.png"
  import BackgroundImage from "./assets/LayeredKeyArt_HighRes2.png"

  const BASE_EMBRACE_MADNESS_DEBUFF_DURATION = 20000 // 20000ms = 20s 
  const startingDebuffs = Array.from({length:4},() => ({
    stacks: 0,
    remainingTime: 0,
    lastUpdate: performance.now(),
  }))

  let simulationIsRunning = $state(false)
  let EmbraceMadnessDebuffCycle = 2000 // 2s, how long it takes for a new debuff stack to apply
  let fasterExpirationOfDebuffs = $state(0)  
  let embraceMadnessDebuffLength = $derived(
    Math.round(BASE_EMBRACE_MADNESS_DEBUFF_DURATION / ( 1 + fasterExpirationOfDebuffs / 100 ))
  )
  let EmbraceMadnessDebuffs = $state(startingDebuffs)
  let sourcesOfFasterDebuffExpiration = $state(false)
  let debuffExplanationWindow = $state(false)

  $effect(() => {

    if (!simulationIsRunning) return 

    const interval = setInterval(() => {

      const i = Math.floor(Math.random() * 4)
      // const debuffs = EmbraceMadnessDebuffs

      EmbraceMadnessDebuffs[i] = {
        stacks: EmbraceMadnessDebuffs[i].stacks + 1 < 11 ? EmbraceMadnessDebuffs[i].stacks + 1 : EmbraceMadnessDebuffs[i].stacks ,
        remainingTime: embraceMadnessDebuffLength,
        lastUpdate: performance.now(),
      }

    }, EmbraceMadnessDebuffCycle)

    return () => clearInterval(interval)
  })

  $effect(() => {

    if (!simulationIsRunning) return

    let running = true

    const tick = () => {

      const now = performance.now()
      EmbraceMadnessDebuffs = EmbraceMadnessDebuffs.map((debuff) => {

        const elapsed = now - debuff.lastUpdate
        const remaining = Math.max(0, debuff.remainingTime - elapsed)

        if (remaining === 0 && debuff.remainingTime > 0) {
          return { ...debuff, stacks: 0, remainingTime: 0, lastUpdate: now }
        }
        return { ...debuff, remainingTime: remaining, lastUpdate: now }
      })

      if (running) requestAnimationFrame(tick)
    };

    requestAnimationFrame(tick)
    return () => (running = false)
  })

</script>

<main 
  class="max-w-screen w-full min-h-screen flex flex-col justify-center items-center bg-cover bg-center "
  style={`background-image: url('${BackgroundImage}')`}
>

  <div class="flex flex-col gap-2 mt-8">

    <h1 class="text-4xl text-white">
      Beacons of Madness debuff expiration Simulator
    </h1>

    <div class="flex justify-center gap-2">

      <button 
        class="p-2 border bg-gray-300 rounded-lg transition-all hover:scale-105 hover:cursor-pointer" 
        onclick={() => sourcesOfFasterDebuffExpiration = !sourcesOfFasterDebuffExpiration}
      >
        {sourcesOfFasterDebuffExpiration ? "Close" : "Show"} list of faster debuff expiration
      </button>

      <button 
        class="p-2 border bg-gray-300 rounded-lg transition-all hover:scale-105 hover:cursor-pointer" 
        onclick={() => debuffExplanationWindow = !debuffExplanationWindow}
      >
        {debuffExplanationWindow ? "Close" : "Show"} explanation of the debuffs
      </button>

    </div>

  </div>

  <div class="grow flex flex-col gap-4 justify-center items-center">
    <div class="flex flex-col bg-gray-500/80 border rounded-lg p-2">
      <div class="flex items-center">
        <input type="number" bind:value={fasterExpirationOfDebuffs} class="border rounded-md bg-white/50 p-2" />
        <span class="text-gray-200">% Faster expiration of debuffs</span>
      </div>
      <p class="text-gray-200">Duration of the debuffs: {embraceMadnessDebuffLength / 1000}s</p>
    </div>
    
    <div class="flex gap-2">

      <div class="flex flex-col gap-1">
        <div class="relative min-w-[98px] min-h-24 flex justify-center items-center" style={`background-image: url('${DebuffBorder}')`}>
          <div class="min-w-16 min-h-16" style={`background-image: url('${DilutingTouchIcon}')`}></div>
          <p class="absolute bottom-0 rounded-b-lg w-full bg-black/50 text-white flex justify-center">{EmbraceMadnessDebuffs[0].stacks}</p>
        </div>
        <p class="border rounded-lg w-full flex justify-center bg-gray-600 text-white">
          {(Math.round(EmbraceMadnessDebuffs[0].remainingTime) / 1000).toFixed(2)}s
        </p>
      </div>

      <div class="flex flex-col gap-1">
        <div class="relative min-w-[98px] min-h-24 flex justify-center items-center" style={`background-image: url('${DebuffBorder}')`}>
          <div class="min-w-16 min-h-16" style={`background-image: url('${ErodingTouchIcon}')`}></div>
          <p class="absolute bottom-0 rounded-b-lg w-full bg-black/50 text-white flex justify-center">{EmbraceMadnessDebuffs[1].stacks}</p>
        </div>
        <p class="border rounded-lg w-full flex justify-center bg-gray-600 text-white">
          {(Math.round(EmbraceMadnessDebuffs[1].remainingTime) / 1000).toFixed(2)}s
        </p>
      </div>

      <div class="flex flex-col gap-1">
        <div class="relative min-w-[98px] min-h-24 flex justify-center items-center" style={`background-image: url('${DebuffBorder}')`}>
          <div class="min-w-16 min-h-16" style={`background-image: url('${ParalysingTouchIcon}')`}></div>
          <p class="absolute bottom-0 rounded-b-lg w-full bg-black/50 text-white flex justify-center">{EmbraceMadnessDebuffs[2].stacks}</p>
        </div>
        <p class="border rounded-lg w-full flex justify-center bg-gray-600 text-white">
          {(Math.round(EmbraceMadnessDebuffs[2].remainingTime) / 1000).toFixed(2)}s
        </p>
      </div>

      <div class="flex flex-col gap-1">
        <div class="relative min-w-[98px] min-h-24 flex justify-center items-center" style={`background-image: url('${DebuffBorder}')`}>
          <div class="min-w-16 min-h-16" style={`background-image: url('${WastingTouchIcon}')`}></div>
          <p class="absolute bottom-0 rounded-b-lg w-full bg-black/50 text-white flex justify-center">{EmbraceMadnessDebuffs[3].stacks}</p>
        </div>
        <p class="border rounded-lg w-full flex justify-center bg-gray-600 text-white">
          {(Math.round(EmbraceMadnessDebuffs[3].remainingTime) / 1000).toFixed(2)}s
        </p>
      </div>
    </div>

    {#if simulationIsRunning}
      <div class="border-green-500 rounded-lg bg-green-700/90 text-green-500 p-2">
        Simulation is running
      </div>
    {:else}
      <div class="border-orange-500 rounded-lg bg-orange-700/90 text-orange-300 p-2">
        Simulation is paused
      </div>
    {/if}

    <div class="flex gap-2">

      <button class="p-2 border bg-gray-300 rounded-lg transition-all hover:scale-105 hover:cursor-pointer" onclick={() => simulationIsRunning = true}>
        Start Simulation
      </button>
      
      <button class="p-2 border bg-gray-300 rounded-lg transition-all hover:scale-105 hover:cursor-pointer" onclick={() => simulationIsRunning = false}>
        Pause Simulation
      </button>

      <button class="p-2 border bg-gray-300 rounded-lg transition-all hover:scale-105 hover:cursor-pointer" onclick={() => EmbraceMadnessDebuffs = startingDebuffs}>
        Reset
      </button>

    </div>
  </div>

  <div class="w-full text-gray-500 flex justify-center p-2">
    This website is not endorsed or affiliated with Grinding Gear Games.
  </div>

  <div class={`${sourcesOfFasterDebuffExpiration ? "translate-x-0" : "-translate-x-[550px]"} absolute p-4 left-0 top-0 bg-black/50 text-gray-400 w-[550px] h-screen flex flex-col gap-3 overflow-auto transition-all`}>

    <h2 class="text-2xl py-[50px]">Sources of faster expiration of debuffs:</h2>

    <p>- 100% from <a href="https://www.poewiki.net/wiki/Warped_Timepiece" target="_blank">Warped Timepiece</a> unique amulet</p>
    <p>- (80-100)% from <a href="https://www.poewiki.net/wiki/Stasis_Prison" target="_blank">Stasis Prison</a> unique armour</p>
    <p>- (80-100)% from a <a href="https://www.poewiki.net/wiki/Modifier:MercenaryModDebilitate" target="_blank">Infamous modifier</a> on a belt (not obtainable in 3.26)</p>
    <p>- 30% from <a href="https://www.poewiki.net/wiki/Runegraft_of_the_Warp" target="_blank">Runegraft of the Warp</a></p>
    <p>- (15-20)% from <a href="https://www.poewiki.net/wiki/The_Flow_Untethered" target="_blank">The Flow Untethered</a> unique belt</p>
    <p>- (15-20)% from <a href="https://www.poewiki.net/wiki/The_Torrent%27s_Reclamation" target="_blank">The Torrents Reclamation</a> unique belt</p>
    <p>- (15-20)% from a <a href="https://www.poewiki.net/wiki/Modifier:HasteDebuffsExpireFaster" target="_blank">Haste Watcher's Eye modifier</a></p>
    <p>- 15% from the <a href="https://www.poewiki.net/wiki/Window_of_Opportunity" target="_blank">Window of Opportunity</a> notable passive skill</p>
    <p>- 15% from a <a href="https://www.poewiki.net/wiki/Duration_Mastery" target="_blank">Duration Mastery</a></p>
    <p>- 15% from a <a href="https://web.poecdn.com/public/news/2025-10-27/BloodlinesAscendancy/Imposter_Delirious.png" target="_blank">small Delirious Bloodline noteable</a></p>
    <p>- 10% from the small cluster jewel <a href="https://www.poewiki.net/wiki/Frantic_Aspect" target="_blank">Frantic Aspect noteable</a></p>
    <p>- (0-??)% from Quality on the <a href="https://www.poewiki.net/wiki/Temporal_Rift" target="_blank">Temporal Rift</a> skill gem</p>
    <p class="mt-4">There might be more, this is just all I could find and remember.</p>

  </div>

  <div class={`${debuffExplanationWindow ? "translate-x-0" : "translate-x-[550px]"} absolute p-4 right-0 top-0 bg-black/50 text-gray-400 w-[550px] h-screen flex flex-col gap-3  transition-all`}>
  
    <h2 class="text-2xl py-[50px]">Explanation of the debuffs:</h2>

    <p>Glorious Madness inflicts a random Touched Debuff every 2 seconds: The touched debuffs are Diluting Touch, Eroding Touch, Paralysing Touch, Wasting Touch. Debuffs inflicted by Glorious Madness have a duration of 20 seconds, which is refreshed each time a stack is added, or would be added while the debuff is at maximum stacks. These debuffs can stack up to ten times. Debuff duration cannot be modified by skill duration.</p>

    <div class="flex flex-col gap-1">
      <div class="flex items-center gap-2">
        <img src={DilutingTouchIcon} alt="Diluting touch icon"/>
        <h3 class="text-xl">Diluting Touch</h3>
      </div>
      <p>9% reduced Flask Charges gained <span class="text-red-500">currently: {9 * EmbraceMadnessDebuffs[0].stacks}%</span></p>
      <p>9% reduced effect of Flasks on you <span class="text-red-500">currently: {9 * EmbraceMadnessDebuffs[0].stacks}%</span></p>
    </div>

    <div class="flex items-center gap-2">
      <img src={ErodingTouchIcon} alt="Eroding touch icon"/>
      <h3 class="text-xl">Eroding touch</h3>
    </div>
    <p>6% increased damage taken  <span class="text-red-500">currently: {6 * EmbraceMadnessDebuffs[1].stacks}%</span></p>

    <div class="flex items-center gap-2">
      <img src={ParalysingTouchIcon} alt="Paralysing touch icon"/>
      <h3 class="text-xl">Paralysing touch</h3>
    </div>
    <p>-6% action speed <span class="text-red-500">currently: {6 * EmbraceMadnessDebuffs[2].stacks}%</span></p>

    <div class="flex flex-col gap-1">
      <div class="flex items-center gap-2">
        <img src={WastingTouchIcon} alt="Wasting touch icon"/>
        <h3 class="text-xl">Wasting touch</h3>
      </div>
      <p>9% reduced Life Recovery Rate <span class="text-red-500">currently: {9 * EmbraceMadnessDebuffs[3].stacks}%</span></p>
      <p>9% reduced Energy Shield Recovery Rate <span class="text-red-500">currently: {9 * EmbraceMadnessDebuffs[3].stacks}%</span></p>
    </div>

  </div>
  
</main>

<style>
  a {
    text-decoration: underline;
  }

  a:hover {
    color: orange;
  }
</style>