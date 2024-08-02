<script lang="ts">
  import { fade, fly } from 'svelte/transition';
  import { Alert, AlertDescription } from "$lib/components/ui/alert";

  export let message: string;
</script>

<div class="fixed left-4 bottom-4 z-50 max-w-sm" 
     in:fly="{{ x: -100, duration: 500 }}" 
     out:fade="{{ duration: 300 }}">
  <Alert variant="default" class="bg-white dark:bg-gray-800 border-l-4 border-blue-500 shadow-lg">
    <div class="flex items-center">
      <svg class="w-6 h-6 text-blue-500 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
      </svg>
      <AlertDescription class="text-sm font-medium text-gray-900 dark:text-gray-100">
        {message}
      </AlertDescription>
    </div>
  </Alert>
</div>

<style>
  :global(.notification-enter) {
    animation: notification-enter 0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  }

  @keyframes notification-enter {
    0% {
      transform: translateX(-100px);
      opacity: 0;
    }
    100% {
      transform: translateX(0);
      opacity: 1;
    }
  }
</style>