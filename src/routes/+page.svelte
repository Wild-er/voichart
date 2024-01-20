<script>
    import { onMount } from 'svelte';
    import { Line } from 'svelte-chartjs';
    import Chart from 'chart.js/auto';
    import 'chartjs-adapter-date-fns'; // Ensure this is installed and imported
    import { CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend, TimeScale } from 'chart.js'; // Import required modules

    Chart.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend, TimeScale); // Register the required modules


    let humbleLPInfo;
    let nomadexLPInfo;
    let priceHistory = [];

    const chartOptions = {
        type: 'line',
        data: {
            datasets: [
                {
                    label: 'Humble LP Price',
                    borderColor: 'rgb(75, 192, 192)',
                    fill: false,
                },
                {
                    label: 'Nomadex LP Price',
                    borderColor: 'rgb(255, 99, 132)',
                    fill: false,
                }
            ]
        },
        options: {
            scales: {
                x: {
                    type: 'time',
                    time: {
                        unit: 'minute',
                        displayFormats: {
                            minute: 'h:mm a' // Format of the time
                        }
                    },
                    title: {
                        display: true,
                        text: 'Time'
                    }
                },
                y: {
                    beginAtZero: true,
                    title: {
                        display: true,
                        text: 'Price in VIA'
                    }
                }
            },
            responsive: true,
            maintainAspectRatio: false
        }
    };

    async function fetchData() {
        const response = await fetch('/api');
        if (response.ok) {
            const data = await response.json();
            humbleLPInfo = data.humbleLPInfo;
            nomadexLPInfo = data.nomadexLPInfo;
            priceHistory = data.priceHistory;

            // Update chart data
            const humbleData = priceHistory.filter(p => p.dex === 'Humble').map(p => ({
                x: new Date(p.timestamp),
                y: p.price
            }));
            const nomadexData = priceHistory.filter(p => p.dex === 'Nomadex').map(p => ({
                x: new Date(p.timestamp),
                y: p.price
            }));
            chartOptions.data.datasets[0].data = humbleData;
            chartOptions.data.datasets[1].data = nomadexData;
        } else {
            console.error('Failed to fetch data');
        }
    }

    onMount(fetchData);
</script>

<main>
    {#if humbleLPInfo || nomadexLPInfo}
        <h1>Price of VOI</h1>
        <div class="chart-container">
            <Line {...chartOptions} />
        </div>
        {#if humbleLPInfo}
            <h2>Humble LP Information</h2>
            <div>
                <strong>wVOI Balance:</strong> {humbleLPInfo.wVoi / 1e6}
                <strong>VIA Balance:</strong> {humbleLPInfo.Via / 1e6}
                <h3><a href="https://shellyssandbox.xyz/#/swap?poolId=24590736&mode=swap">Swap on Humble</a></h3>
            </div>
        {/if}
        {#if nomadexLPInfo}
            <h2>Nomadex LP Information</h2>
            <div>
                <strong>VOI Balance:</strong> {nomadexLPInfo.Voi / 1e6}
                <strong>VIA Balance:</strong> {nomadexLPInfo.Via / 1e6}
                <h3><a href="https://voi.nomadex.app/swap/VOI-VIA">Swap on Nomadex</a></h3>
            </div>
        {/if}
    {/if}

    <h1>Testnet Voi Value in USD</h1>

    <iframe
    title="Vestige Widget"
    src="https://vestige.fi/widget/1392374998/chart?currency=USD&tools=false&adjust=true&interval=60"
    width="500" height="300"
    />

    <h1><a href="https://voitest.k8s.aramid.finance/">Aramid Bridge</a> Voi testnet & Algorand mainnet</h1>



    {#if !humbleLPInfo && !nomadexLPInfo}
        <p>Loading LP information...</p>
    {/if}

    <h2>Price History</h2>
    {#if priceHistory.length > 0}
        <ul>
            {#each priceHistory as entry}
                <li>{entry.dex}: {new Date(entry.timestamp).toLocaleString()} - {Number(entry.price).toFixed(4)}</li>
            {/each}
        </ul>
    {:else}
        <p>Loading price history...</p>
    {/if}
</main>

<style>
    .chart-container {
        width: 500px;
        height: 300px;
    }
</style>