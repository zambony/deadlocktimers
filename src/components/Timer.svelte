<script>
    import { onDestroy } from 'svelte';

    export let duration = 60; // Duration in seconds
    export let title = 'Timer'; // Title of the timer

    let timeLeft = duration;
    let isRunning = false;
    let interval;
    let startTime = 0;

    let tanColor = "#EFDEC0FF";

    $: progress = 1 - timeLeft / duration;
    $: minutes = Math.floor(timeLeft / 60);
    $: seconds = timeLeft % 60;
    $: flashing = false;

    // Calculate SVG parameters
    const radius = 45;
    const circumference = 2 * Math.PI * radius;
    const notchSize = 90; // 2 degrees
    const startAngle = 180 + notchSize / 2;
    const notchOffset = (notchSize / 360) * circumference;

    function polarToCartesian(centerX, centerY, radius, angleInDegrees) {
        const angleInRadians = (angleInDegrees - 90) * Math.PI / 180.0;
        return {
            x: centerX + (radius * Math.cos(angleInRadians)),
            y: centerY + (radius * Math.sin(angleInRadians))
        };
    }

    function describeArc(x, y, radius, startAngle, endAngle) {
        const start = polarToCartesian(x, y, radius, endAngle);
        const end = polarToCartesian(x, y, radius, startAngle);
        const largeArcFlag = endAngle - startAngle <= 180 ? "0" : "1";
        return [
            "M", start.x, start.y,
            "A", radius, radius, 0, largeArcFlag, 0, end.x, end.y
        ].join(" ");
    }

    let endAngle = startAngle + 360 - notchSize;

    $: backgroundPath = describeArc(50, 50, radius, startAngle + (360 - notchSize) * progress, endAngle);
    $: progressPath = describeArc(50, 50, radius, startAngle, startAngle + (360 - notchSize) * progress);

    export function startTimer() {
        isRunning = true;
        flashing = false;
        startTime = new Date().getTime();
        timeLeft = (new Date().getTime() - startTime);
        timeLeft = duration - Math.floor(timeLeft / 1000);
        interval = setInterval(() => {
            let elapsed = (new Date().getTime() - startTime);
            elapsed = Math.floor(elapsed / 1000);
            timeLeft = duration - elapsed;
            if (elapsed >= duration) {
                stopTimer();
                flashing = true
                setTimeout(() => {flashing = false;}, 2000);
            }
        }, 1000);

    }
    export function stopTimer() {
        isRunning = false;
        clearInterval(interval);
        timeLeft = duration;
    }

    export function toggleTimer() {
        if (isRunning) {
            stopTimer();
        } else {
            startTimer();
        }
    }

    onDestroy(() => {
        clearInterval(interval);
    });
</script>

<div id="timer" class="flex flex-col justify-center content-center items-center text-[#202023] h-[calc(50vmin-2.5rem)] aspect-square">
    <div class="relative h-full">
        <svg viewBox="0 0 100 100" class:flasher={flashing} class="h-full">
            <path
                    d={backgroundPath}
                    fill="none"
                    stroke={tanColor}
                    stroke-width="10"
            />
            <path
                    d={progressPath}
                    fill="none"
                    stroke="#7dbd1e"
                    stroke-width="10"
            />
        </svg>
        <div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 space-y-2">
            <div id="count" class="font-bold text-center text-[#EFDEC0FF] text-[1em]">
                {minutes.toString().padStart(2, '0')}:{seconds.toString().padStart(2, '0')}
            </div>
            <div id="label" class="text-center text-[0.6em] text-[#EFDEC0FF]">{title}</div>
        </div>
    </div>
    <button on:click={toggleTimer} class="p-1 text-[#EFDEC0FF] text-center w-1/3 border-4 border-[#EFDEC0FF] rounded-full hover:bg-[#7dbd1e] hover:text-black hover:border-[#7dbd1e] transition-all duration-200 font-medium !text-[0.5em]">
        {isRunning ? 'Stop' : 'Start'}
    </button>
</div>

<style>
    #timer {
        font-size: calc((50vmin - 2.5rem) / 8);
    }

    .flasher path {
        animation: greenFlash 0.5s ease 0s infinite normal forwards;
    }

    @media (max-aspect-ratio: 3/4) {
        #timer {
            width: unset;
            height: min(calc(25vh - 1.5rem), calc(100vw - 3rem));
            font-size: calc(min(calc(25vh - 1.5rem), calc(100vw - 3rem)) / 8);
        }
    }

    @media (min-aspect-ratio: 4/3) {
        #timer {
            width: unset;
            height: min(calc(25vw - 1.5rem), calc(100vh - 3rem));
            font-size: calc(min(calc(25vw - 1.5rem), calc(100vh - 3rem)) / 8);
        }
    }

    @keyframes greenFlash {
        0% {
            stroke: #EFDEC0FF;
        }

        50% {
            stroke: #7dbd1e;
        }

        100% {
            stroke: #EFDEC0FF;
        }
    }
</style>