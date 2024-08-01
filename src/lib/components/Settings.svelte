<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Switch } from "$lib/components/ui/switch";
    import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogDescription } from "$lib/components/ui/dialog";
    import { Label } from "$lib/components/ui/label";
    import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "$lib/components/ui/select";
    import { createEventDispatcher } from 'svelte';
    import { browser } from '$app/environment';

    const dispatch = createEventDispatcher();

    function getFromLocalStorage(key: string, defaultValue: any) {
        if (browser) {
            const storedValue = localStorage.getItem(key);
            return storedValue ? JSON.parse(storedValue) : defaultValue;
        }
        return defaultValue;
    }

    export let workTime = getFromLocalStorage('workTime', 25);
    export let shortBreakTime = getFromLocalStorage('shortBreakTime', 5);
    export let longBreakTime = getFromLocalStorage('longBreakTime', 15);
    export let sessionsBeforeLongBreak = getFromLocalStorage('sessionsBeforeLongBreak', 4);
    export let darkMode = getFromLocalStorage('darkMode', false);
    export let notifications = getFromLocalStorage('notifications', true);
    export let autoStartBreaks = getFromLocalStorage('autoStartBreaks', true);
    export let autoStartPomodoros = getFromLocalStorage('autoStartPomodoros', true);
    export let alarmSound = getFromLocalStorage('alarmSound', "bell");

    let open = false;

    function toggleSettings() {
        open = !open;
    }

    function saveSettings() {
        dispatch('saveSettings', {
            workTime,
            shortBreakTime,
            longBreakTime,
            sessionsBeforeLongBreak,
            darkMode,
            notifications,
            autoStartBreaks,
            autoStartPomodoros,
            alarmSound
        });
        toggleSettings();
    }
</script>

<Button
    class="fixed bottom-4 right-4 rounded-full w-12 h-12 flex items-center justify-center bg-blue-500 hover:bg-blue-600 text-white"
    on:click={toggleSettings}
>
    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
    </svg>
</Button>

<Dialog bind:open>
    <DialogContent class="sm:max-w-[425px]">
        <DialogHeader>
            <DialogTitle>Settings</DialogTitle>
            <DialogDescription>
                Customize your Pomodoro experience
            </DialogDescription>
        </DialogHeader>
        <div class="grid gap-4 py-4">
            <div class="grid grid-cols-4 items-center gap-4">
                <Label class="text-right">Work Time (minutes)</Label>
                <Input type="number" bind:value={workTime} min="1" max="60" class="col-span-3" />
            </div>
            <div class="grid grid-cols-4 items-center gap-4">
                <Label class="text-right">Short Break (minutes)</Label>
                <Input type="number" bind:value={shortBreakTime} min="1" max="30" class="col-span-3" />
            </div>
            <div class="grid grid-cols-4 items-center gap-4">
                <Label class="text-right">Long Break (minutes)</Label>
                <Input type="number" bind:value={longBreakTime} min="1" max="60" class="col-span-3" />
            </div>
            <div class="grid grid-cols-4 items-center gap-4">
                <Label class="text-right">Sessions before Long Break</Label>
                <Input type="number" bind:value={sessionsBeforeLongBreak} min="1" max="10" class="col-span-3" />
            </div>
            <div class="flex items-center space-x-2">
                <Switch bind:checked={darkMode} id="dark-mode" />
                <Label for="dark-mode">Dark Mode</Label>
            </div>
            <div class="flex items-center space-x-2">
                <Switch bind:checked={notifications} id="notifications" />
                <Label for="notifications">Enable Notifications</Label>
            </div>
            <div class="flex items-center space-x-2">
                <Switch bind:checked={autoStartBreaks} id="auto-start-breaks" />
                <Label for="auto-start-breaks">Auto-start Breaks</Label>
            </div>
            <div class="flex items-center space-x-2">
                <Switch bind:checked={autoStartPomodoros} id="auto-start-pomodoros" />
                <Label for="auto-start-pomodoros">Auto-start Pomodoros</Label>
            </div>
            <div class="grid grid-cols-4 items-center gap-4">
                <Label class="text-right">Alarm Sound</Label>
                <Select bind:value={alarmSound} class="col-span-3">
                    <SelectTrigger>
                        <SelectValue placeholder="Select a sound" />
                    </SelectTrigger>
                    <SelectContent>
                        <SelectItem value="bell">Bell</SelectItem>
                        <SelectItem value="digital">Digital</SelectItem>
                        <SelectItem value="kitchen">Kitchen Timer</SelectItem>
                        <SelectItem value="wood">Wood Block</SelectItem>
                    </SelectContent>
                </Select>
            </div>
        </div>
        <div class="flex justify-end">
            <Button on:click={saveSettings}>Save Changes</Button>
        </div>
    </DialogContent>
</Dialog>