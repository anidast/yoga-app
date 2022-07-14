<script>
    // import user from '../assets/user.jpg';
    import instructor from '../assets/instructor.jpg';
    import { PlayIcon, PauseIcon} from 'svelte-feather-icons';
    import { readable } from 'svelte/store';
import Preparation from './Preparation.svelte';
	
	let timer = 30; // seconds
	
	const mstime = readable(new Date().getTime(), set => {
        let animationFrame
            const next = () => {
                set(new Date().getTime())
                animationFrame = requestAnimationFrame(next)
            }
            if (window.requestAnimationFrame) {
                next()
                return () => cancelAnimationFrame(animationFrame)
            }
	})
    console.log($mstime);
	
    let start, time, toWait, minutes, seconds;
	$: if (status == 'play'){
        time = Math.floor(($mstime - start) / 1000)
	    toWait = timer - time > 0 ? timer - time : 0
	    minutes = Math.floor(toWait/60)
	    seconds = toWait - minutes * 60
    } else if (status == 'pause'){
        start = new Date().getTime();
        timer = minutes * 60 + seconds
    }

    $: status = 'start';
    let videoSource = null;
    let loading = false;
    const obtenerVideoCamara = async () => {
        status = 'play';
        start = new Date().getTime();
        try {
        loading = true;
        const stream = await navigator.mediaDevices.getUserMedia({
            video: true,
        });
        videoSource.srcObject = stream;
        videoSource.play();
        loading = false;
        console.log(videoSource);
        } catch (error) {
        console.log(error);
        }
    };
</script>

<section class="container is-fullhd">
    <div class="columns has-text-centered pt-4">
        <div class="column is-4 pl-6 pr-4">
            {#if status == 'start'}
            <button on:click={obtenerVideoCamara} class="button pr my-3">Mulai</button>
            {:else}
            <!-- svelte-ignore a11y-media-has-caption -->
            <video bind:this={videoSource}  width="300" height="200"/>
                <p class="timer mt-4">{minutes}:{seconds}</p>

                {#if status == 'play'}
                <button class="button pr my-3" on:click={() => status='pause'}>
                    <PauseIcon class="pause" size={"4x"}/>
                </button>
                {:else if status == 'pause'}
                <button class="button pr my-3" on:click={() => status='play'}>
                    <PlayIcon class="pause" size={"4x"}/>
                </button>
                {/if}

                <p class="rate is-size-2 my-2">
                    Rate: <br>
                    0.0%
                </p>
            {/if}
        </div>
        <div class="column is-8 pr-6 pl-4">
            <figure class="image px-5">
                <img src={instructor} alt="Instructor">
                <button class="button is-fullwidth alert is-size-4 px-6 my-4">
                    Perhatikan posisi kaki Anda!
                </button>
            </figure>
        </div>
    </div>
</section>

<style>
    video{
        border-radius: 1rem;
    }

    .timer{
        color: #4B66F1;
        font-weight: 800;
        font-size: 4rem;
    }

    .pr{
        width: 5rem;
        height: 5rem;
        background-color: #14AFC5;
        border-radius: 4rem;


        color: white;
        font-weight: 800;
    }

    .rate{
        color: #6F89FA;
        font-weight: 800;
    }

    .alert{
        background-color: transparent;
        border-color: #F96BBF;
        border-width: 2px;
        border-radius: 1rem;

        font-weight: 700;
        color: #F96BBF;
    }
</style>