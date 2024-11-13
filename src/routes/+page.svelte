<script lang="ts">
    import '$lib/styles/grid.css';
    import { onMount, tick } from "svelte";

    let wrapper: HTMLDivElement;

    const boxSizePx = 10;
    let numRows = $state(0);
    let numCols = $state(0);

    const initialCooperatorFraction = 0.999;
    let cost = $state(51);
    let benefit = $state(100);
    let intervalMs = $state(100);

    const directions = [
        [-1, 0],
        [1, 0],
        [0, -1],
        [0, 1],
        [-1, -1],
        [-1, 1],
        [1, -1],
        [1, 1],
    ]

    const initializeGrid = () => {
        numRows = Math.floor(wrapper.clientHeight / boxSizePx);
        numCols = Math.floor(wrapper.clientWidth / boxSizePx);

        wrapper.style.setProperty('--rows', String(numRows));
        wrapper.style.setProperty('--cols', String(numCols));
    }

    const calculatePayoffs = () => {
        const getPayoff = (cellStrat: string, neigbourStrat: string): number => {
            if (cellStrat === 'c') {
                if (neigbourStrat === 'c') {
                    return benefit - cost;
                } else {
                    return -cost;
                }
            } else {
                if (neigbourStrat === 'c') {
                    return benefit;
                } else {
                    return 0;
                }
            }
        }

        for (let i = 0; i < numRows; i++) {
            for (let j = 0; j < numCols; j++) {
                const cell = document.getElementById(`${i}-${j}`) as HTMLDivElement;
                const cellStrat = cell.getAttribute('data-strategy')!;
                let cellPayoff = 0;
                const neighbourStrategies: string[] = [];

                directions.forEach(([di, dj]) => {
                    const localCell = document.getElementById(`${i + di}-${j + dj}`);
                    if (localCell) neighbourStrategies.push(localCell.getAttribute('data-strategy')!);
                });

                neighbourStrategies.forEach((neighbourStrat) => {
                    cellPayoff += getPayoff(cellStrat, neighbourStrat);
                });
                cell.setAttribute('data-payoff', String(cellPayoff));
            }
        }
    }

    const updateStrategy = () => {
        for (let i = 0; i < numRows; i++) {
            for (let j = 0; j < numCols; j++) {
                const cell = document.getElementById(`${i}-${j}`) as HTMLDivElement;
                const cellStrat = cell.getAttribute('data-strategy')!;
                
                const neighbourStrategies: string[] = [cellStrat];
                const neighbourPayoffs: number[] = [Number(cell.getAttribute('data-payoff')!)];
                
                directions.forEach(([di, dj]) => {
                    const localCell = document.getElementById(`${i + di}-${j + dj}`);
                    if (localCell) {
                        neighbourStrategies.push(localCell.getAttribute('data-strategy')!);
                        neighbourPayoffs.push(Number(localCell.getAttribute('data-payoff')!));
                    }
                });

                const maxPayoff = Math.max(...neighbourPayoffs);
                const bestIndices = neighbourPayoffs
                    .map((value, index) => value === maxPayoff ? index : -1)
                    .filter((_, index) => index !== -1);
                for (const index of bestIndices) {
                    const neighbourStrat = neighbourStrategies[index];
                    cell.setAttribute('data-strategy', neighbourStrat);
                    if (cellStrat === neighbourStrat) {
                        break;
                    }
                }
            }
        }
    }

    onMount(() => {
        initializeGrid();
        window.addEventListener('resize', () => initializeGrid());

        return () => {
            window.removeEventListener('resize', () => initializeGrid());
        }
    })

    $effect(() => {
        const intervalId = setInterval(async () => {
            await tick();
            calculatePayoffs();
            await tick();
            updateStrategy();
        }, intervalMs);
        return () => clearInterval(intervalId);

        // (async () => {
        //     for (let i = 0; i < 10; i++) {
        //         await new Promise((r) => setInterval(r, 1000))
        //         await tick();
        //         calculatePayoffs();
        //         await tick();
        //         updateStrategy();
        //     }
        // })();
    })

    const handleReset = async () => {
        numRows = 0;
        numCols = 0;
        await tick();
        initializeGrid();
    }
</script>

<button on:click={handleReset} class={`absolute top-2 right-2 px-8 py-2 bg-neutral-700/10 rounded-md border border-neutral-500 text-neutral-500`}>
    Reset
</button>

<div
    bind:this={wrapper}
    class={`h-dvh w-dvw bg-neutral-500 grid`}
    style={`grid-template-rows: repeat(var(--rows), 1fr); grid-template-columns: repeat(var(--cols), 1fr);`}
>
    {#each Array(numRows * numCols) as _, index (index)}
    <!-- data-strategy={Math.random() < initialCooperatorFraction ? 'c' : 'd'} -->
        <div
            id={`${Math.floor(index / numCols)}-${Math.floor(index % numCols)}`}
            data-strategy={index === Math.floor(numRows * numCols / 2) ? 'd' : 'c'}
            data-payoff={0}
            class={`cell h-full w-full`}
        ></div>
    {/each}
</div>