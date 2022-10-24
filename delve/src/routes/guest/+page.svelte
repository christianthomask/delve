<script>
    import {createClient} from "@supabase/supabase-js";
    import {onMount} from 'svelte';

    //Init Supabase
    const supaUrl = 'https://dzfjtjzuhrpctluwbzlm.supabase.co'
    const supaAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImR6Zmp0anp1aHJwY3RsdXdiemxtIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjY1MDIyNzMsImV4cCI6MTk4MjA3ODI3M30.fyzvcc4gEroDuPNEo9AHqOPQNmt9oBgzQ_N6C3QP9x8'
    const supabase = createClient(supaUrl, supaAnonKey)

    let state = "unjoined"
    let roomCode;
    let name;
    let playerRace;
    let playerClass;

    function game(inst) {

        //Object Instructions

            if(inst.action === 'add_player') {
                state = 'joined'
            }

    }

    async function join() {
        name = document.getElementById('name').value
        roomCode = document.getElementById('code').value
        const raceChoices = document.getElementsByName('race')
        // console.log(raceChoices)
        await raceChoices.forEach((elem) => {
            if(elem.checked) {
                // console.log(elem.value)
                playerRace = elem.value;
            }
        })
        const classChoices = document.getElementsByName('class')
        classChoices.forEach((elem) => {
            if(elem.checked) {
                playerClass = elem.value
            }
        })
        const gameSession = supabase.channel(roomCode, {
            config: {
                broadcast: {ack: true, self: true}
            }
        })

        gameSession.on('broadcast', {event: 'inst'}, (payload) => {
            // console.log(payload);
            game(payload.payload);
        }).subscribe((status) => {
            if(status === 'SUBSCRIBED') {
                gameSession.send({
                    type: 'broadcast',
                    event: 'inst',
                    payload: {action: 'add_player', player_name: name, player_race: playerRace, player_class: playerClass}
                })
                console.log(gameSession);
            }
        })
    }

</script>
<div class="w-full h-screen flex flex-col items-center justify-center bg-gray-800 text-white">
    {#if state === 'unjoined'}
        <div class="w-full h-fit max-w-3xl flex flex-col p-4 bg-blue-700 rounded-lg gap-y-4">
            <h1 class="text-2xl font-extrabold">Name your Adventurer!</h1>
            <input class="rounded-md text-black" id="name" type="text">
            <h2 class="text-2xl font-extrabold">From which mighty race do they hail?</h2>
            <fieldset>
                <label for="human">Human: +1 Health; +1 Energy</label>
                <input type="radio" name="race" value="Human" id="human" checked required><br>
                <label for="dwarf">Dwarf: +2 Health</label>
                <input type="radio" name="race" value="Dwarf" id="dwarf"><br>
                <label for="elf">Elf +2 Energy</label>
                <input type="radio" name="race" value="Elf" id="elf"><br>
            </fieldset>
            <h2 class="text-2xl font-extrabold">And what daring Profession?</h2>
            <fieldset>
                <label for="human">Warrior: spend 1 energy to escape death once!</label>
                <input type="radio" name="class" value="Warrior" id="warrior" checked required><br>
                <label for="dwarf">Rogue: spend 1 energy to see the choice of another party member once per turn!</label>
                <input type="radio" name="class" value="Rogue" id="rogue"><br>
                <label for="elf">Wizard: select a single spell and, when able, spend 1 energy to cast it each turn!</label>
                <input type="radio" name="class" value="Wizard" id="wizard"><br>
            </fieldset>
            <h2 class="text-2xl font-extrabold">And the Party you will join?</h2>
            <input class="rounded-md text-black" id="code" type="text">
            <button class="py-8 px-4 bg-blue-500 rounded-lg text-yellow-300 text-2xl font-extrabold" on:click={join}>Join!</button>
        </div>
    {:else if state === 'joined'}
        <h1 class="text-2xl font-extrabold">{name} the {playerRace} {playerClass} joined party {roomCode}!</h1>
    {/if}
</div>
