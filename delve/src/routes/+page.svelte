<script>
    import {goto} from '$app/navigation'

</script>

<div class="w-full h-screen px-8 flex flex-col items-center bg-gray-200">
    <h1 class="text-5xl mt-24">Welcome to Delve with Friends!!</h1>
    <h2 class="text-3xl mt-16">To start a game select 'Host'. Then you and your party can join by going to <span class="font-bold">delvewithfriends.app</span>, selecting 'Join', and inputting a name and the room code on the screen.</h2>
    <div class="flex flex-col items-center py-12 gap-y-8">
        <button class="w-full rounded-lg bg-blue-500 px-12 py-8 text-yellow-300 text-3xl font-extrabold cursor-pointer" on:click={() => goto('/host')}>Host</button>
        <button class="w-full rounded-lg bg-green-500 px-12 py-8 text-yellow-300 text-3xl font-extrabold cursor-pointer" on:click={() => goto('/guest')}>Join</button>
    </div>
</div>
