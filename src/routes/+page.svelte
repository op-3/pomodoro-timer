<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Progress } from "$lib/components/ui/progress";
    import { onMount, afterUpdate } from "svelte";
    import Notification from '../lib/components/Notification.svelte';
    import TodoList from '../lib/components/TodoList.svelte';
    import Settings from '../lib/components/Settings.svelte';
    import Analytics from '../lib/components/Analytics.svelte';
    import { createEventDispatcher } from 'svelte';
    import { browser } from '$app/environment';
    import { Play, Pause, RotateCcw } from 'lucide-svelte';
    import { Moon, Sun, Settings as SettingsIcon } from 'lucide-svelte';
    
    const dispatch = createEventDispatcher();

    // Local Storage functions
    function saveToLocalStorage(key: string, value: any) {
        if (browser) {
            localStorage.setItem(key, JSON.stringify(value));
        }
    }

    function getFromLocalStorage(key: string, defaultValue: any) {
        if (browser) {
            const storedValue = localStorage.getItem(key);
            return storedValue ? JSON.parse(storedValue) : defaultValue;
        }
        return defaultValue;
    }

    // Timer settings
    let workTime = 25;
    let shortBreakTime = 5;
    let longBreakTime = 15;
    let sessionsBeforeLongBreak = 4;
    let darkMode = true;
    let notifications = true;
    let autoStartBreaks = true;
    let autoStartPomodoros = true;
    let alarmSound = "bell";
    let workVideo = 'https://www.youtube.com/watch?v=jfKfPfyJRdk';
    let breakVideo = 'https://www.youtube.com/watch?v=wxnsKBFsPTU';
    let showVideoPlayer = true;
    let playerVolume = 50;
    let isMuted = false;
    let showSettings = false;
    let playerWidth = '100%';
    let playerHeight = '100%';

    // Timer state
    let timeLeft: number;
    let isRunning = false;
    let progress = 100;
    let timer: number;
    let sessionCount = 0;
    let currentMode: 'work' | 'shortBreak' | 'longBreak' = 'work';
    let showNotification = false;
    let notificationMessage = '';
    let isFirstStart = true;
    let initialTime: number;

    // YouTube player
    let player: any;
    let isMobile: boolean;

    // Analytics data
    let totalFocusTime = 0;
    let totalSessions = 0;
    let totalTasks = 0;
    let completedTasks = 0;
    let currentStreak = 0;
    let longestStreak = 0;
    let lastCompletedDate: Date | null = null;
    let focusTimer: number;
    let currentSessionTime = 0;

    $: WORK_TIME = workTime * 60;
    $: SHORT_BREAK_TIME = shortBreakTime * 60;
    $: LONG_BREAK_TIME = longBreakTime * 60;
    $: SESSIONS_BEFORE_LONG_BREAK = sessionsBeforeLongBreak;

    function toggleSettings() {
        showSettings = !showSettings;
    }

    function saveTimerState() {
        if (browser) {
            saveToLocalStorage('timeLeft', timeLeft);
            saveToLocalStorage('isRunning', isRunning);
            saveToLocalStorage('currentMode', currentMode);
            saveToLocalStorage('sessionCount', sessionCount);
            saveToLocalStorage('initialTime', initialTime);
            saveToLocalStorage('isFirstStart', isFirstStart);
        }
    }

    function checkMobile() {
        isMobile = window.innerWidth < 768;
    }

    function toggleMute() {
        if (player && typeof player.mute === 'function' && typeof player.unMute === 'function') {
            if (isMuted) {
                player.unMute();
                player.setVolume(playerVolume);
            } else {
                player.mute();
            }
            isMuted = !isMuted;
        }
    }

    function toggleDarkMode() {
        darkMode = !darkMode;
        saveToLocalStorage('darkMode', darkMode);
        applyDarkMode(darkMode);
    }

    function applyDarkMode(isDark: boolean) {
        if (isDark) {
            document.documentElement.classList.add('dark');
        } else {
            document.documentElement.classList.remove('dark');
        }
    }

    function setVolume(volume: number) {
        if (player && typeof player.setVolume === 'function') {
            player.setVolume(volume);
            playerVolume = volume;
            if (isMuted && volume > 0) {
                isMuted = false;
            }
        }
    }

    function startTimer() {
        if (!isRunning) {
            isRunning = true;
            isFirstStart = false;
            currentSessionTime = 0;
            focusTimer = setInterval(() => {
                currentSessionTime++;
                timeLeft--;
                updateProgress();
                if (timeLeft <= 0) {
                    handleSessionEnd();
                }
            }, 1000);
            saveTimerState();
            if (player && typeof player.playVideo === 'function') {
                player.playVideo();
            }
        }
    }

    function stopTimer() {
        clearInterval(focusTimer);
        isRunning = false;
        const actualSessionTime = initialTime - timeLeft;
        addFocusTime(actualSessionTime);
        saveTimerState();
        if (player && typeof player.pauseVideo === 'function') {
            player.pauseVideo();
        }
    }
    

    function resetTimer() {
        clearInterval(focusTimer);
        isRunning = false;
        isFirstStart = true;
        sessionCount = 0;
        setMode(currentMode);
        saveTimerState();
    }

    function handleSessionEnd() {
        clearInterval(focusTimer);
        const actualSessionTime = initialTime - timeLeft;
        addFocusTime(actualSessionTime);
        if (currentMode === 'work') {
            sessionCount++;
            if (sessionCount % SESSIONS_BEFORE_LONG_BREAK === 0) {
                setMode('longBreak');
            } else {
                setMode('shortBreak');
         }
        } else {
            setMode('work');
        }
        if ((currentMode !== 'work' && autoStartBreaks) || (currentMode === 'work' && autoStartPomodoros)) {
            startTimer();
        }
        playAlarmSound();
    }

    function setMode(mode: 'work' | 'shortBreak' | 'longBreak') {
        currentMode = mode;
        switch (mode) {
            case 'work':
                timeLeft = WORK_TIME;
                notificationMessage = 'The work session has begun';
                break;
            case 'shortBreak':
                timeLeft = SHORT_BREAK_TIME;
                notificationMessage = 'The short break has begun';
                break;
            case 'longBreak':
                timeLeft = LONG_BREAK_TIME;
                notificationMessage = 'The long break has begun';
                break;
        }
        initialTime = timeLeft;
        isRunning = false;
        clearInterval(focusTimer);
        showNotification = true;
        updateProgress();
        if (notifications) {
            dispatch('showNotification', { message: notificationMessage });
        }
        setTimeout(() => {
            showNotification = false;
        }, 3000);
        saveTimerState();
        if (player && typeof player.loadVideoById === 'function') {
            player.loadVideoById(extractVideoId(currentMode === 'work' ? workVideo : breakVideo));
            player.pauseVideo();
        }
    }

    function updateProgress() {
        const totalTime = currentMode === 'work' ? WORK_TIME : 
                        currentMode === 'shortBreak' ? SHORT_BREAK_TIME : LONG_BREAK_TIME;
        progress = (timeLeft / totalTime) * 100;
    }

    function formatTime(seconds: number) {
        if (isNaN(seconds) || seconds < 0) {
            return "00:00";
        }
        const minutes = Math.floor(seconds / 60);
        const remainingSeconds = seconds % 60;
        return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`;
    }

    function playAlarmSound() {
        // Implement alarm sound logic here
        console.log(`Playing ${alarmSound} sound`);
    }

    function extractVideoId(url: string): string {
        const regExp = /^.*(youtu.be\/|v\/|u\/\w\/|embed\/|watch\?v=|\&v=)([^#\&\?]*).*/;
        const match = url.match(regExp);
        return (match && match[2].length === 11) ? match[2] : "";
    }

    function handleSaveSettings(event: CustomEvent) {
        const newSettings = event.detail;
        workTime = newSettings.workTime;
        shortBreakTime = newSettings.shortBreakTime;
        longBreakTime = newSettings.longBreakTime;
        sessionsBeforeLongBreak = newSettings.sessionsBeforeLongBreak;
        notifications = newSettings.notifications;
        autoStartBreaks = newSettings.autoStartBreaks;
        autoStartPomodoros = newSettings.autoStartPomodoros;
        alarmSound = newSettings.alarmSound;
        workVideo = newSettings.workVideo;
        breakVideo = newSettings.breakVideo;
        showVideoPlayer = newSettings.showVideoPlayer;

        saveToLocalStorage('workTime', workTime);
        saveToLocalStorage('shortBreakTime', shortBreakTime);
        saveToLocalStorage('longBreakTime', longBreakTime);
        saveToLocalStorage('sessionsBeforeLongBreak', sessionsBeforeLongBreak);
        saveToLocalStorage('notifications', notifications);
        saveToLocalStorage('autoStartBreaks', autoStartBreaks);
        saveToLocalStorage('autoStartPomodoros', autoStartPomodoros);
        saveToLocalStorage('alarmSound', alarmSound);
        saveToLocalStorage('workVideo', workVideo);
        saveToLocalStorage('breakVideo', breakVideo);
        saveToLocalStorage('showVideoPlayer', showVideoPlayer);

        // Update timer values
        WORK_TIME = workTime * 60;
        SHORT_BREAK_TIME = shortBreakTime * 60;
        LONG_BREAK_TIME = longBreakTime * 60;
        SESSIONS_BEFORE_LONG_BREAK = sessionsBeforeLongBreak;

        // Reset current timer if not running
        if (!isRunning) {
            setMode(currentMode);
        } else {
            updateProgress();
        }

        showSettings = false;
    }

    // Analytics functions
    function updateAnalytics() {
        if (browser) {
            totalFocusTime = parseInt(localStorage.getItem('totalFocusTime') || '0');
            totalSessions = parseInt(localStorage.getItem('totalSessions') || '0');
            totalTasks = parseInt(localStorage.getItem('totalTasks') || '0');
            completedTasks = parseInt(localStorage.getItem('completedTasks') || '0');
            currentStreak = parseInt(localStorage.getItem('currentStreak') || '0');
            longestStreak = parseInt(localStorage.getItem('longestStreak') || '0');
            lastCompletedDate = localStorage.getItem('lastCompletedDate') ? new Date(localStorage.getItem('lastCompletedDate')) : null;
        }
    }

    function saveAnalytics() {
        if (browser) {
            localStorage.setItem('totalFocusTime', totalFocusTime.toString());
            localStorage.setItem('totalSessions', totalSessions.toString());
            localStorage.setItem('totalTasks', totalTasks.toString());
            localStorage.setItem('completedTasks', completedTasks.toString());
            localStorage.setItem('currentStreak', currentStreak.toString());
            localStorage.setItem('longestStreak', longestStreak.toString());
            if (lastCompletedDate) {
                localStorage.setItem('lastCompletedDate', lastCompletedDate.toISOString());
            }
        }
    }

    function addFocusTime(seconds: number) {
        totalFocusTime += seconds;
        totalSessions++;
        updateWeeklyProductivity(seconds);
        updateStreak();
        saveAnalytics();
    }

    function updateWeeklyProductivity(seconds: number) {
        const weeklyProductivity = JSON.parse(localStorage.getItem('weeklyProductivity') || '{}');
        const today = new Date().toISOString().split('T')[0];
        weeklyProductivity[today] = (weeklyProductivity[today] || 0) + seconds;
        
        const lastWeek = Object.entries(weeklyProductivity)
            .sort((a, b) => new Date(b[0]).getTime() - new Date(a[0]).getTime())
            .slice(0, 7);
        
        localStorage.setItem('weeklyProductivity', JSON.stringify(Object.fromEntries(lastWeek)));
    }

    function updateStreak() {
        const today = new Date();
        if (lastCompletedDate) {
            const dayDiff = (today.getTime() - lastCompletedDate.getTime()) / (1000 * 3600 * 24);
            if (dayDiff <= 1) {
                currentStreak++;
            } else {
                currentStreak = 1;
            }
        } else {
            currentStreak = 1;
        }
        lastCompletedDate = today;
        
        if (currentStreak > longestStreak) {
            longestStreak = currentStreak;
        }
    }

    function addTask() {
        totalTasks++;
        saveAnalytics();
    }

    function completeTask() {
        completedTasks++;
        saveAnalytics();
    }

    function removeTask(wasCompleted: boolean) {
        totalTasks--;
        if (wasCompleted) {
            completedTasks--;
        }
        saveAnalytics();
    }

    onMount(() => {
        if (browser) {
            workTime = getFromLocalStorage('workTime', 25);
            shortBreakTime = getFromLocalStorage('shortBreakTime', 5);
            longBreakTime = getFromLocalStorage('longBreakTime', 15);
            sessionsBeforeLongBreak = getFromLocalStorage('sessionsBeforeLongBreak', 4);
            darkMode = getFromLocalStorage('darkMode', false);
            notifications = getFromLocalStorage('notifications', true);
            autoStartBreaks = getFromLocalStorage('autoStartBreaks', true);
            autoStartPomodoros = getFromLocalStorage('autoStartPomodoros', true);
            alarmSound = getFromLocalStorage('alarmSound', "bell");
            workVideo = getFromLocalStorage('workVideo', 'https://www.youtube.com/watch?v=jfKfPfyJRdk');
            breakVideo = getFromLocalStorage('breakVideo', 'https://www.youtube.com/watch?v=wxnsKBFsPTU');
            showVideoPlayer = getFromLocalStorage('showVideoPlayer', true);
            
            currentMode = getFromLocalStorage('currentMode', 'work');
            sessionCount = getFromLocalStorage('sessionCount', 0);
            isFirstStart = getFromLocalStorage('isFirstStart', true);
            initialTime = getFromLocalStorage('initialTime', WORK_TIME);
            timeLeft = getFromLocalStorage('timeLeft', initialTime);
            isRunning = getFromLocalStorage('isRunning', false);
            
            updateAnalytics();

            // Ensure timeLeft is not greater than initialTime and not negative
            if (timeLeft > initialTime || timeLeft < 0) {
                timeLeft = initialTime;
            }

            updateProgress();

            // Load YouTube IFrame API
            const tag = document.createElement('script');
            tag.src = "https://www.youtube.com/iframe_api";
            const firstScriptTag = document.getElementsByTagName('script')[0];
            firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);

            window.onYouTubeIframeAPIReady = () => {
                player = new YT.Player('player', {
                    width: playerWidth,
                    height: playerHeight,
                    videoId: extractVideoId(currentMode === 'work' ? workVideo : breakVideo),
                    playerVars: {
                        'playsinline': 1,
                        'controls': 0,
                        'disablekb': 1,
                        'autoplay': 0,
                    },
                    events: {
                        'onReady': onPlayerReady,
                        'onStateChange': onPlayerStateChange
                    }
                });
            };

            function onPlayerStateChange(event) {
                if (event.data === YT.PlayerState.ENDED && isRunning) {
                    player.playVideo();
                }
            }

            function onPlayerReady(event) {
                setVolume(playerVolume);
                if (isRunning) {
                    event.target.playVideo();
                } else {
                    event.target.pauseVideo();
                }
            }

            if (isRunning) {
                startTimer();
            }
        }

        applyDarkMode(darkMode);
        checkMobile();
        window.addEventListener('resize', checkMobile);

        return () => {
            clearInterval(focusTimer);
            saveTimerState();
            window.removeEventListener('resize', checkMobile);
        };
    });

    afterUpdate(() => {
        applyDarkMode(darkMode);
    });

    // TodoList related functions
    function handleAddTodo() {
        addTask();
    }

    function handleCompleteTodo() {
        completeTask();
    }

    function handleUncompleteTodo() {
        completedTasks--;
        saveAnalytics();
    }

    function handleRemoveTask(event: CustomEvent<{ wasCompleted: boolean }>) {
        removeTask(event.detail.wasCompleted);
    }

    // Expose functions to be used in the component's markup
    const timerFunctions = {
        startTimer,
        stopTimer,
        resetTimer,
        setMode,
        toggleDarkMode,
        toggleSettings,
        toggleMute,
        setVolume,
    };

    const analyticsFunctions = {
        addFocusTime,
        addTask,
        completeTask,
        removeTask,
    };
</script>

<div class="flex h-screen bg-gray-100 dark:bg-gray-900 transition-colors duration-300" class:dark={darkMode}>
    <div class="w-1/3 flex flex-col p-4">
        <!-- Pomodoro Timer Box -->
        <div class="bg-white dark:bg-gray-800 shadow-lg rounded-lg p-6 mb-4 flex-1 transition-colors duration-300 flex flex-col justify-between">
            <h1 class="text-2xl md:text-3xl font-bold mb-4 dark:text-white text-center">Pomodoro Timer</h1>
            
            <div class="flex-grow flex items-center justify-center">
                <div class="w-64 h-64 relative">
                    <svg class="w-full h-full" viewBox="0 0 100 100">
                        <circle class="text-gray-200 stroke-current" stroke-width="10" cx="50" cy="50" r="40" fill="none"/>
                        <circle class="text-blue-500 stroke-current" stroke-width="10" cx="50" cy="50" r="40" fill="none"
                            stroke-dasharray="251.2" stroke-dashoffset={251.2 - (progress / 100) * 251.2}
                        />
                    </svg>
                    <div class="absolute top-0 left-0 w-full h-full flex items-center justify-center">
                        <span class="text-4xl font-bold">{formatTime(timeLeft)}</span>
                    </div>
                </div>
            </div>
            <div class="flex justify-center space-x-4 mb-6">
                {#if !isRunning}
                    <button on:click={startTimer} class="p-3 bg-green-500 text-white rounded-full hover:bg-green-600 transition-colors duration-300">
                        <Play size={24} />
                    </button>
                {:else}
                    <button on:click={stopTimer} class="p-3 bg-yellow-500 text-white rounded-full hover:bg-yellow-600 transition-colors duration-300">
                        <Pause size={24} />
                    </button>
                {/if}
                <button on:click={resetTimer} class="p-3 bg-red-500 text-white rounded-full hover:bg-red-600 transition-colors duration-300">
                    <RotateCcw size={24} />
                </button>
            </div>
            <div class="grid grid-cols-3 gap-2">
                <Button class="w-full" on:click={() => { setMode('work'); if (!isFirstStart) startTimer(); }} disabled={isRunning}>Work</Button>
                <Button class="w-full" on:click={() => { setMode('shortBreak'); if (!isFirstStart) startTimer(); }} disabled={isRunning}>Short Break</Button>
                <Button class="w-full" on:click={() => { setMode('longBreak'); if (!isFirstStart) startTimer(); }} disabled={isRunning}>Long Break</Button>
            </div>
        </div>

        <!-- Video Player Box -->
        {#if showVideoPlayer}
        <div class="bg-white dark:bg-gray-800 shadow-lg rounded-lg p-4 transition-colors duration-300">
            <div class="overflow-hidden rounded-lg" style="width: {playerWidth}px; height: {playerHeight}px;">
                <div id="player"></div>
            </div>
            <div class="mt-2 flex items-center justify-between">
                <button on:click={toggleMute} class="p-2 bg-gray-200 dark:bg-gray-700 rounded transition-colors duration-300">
                    {isMuted ? '🔇' : '🔊'}
                </button>
                <input 
                    type="range" 
                    min="0" 
                    max="100" 
                    bind:value={playerVolume} 
                    on:input={() => setVolume(playerVolume)}
                    class="w-3/4"
                />
            </div>
        </div>
        {/if}
    </div>

    <!-- Todo List -->
    <div class="w-2/3 p-4">
        <div class="bg-white dark:bg-gray-800 shadow-lg rounded-lg p-6 h-full overflow-auto transition-colors duration-300">
            <TodoList 
    on:addTask={handleAddTodo}
    on:completeTask={handleCompleteTodo}
    on:uncompleteTask={handleUncompleteTodo}
    on:removeTask={handleRemoveTask}
/>
        </div>
    </div>
</div>
 
 <div class="w-full p-4">
    <Analytics 
    {totalFocusTime}
    {totalTasks}
    {completedTasks}
    {longestStreak}
/>
</div>

<!-- Floating buttons for Dark Mode and Settings -->
<div class="fixed bottom-4 right-4 flex flex-col space-y-2">
    <button 
        on:click={toggleDarkMode}
        class="p-3 rounded-full bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300 hover:bg-gray-300 dark:hover:bg-gray-600 shadow-lg"
        aria-label={darkMode ? "Switch to Light Mode" : "Switch to Dark Mode"}
    >
        {#if darkMode}
            <Sun size={24} />
        {:else}
            <Moon size={24} />
        {/if}
    </button>
    <button 
        on:click={toggleSettings}
        class="p-3 rounded-full bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-gray-200 transition-colors duration-300 hover:bg-gray-300 dark:hover:bg-gray-600 shadow-lg"
        aria-label="Open Settings"
    >
        <SettingsIcon size={24} />
    </button>
</div>

{#if showNotification && notifications}
    <Notification message={notificationMessage} />
{/if}

{#if showSettings}
    <Settings 
        bind:workTime
        bind:shortBreakTime
        bind:longBreakTime
        bind:sessionsBeforeLongBreak
        bind:notifications
        bind:autoStartBreaks
        bind:autoStartPomodoros
        bind:alarmSound
        bind:workVideo
        bind:breakVideo
        bind:showVideoPlayer
        on:saveSettings={handleSaveSettings}
        on:close={() => showSettings = false}
    />
{/if}
<style>
    :root {
  --bg-light: #f3f4f6;
  --bg-dark: #111827;
  --text-light: #1f2937;
  --text-dark: #f3f4f6;
  --primary-light: #3b82f6;
  --primary-dark: #60a5fa;
  --secondary-light: #e5e7eb;
  --secondary-dark: #374151;
}

body {
  transition: background-color 0.3s ease, color 0.3s ease;
}

.dark body {
  background-color: var(--bg-dark);
  color: var(--text-dark);
}

.dark .bg-white {
  background-color: #1f2937;
}

.dark .shadow-lg {
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.3), 0 4px 6px -2px rgba(0, 0, 0, 0.1);
}

.dark .border-gray-200 {
  border-color: #374151;
}

.dark button:not(.bg-primary) {
  background-color: #374151;
  color: #f3f4f6;
}

.dark button:hover:not(.bg-primary) {
  background-color: #4b5563;
}

.dark input[type="range"] {
  background-color: #4b5563;
}

.dark input[type="range"]::-webkit-slider-thumb {
  background-color: var(--primary-dark);
}
</style>