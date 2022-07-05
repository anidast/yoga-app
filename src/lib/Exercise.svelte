<script>
    import user from '../assets/user.jpg';
    import instructor from '../assets/instructor.jpg';
    import { PlayIcon, PauseIcon} from 'svelte-feather-icons';
</script>

<section class="container is-fullhd">
    <div class="columns has-text-centered pt-4">
        <div class="column is-4 pl-6 pr-4">
            <figure class="image">
                <img src={user} alt="User">
            </figure>

            <p class="timer mt-4">0:30</p>
            <button class="button pr my-3">
                <PauseIcon class="pause" size={"4x"}/>
            </button>
            <p class="rate is-size-2 my-2">
                Rate: <br>
                0.0%
            </p>
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
    img{
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