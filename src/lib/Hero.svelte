<script>
    import hero from '../assets/yoga.jpg';
</script>

<section class="section">
    <div class="columns py-6">
        <div class="column is-5 py-6">
            <p class="title-yoga">Hidup Sehat dengan Yoga</p>
            <p class="subtitle my-5">Berlatih yoga sendiri dimanapun dan kapanpun bisa kamu lakukan.</p>
            <button class="button py-5 px-6"><a href="/">Mulai Latihan</a></button>
        </div>
        <div class="column is-7 py-4">
            <figure class="image">
                <img src={hero} alt="Yoga">
            </figure>
        </div>
    </div>
</section>

<style>
    .title-yoga{
        font-size: 4rem;
        font-weight: 900;
        color: #14AFC5;
    }

    .subtitle{
        color: #6F89FA;
    }

    .button{
        background-color: #4B66F1;
        border-radius: 2rem;
    }

    .button a{
        font-weight: 800;
        color: white;
        text-decoration: none;
        letter-spacing: 1px;
    }
</style>