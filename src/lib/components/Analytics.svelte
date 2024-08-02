<!-- src/lib/components/Analytics.svelte -->
<script lang="ts">
    import { onMount, onDestroy } from 'svelte';
    import { Chart, registerables } from 'chart.js';
    import { browser } from '$app/environment';
    import { fade } from 'svelte/transition';
    import { Clock, Calendar, CheckSquare, BarChart2 } from 'lucide-svelte';

    Chart.register(...registerables);

    let productivityChart: Chart;
    let progressChart: Chart;
    export let totalFocusTime: number;
    export let totalTasks: number;
    export let completedTasks: number;
    export let longestStreak: number;

    function updateData() {
        if (browser) {
            totalFocusTime = parseInt(localStorage.getItem('totalFocusTime') || '0');
            totalTasks = parseInt(localStorage.getItem('totalTasks') || '0');
            completedTasks = parseInt(localStorage.getItem('completedTasks') || '0');
            longestStreak = parseInt(localStorage.getItem('longestStreak') || '0');

            updateCharts();
        }
    }
    function secondsToHours(seconds: number): number {
        return Math.round((seconds / 3600) * 100) / 100;
    }

    function updateCharts() {
        if (productivityChart) {
            const lastWeekData = JSON.parse(localStorage.getItem('weeklyProductivity') || '[]');
            productivityChart.data.datasets[0].data = lastWeekData;
            productivityChart.update();
        }

        if (progressChart) {
            progressChart.data.datasets[0].data = [completedTasks, totalTasks - completedTasks];
            progressChart.update();
        }
    }

    onMount(() => {
        updateData();

        const ctx1 = document.getElementById('productivityChart') as HTMLCanvasElement;
        productivityChart = new Chart(ctx1, {
            type: 'line',
            data: {
                labels: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'],
                datasets: [{
                    label: 'Focus hours',
                    data: [],
                    fill: false,
                    borderColor: 'rgb(75, 192, 192)',
                    tension: 0.1
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    title: {
                        display: true,
                        text: 'Weekly productivity'
                    }
                }
            }
        });

        const ctx2 = document.getElementById('progressChart') as HTMLCanvasElement;
        progressChart = new Chart(ctx2, {
            type: 'doughnut',
            data: {
                labels: ['Completed tasks', 'Remaining tasks'],
                datasets: [{
                    data: [completedTasks, totalTasks - completedTasks],
                    backgroundColor: ['rgba(75, 192, 192, 0.8)', 'rgba(255, 99, 132, 0.8)'],
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    title: {
                        display: true,
                        text: 'Task progress'
                    }
                }
            }
        });

        const interval = setInterval(updateData, 60000);

        return () => clearInterval(interval);
    });
</script>

<div class="bg-white dark:bg-gray-800 rounded-lg shadow-lg p-6" transition:fade>
    <h2 class="text-2xl font-bold mb-6 dark:text-white flex items-center">
        <BarChart2 class="mr-2" /> Analytics and statistics
    </h2>
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
        <div class="bg-blue-100 dark:bg-blue-900 p-4 rounded-lg shadow">
            <h3 class="text-lg font-semibold mb-2 dark:text-white flex items-center">
                <Clock class="mr-2" /> Total focus time
            </h3>
            <p class="text-3xl font-bold text-blue-600 dark:text-blue-300">{secondsToHours(totalFocusTime)} Hour</p>
        </div>
        <div class="bg-green-100 dark:bg-green-900 p-4 rounded-lg shadow">
            <h3 class="text-lg font-semibold mb-2 dark:text-white flex items-center">
                <CheckSquare class="mr-2" /> Completed tasks
            </h3>
            <p class="text-3xl font-bold text-green-600 dark:text-green-300">{completedTasks} / {totalTasks}</p>
        </div>
        <div class="bg-yellow-100 dark:bg-yellow-900 p-4 rounded-lg shadow">
            <h3 class="text-lg font-semibold mb-2 dark:text-white flex items-center">
                <Calendar class="mr-2" /> Longest focus streak
            </h3>
            <p class="text-3xl font-bold text-yellow-600 dark:text-yellow-300">{longestStreak} Day</p>
        </div>
        <div class="bg-purple-100 dark:bg-purple-900 p-4 rounded-lg shadow">
            <h3 class="text-lg font-semibold mb-2 dark:text-white flex items-center">
                <BarChart2 class="mr-2" /> Completion rate
            </h3>
            <p class="text-3xl font-bold text-purple-600 dark:text-purple-300">
                {totalTasks > 0 ? Math.round((completedTasks / totalTasks) * 100) : 0}%
            </p>
        </div>
    </div>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="bg-gray-50 dark:bg-gray-700 p-4 rounded-lg shadow">
            <canvas id="productivityChart"></canvas>
        </div>
        <div class="bg-gray-50 dark:bg-gray-700 p-4 rounded-lg shadow">
            <canvas id="progressChart"></canvas>
        </div>
    </div>
</div>