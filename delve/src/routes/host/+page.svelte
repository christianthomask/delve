<script>
    import {createClient} from "@supabase/supabase-js"
    import kaboom from "kaboom";
    import {onMount} from "svelte";

    const characters ='abcdefghijklmnopqrstuvwxyz';

    function generateString(length) {
        let result = '';
        const charactersLength = characters.length;
        for ( let i = 0; i < length; i++ ) {
            result += characters.charAt(Math.floor(Math.random() * charactersLength));
        }

        return result;
    }

    let roomCode;
    let currentBgm;
    let bgmPlayer;
    let soundEnabled = 'Off';

    let players = [];

    onMount(() => {
        const canvas = document.getElementById('canvas')

        //Init Supabase
        const supaUrl = 'https://dzfjtjzuhrpctluwbzlm.supabase.co'
        const supaAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImR6Zmp0anp1aHJwY3RsdXdiemxtIiwicm9sZSI6ImFub24iLCJpYXQiOjE2NjY1MDIyNzMsImV4cCI6MTk4MjA3ODI3M30.fyzvcc4gEroDuPNEo9AHqOPQNmt9oBgzQ_N6C3QP9x8'
        const supabase = createClient(supaUrl, supaAnonKey)

        roomCode = generateString(4);
        const gameSession = supabase.channel(roomCode, {
            config: {
                broadcast: {ack: true, self: true}
            }
        })

        function game(inst) {
            if(inst.action === 'init') {
                console.log('Game Start!')
                kaboom({
                    width: 1080,
                    height: 720,
                    canvas: canvas
                })
                scene('lobby', (args) => {
                    onDraw(() => {
                        drawSprite({
                            sprite: 'tavernBg',
                            pos: vec2(0, 0),
                            width: 1080,
                            height: 720
                        })
                        drawSprite({
                            sprite: 'bulletin',
                            pos: vec2(200, -80),
                            width: 700,
                            height: 800
                        })
                    })
                })
                loadSound('tavernBgm', 'audio/Tavern_Theme_2nd_loop.ogg')
                currentBgm = 'tavernBgm';
                loadSprite('bulletin', 'bulletin.png')
                loadSprite('tavernBg', 'tavern.png').then(() => {
                    go('lobby')
                })
            }
            if (inst.action === 'test') {
                console.log('test');
            }

            if (inst.action === 'add_player') {
                players.push({name: inst.player_name, charRace: inst.player_race, charClass: inst.player_class})
                console.log(players)
                onDraw(() =>{
                    players.forEach((player, index)=> {
                        drawText({
                            text: player.name + ' the ' + player.charRace + ' ' + player.charClass,
                            size: 24,
                            font: "sink",
                            width: 700,
                            pos: vec2(320, (36 * (index + 1)) + 200),
                            color: rgb(255, 255, 255),
                        })
                    })
                })
            }

        }

        gameSession.on('broadcast', {event: 'inst'}, (payload) => {
            // console.log(payload);
            game(payload.payload);
        }).subscribe((status) => {
            if(status === 'SUBSCRIBED') {
                gameSession.send({
                    type: 'broadcast',
                    event: 'inst',
                    payload: {action: 'init'}
                })
                console.log(gameSession);
            }
        })

        window.addEventListener('click', () => {
           // console.log('click')
           gameSession.send({
              type: 'broadcast',
               event: 'inst',
               payload: {action: 'test'}
           })
        })

    })

</script>

<div class="w-full h-screen flex flex-col items-center justify-center bg-black gap-4">
    <h1 class="text-2xl text-white font-extrabold">Keep this tab open to continue playing!</h1>
    <canvas id="canvas"></canvas>
    <p class="text-xl text-white font-bold">Room Code: {roomCode}</p>
    <button class="py-4 px-8 bg-white text-xl font-bold rounded-lg" on:click={() => {
        if(soundEnabled === 'Off') {
             soundEnabled = 'On'
             volume(0.5)
             bgmPlayer = play(currentBgm, {
                 volume: 0.5,
                 loop: true
             })
        }else if (soundEnabled === 'On') {
            soundEnabled = 'Off'
            bgmPlayer.pause()
            volume(0)
        }
    }}>Sound: {soundEnabled}</button>
</div>