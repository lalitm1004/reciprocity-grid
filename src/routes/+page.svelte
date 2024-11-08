<script lang="ts">
    import '$lib/styles/grid.css';
    import { onMount, tick } from "svelte";

    let wrapper: HTMLDivElement;

    const boxSizePx = 10;
    let numRows = $state(1);
    let numCols = $state(1);

    const initialCooperatorFraction = 0.66;
    const defectionReward = 1.59;
    const intervalMs = 100;

    const initializeGrid = async () => {
        numRows = Math.floor(wrapper.clientHeight / boxSizePx);
        numCols = Math.floor(wrapper.clientWidth / boxSizePx);

        wrapper.style.setProperty('--rows', numRows as unknown as string);
        wrapper.style.setProperty('--cols', numCols as unknown as string);
    }

    const calculatePayoffs = () => {
        for (let i = 0; i < numRows; i++) {
            for (let j = 0; j < numCols; j++) {
                const cell = document.getElementById(`${i}-${j}`) as HTMLDivElement;
                const neighbourStrategies: string[] = [];

                const directions = [
                    [-1, 0],  // UU
                    [-1, 1],  // UR
                    [0, 1],   // RR
                    [1, 1],   // DR
                    [1, 0],   // DD
                    [1, -1],  // DL
                    [0, -1],  // LL
                    [-1, -1]  // UL
                ];

                directions.forEach(([di, dj]) => {
                    const localCell = document.getElementById(`${i + di}-${j + dj}`);
                    if (localCell) neighbourStrategies.push(localCell.getAttribute('data-cooperator') ?? 'false');
                });

                const numCooperators = neighbourStrategies.filter(e => e === 'true').length;
                const payoff = cell.getAttribute('data-cooperator') === 'true' ? numCooperators : numCooperators * defectionReward;
                cell.setAttribute('data-payoff', payoff as unknown as string);
            }
        }
    }

    const updateStrategy = () => {
        for (let i = 0; i < numRows; i++) {
            for (let j = 0; j < numCols; j++) {
                const cell = document.getElementById(`${i}-${j}`) as HTMLDivElement;
                const cellStrat = cell.getAttribute('data-cooperator')!;
                const neighbourStrategies: string[] = [];
                const neighbourPayoffs: number[] = []

                const directions = [
                    [-1, 0],  // UU
                    [-1, 1],  // UR
                    [0, 1],   // RR
                    [1, 1],   // DR
                    [1, 0],   // DD
                    [1, -1],  // DL
                    [0, -1],  // LL
                    [-1, -1]  // UL
                ];

                directions.forEach(([di, dj]) => {
                    const localCell = document.getElementById(`${i + di}-${j + dj}`);
                    if (localCell) {
                        neighbourPayoffs.push(Number.parseFloat(localCell.getAttribute('data-payoff')!));
                        neighbourStrategies.push(localCell.getAttribute('data-cooperator') as unknown as string);
                    }
                });

                const maxPayoff = Math.max(...neighbourPayoffs);
                const bestIndices = neighbourPayoffs
                    .map((value, index) => (value === maxPayoff ? index : -1))
                    .filter(index => index !== -1);
                if (bestIndices.length > 1) {
                    for (const index of bestIndices) {
                        const neighbourStrat = neighbourStrategies[index];
                        cell.setAttribute('data-cooperator', neighbourStrat);
                        if (cellStrat === neighbourStrat) {
                            break;
                        }
                    }
                } else {
                    cell.setAttribute('data-cooperator', neighbourStrategies[bestIndices[0]])
                }
            }
        }
    }

    onMount(() => {
        initializeGrid()
        window.onresize = () => initializeGrid();

        const intervalId = setInterval(async () => {
            calculatePayoffs();
            await tick();
            updateStrategy();
        }, intervalMs)
    })
</script>

<div bind:this={wrapper} class={`h-dvh w-dvw grid`} style={`grid-template-rows: repeat(var(--rows), 1fr); grid-template-columns: repeat(var(--cols), 1fr);`}>
    {#each Array(numRows * numCols) as item, index (index)}
        <div id={`${Math.floor(index / numCols)}-${Math.floor(index % numCols)}`} data-cooperator={Math.random() < initialCooperatorFraction} data-payoff={0} class={`cell`}></div>
    {/each}
</div>