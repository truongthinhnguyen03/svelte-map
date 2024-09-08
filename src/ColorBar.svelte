<script>
    export let colors = [];
    export let breaks = [];
    export let title = "Legend";
    export let hoveredValue = null;
    export let selectedValue = null;

    // Create a linear gradient string from the colors
    $: gradientString = `linear-gradient(to right, ${colors.join(', ')})`;

    // Format number with commas for thousands
    function formatNumber(num) {
        return num?.toLocaleString() ?? '';
    }

    function getBreakTransform(index) {
        if (index === 0) return 'translateX(0%)';
        if (index === breaks.length - 1) return 'translateX(-100%)';
        return 'translateX(-50%)';
    }

    function getTickPosition(value) {
        if (value === null || value < breaks[0] || value > breaks[breaks.length - 1]) {
            return null;
        }
        const percentage = (value - breaks[0]) / (breaks[breaks.length - 1] - breaks[0]) * 100;
        return `${percentage}%`;
    }

    $: hoveredTickPosition = getTickPosition(hoveredValue);
    $: selectedTickPosition = getTickPosition(selectedValue);
</script>

<div class="color-bar">
    <h3>{title}</h3>
    <div class="color-scale" style="background: {gradientString};">
        <div class="color-scale-inner">
            {#each breaks as breakValue, i}
                <span class="break-marker" style="left: {i * 100 / (breaks.length - 1)}%; transform: {getBreakTransform(i)};">
                    <span class="break-value">{formatNumber(breakValue)}</span>
                </span>
            {/each}
        </div>
        
        {#if hoveredTickPosition !== null}
            <div class="value-tick hovered" style="left: {hoveredTickPosition};">
                <div class="tick-line"></div>
                <div class="tick-value">{formatNumber(hoveredValue)}</div>
            </div>
        {/if}

        {#if selectedTickPosition !== null}
            <div class="value-tick selected" style="left: {selectedTickPosition};">
                <div class="tick-line"></div>
                <div class="tick-value">{formatNumber(selectedValue)}</div>
            </div>
        {/if}
    </div>
</div>

<style>
    .color-bar {
        padding: 10px;
        border-radius: 4px;
        position: absolute;
        top: 10px;
        left: 10px;
        z-index: 1000;
        width: 400px;
    }

    h3 {
        margin: 0 0 10px 0;
        font-size: 14px;
    }

    .color-scale {
        height: 20px;
        position: relative;
        border-radius: 2px;
        top: 10px;
    }

    .break-marker {
        position: absolute;
        top: -20px;
    }

    .break-value {
        font-size: 10px;
        white-space: nowrap;
    }

    .value-tick {
        position: absolute;
        bottom: -10px;
    }

    .tick-line {
        width: 2px;
        height: 30px;
        background-color: black;
    }

    .tick-value {
        position: absolute;
        top: 32px;
        left: 50%;
        transform: translateX(-50%);
        font-size: 10px;
        white-space: nowrap;
        font-weight: bold;
    }

    .hovered .tick-line {
        background-color: orange;
    }

    .selected .tick-line {
        background-color: black;
    }
</style>