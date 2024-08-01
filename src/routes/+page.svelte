<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Progress } from "$lib/components/ui/progress";
    import { onMount } from "svelte";
    import Notification from '../lib/components/Notification.svelte';
    import TodoList from '../lib/components/TodoList.svelte';
    import Settings from '../lib/components/Settings.svelte';
    import { createEventDispatcher } from 'svelte';
    import { browser } from '$app/environment';

    const dispatch = createEventDispatcher();

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

    let workTime = 25;
    let shortBreakTime = 5;
    let longBreakTime = 15;
    let sessionsBeforeLongBreak = 4;
    let darkMode = false;
    let notifications = true;
    let autoStartBreaks = true;
    let autoStartPomodoros = true;
    let alarmSound = "bell";

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

    $: WORK_TIME = workTime * 60;
    $: SHORT_BREAK_TIME = shortBreakTime * 60;
    $: LONG_BREAK_TIME = longBreakTime * 60;
    $: SESSIONS_BEFORE_LONG_BREAK = sessionsBeforeLongBreak;

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

    function startTimer() {
        if (!isRunning) {
            isRunning = true;
            isFirstStart = false;
            runTimer();
            saveTimerState();
        }
    }

    function runTimer() {
        timer = setInterval(() => {
            if (timeLeft > 0) {
                timeLeft--;
                updateProgress();
                saveTimerState();
            } else {
                clearInterval(timer);
                isRunning = false;
                handleSessionEnd();
            }
        }, 1000);
    }

    function stopTimer() {
        clearInterval(timer);
        isRunning = false;
        saveTimerState();
    }

    function resetTimer() {
        clearInterval(timer);
        isRunning = false;
        isFirstStart = true;
        sessionCount = 0;
        setMode(currentMode);
        saveTimerState();
    }

    function handleSessionEnd() {
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
                initialTime = WORK_TIME;
                notificationMessage = 'بدأت جلسة العمل';
                break;
            case 'shortBreak':
                initialTime = SHORT_BREAK_TIME;
                notificationMessage = 'بدأت الاستراحة القصيرة';
                break;
            case 'longBreak':
                initialTime = LONG_BREAK_TIME;
                notificationMessage = 'بدأت الاستراحة الطويلة';
                break;
        }
        timeLeft = initialTime;
        isRunning = false;
        clearInterval(timer);
        showNotification = true;
        updateProgress();
        if (notifications) {
            dispatch('showNotification', { message: notificationMessage });
        }
        setTimeout(() => {
            showNotification = false;
        }, 3000);
        saveTimerState();
    }

    function updateProgress() {
        if (initialTime > 0) {
            progress = (timeLeft / initialTime) * 100;
        } else {
            progress = 0;
        }
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
        // تنفيذ منطق تشغيل الصوت هنا
        console.log(`تشغيل صوت ${alarmSound}`);
    }

    function handleSaveSettings(event: CustomEvent) {
        const newSettings = event.detail;
        workTime = newSettings.workTime;
        shortBreakTime = newSettings.shortBreakTime;
        longBreakTime = newSettings.longBreakTime;
        sessionsBeforeLongBreak = newSettings.sessionsBeforeLongBreak;
        darkMode = newSettings.darkMode;
        notifications = newSettings.notifications;
        autoStartBreaks = newSettings.autoStartBreaks;
        autoStartPomodoros = newSettings.autoStartPomodoros;
        alarmSound = newSettings.alarmSound;

        if (browser) {
            saveToLocalStorage('workTime', workTime);
            saveToLocalStorage('shortBreakTime', shortBreakTime);
            saveToLocalStorage('longBreakTime', longBreakTime);
            saveToLocalStorage('sessionsBeforeLongBreak', sessionsBeforeLongBreak);
            saveToLocalStorage('darkMode', darkMode);
            saveToLocalStorage('notifications', notifications);
            saveToLocalStorage('autoStartBreaks', autoStartBreaks);
            saveToLocalStorage('autoStartPomodoros', autoStartPomodoros);
            saveToLocalStorage('alarmSound', alarmSound);
        }

        setMode(currentMode);
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
            
            currentMode = getFromLocalStorage('currentMode', 'work');
            sessionCount = getFromLocalStorage('sessionCount', 0);
            isFirstStart = getFromLocalStorage('isFirstStart', true);
            initialTime = getFromLocalStorage('initialTime', WORK_TIME);
            timeLeft = getFromLocalStorage('timeLeft', initialTime);
            isRunning = getFromLocalStorage('isRunning', false);

            // Ensure timeLeft is not greater than initialTime and not negative
            if (timeLeft > initialTime || timeLeft < 0) {
                timeLeft = initialTime;
            }

            updateProgress();
            if (isRunning) {
                runTimer();
            }
        }
        return () => {
            clearInterval(timer);
            saveTimerState();
        };
    });
</script>
<div class="flex h-screen bg-gray-100 dark:bg-gray-900" class:dark={darkMode}>
    <!-- Pomodoro Timer Box -->
    <div class="w-1/3 p-6 bg-white dark:bg-gray-800 shadow-lg m-4 rounded-lg overflow-auto">
        <h1 class="text-3xl font-bold mb-4 dark:text-white">Pomodoro Timer</h1>
        <div class="mb-4">
            <Progress value={progress} class="w-full" />
        </div>
        <div class="text-4xl font-bold mb-4 dark:text-white">
            {formatTime(timeLeft)}
        </div>
        <div class="mb-4 dark:text-gray-300">
            <p>Current Mode: {currentMode === 'work' ? 'Work' : currentMode === 'shortBreak' ? 'Short Break' : 'Long Break'}</p>
            <p>Sessions Completed: {sessionCount}</p>
        </div>
        <div class="space-y-2 mb-4">
            {#if !isRunning}
                <Button class="w-full" on:click={startTimer}>Start</Button>
            {:else}
                <Button class="w-full" on:click={stopTimer}>Pause</Button>
            {/if}
            <Button class="w-full" on:click={resetTimer}>Reset</Button>
        </div>
        <div class="space-y-2">
            <Button class="w-full" on:click={() => { setMode('work'); if (!isFirstStart) startTimer(); }} disabled={isRunning}>Work</Button>
            <Button class="w-full" on:click={() => { setMode('shortBreak'); if (!isFirstStart) startTimer(); }} disabled={isRunning}>Short Break</Button>
            <Button class="w-full" on:click={() => { setMode('longBreak'); if (!isFirstStart) startTimer(); }} disabled={isRunning}>Long Break</Button>
        </div>
    </div>

    <!-- Todo List -->
    <div class="w-2/3 p-6 bg-white dark:bg-gray-800 shadow-lg m-4 rounded-lg overflow-auto">
        <TodoList />
    </div>
</div>

{#if showNotification && notifications}
    <Notification message={notificationMessage} />
{/if}

<Settings 
    bind:workTime
    bind:shortBreakTime
    bind:longBreakTime
    bind:sessionsBeforeLongBreak
    bind:darkMode
    bind:notifications
    bind:autoStartBreaks
    bind:autoStartPomodoros
    bind:alarmSound
    on:saveSettings={handleSaveSettings}
/>