<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Switch } from "$lib/components/ui/switch";
    import { Label } from "$lib/components/ui/label";
    import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "$lib/components/ui/select";
    import { createEventDispatcher } from 'svelte';
    import { X } from 'lucide-svelte';

    const dispatch = createEventDispatcher();

    export let workTime: number;
    export let shortBreakTime: number;
    export let longBreakTime: number;
    export let sessionsBeforeLongBreak: number;
    export let notifications: boolean;
    export let autoStartBreaks: boolean;
    export let autoStartPomodoros: boolean;
    export let alarmSound: string;
    export let workVideo: string;
    export let breakVideo: string;
    export let showVideoPlayer: boolean;

    function saveSettings() {
        dispatch('saveSettings', {
            workTime,
            shortBreakTime,
            longBreakTime,
            sessionsBeforeLongBreak,
            notifications,
            autoStartBreaks,
            autoStartPomodoros,
            alarmSound,
            workVideo,
            breakVideo,
            showVideoPlayer
        });
    }

    function closeSettings() {
        dispatch('close');
    }
</script>

<div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white dark:bg-gray-800 p-6 rounded-lg w-full max-w-md max-h-[90vh] overflow-y-auto">
        <div class="flex justify-between items-center mb-4">
            <h2 class="text-2xl font-bold dark:text-white">Settings</h2>
            <button on:click={closeSettings} class="text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-200">
                <X size={24} />
            </button>
        </div>
        
        <div class="space-y-4">
            <div>
                <Label for="work-time">Work Time (minutes)</Label>
                <Input type="number" id="work-time" bind:value={workTime} min="1" max="60" />
            </div>
            
            <div>
                <Label for="short-break">Short Break (minutes)</Label>
                <Input type="number" id="short-break" bind:value={shortBreakTime} min="1" max="30" />
            </div>
            
            <div>
                <Label for="long-break">Long Break (minutes)</Label>
                <Input type="number" id="long-break" bind:value={longBreakTime} min="1" max="60" />
            </div>
            
            <div>
                <Label for="sessions">Sessions before Long Break</Label>
                <Input type="number" id="sessions" bind:value={sessionsBeforeLongBreak} min="1" max="10" />
            </div>
            
            <div class="flex items-center justify-between">
                <Label for="notifications">Enable Notifications</Label>
                <Switch id="notifications" bind:checked={notifications} />
            </div>
            
            <div class="flex items-center justify-between">
                <Label for="auto-start-breaks">Auto-start Breaks</Label>
                <Switch id="auto-start-breaks" bind:checked={autoStartBreaks} />
            </div>
            
            <div class="flex items-center justify-between">
                <Label for="auto-start-pomodoros">Auto-start Pomodoros</Label>
                <Switch id="auto-start-pomodoros" bind:checked={autoStartPomodoros} />
            </div>
            
            <div>
                <Label for="alarm-sound">Alarm Sound</Label>
                <Select bind:value={alarmSound}>
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
            
            <div>
                <Label for="work-video">Work Video URL</Label>
                <Input type="url" id="work-video" bind:value={workVideo} placeholder="https://www.youtube.com/watch?v=..." />
            </div>
            
            <div>
                <Label for="break-video">Break Video URL</Label>
                <Input type="url" id="break-video" bind:value={breakVideo} placeholder="https://www.youtube.com/watch?v=..." />
            </div>
            
            <div class="flex items-center justify-between">
                <Label for="show-video-player">Show Video Player</Label>
                <Switch id="show-video-player" bind:checked={showVideoPlayer} />
            </div>
        </div>
        
        <div class="mt-6 flex justify-end">
            <Button on:click={saveSettings}>Save Changes</Button>
        </div>
    </div>
</div>
