<script lang="ts">
    import { invoke } from "@tauri-apps/api/tauri";
    import { register, unregisterAll } from '@tauri-apps/api/globalShortcut';
    import Timer from "../components/Timer.svelte";
    import {onDestroy} from "svelte";

    let smallCamp: Timer;
    let mediumCamp: Timer;
    let hardCamp: Timer
    let slotMachine: Timer;

    register('F1', () => {
        smallCamp.toggleTimer();
    });

    register('F2', () => {
        mediumCamp.toggleTimer();
    });

    register('F3', () => {
        hardCamp.toggleTimer();
    });

    register('F4', () => {
        slotMachine.toggleTimer();
    });

    onDestroy(() => {
        unregisterAll();
    })
</script>

<div id="container" class="flex flex-wrap gap-y-4 gap-x-4 box-border justify-evenly items-center p-6 h-dvh">
    <Timer title="Easy Camp (2)" duration={60 * 4} bind:this={smallCamp}></Timer>
    <Timer title="Medium Camp (7)" duration={60 * 6} bind:this={mediumCamp}></Timer>
    <Timer title="Hard Camp (7)" duration={60 * 8} bind:this={hardCamp}></Timer>
    <Timer title="Slot Machine (10)" duration={60 * 5} bind:this={slotMachine}></Timer>
</div>

<style>
    @media (max-aspect-ratio: 3/4) {
        #container {
            flex-wrap: nowrap;
            flex-direction: column;
        }
    }

    :root {
        width: 100vw;
        height: 100vh;
        padding: 0;
        margin: 0;
        background: rgb(18, 18, 20);
        background: radial-gradient(circle, rgb(18, 18, 20) 0%, rgb(9, 9, 10) 100%);
        overflow: hidden;
        font-family: Inter, sans-serif;
    }
</style>
