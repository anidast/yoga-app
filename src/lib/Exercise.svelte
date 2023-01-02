<script lang="js">
  // import user from '../assets/user.jpg';
  // import instructor from "../assets/instructor.png";
  import { PlayIcon, PauseIcon } from "svelte-feather-icons";
  import { readable } from "svelte/store";
  import similarity from "compute-cosine-similarity";
  import { onMount } from "svelte";
  import { keypoint, threshold } from './stores';

  // var similarity = require( 'compute-cosine-similarity' );

  // let timeList = [50, 30, 25, 30, 30, 15, 15, 15, 20];
  let timeList = [10, 25, 10, 30, 10, 30, 10, 30, 10, 15, 10, 15, 10, 15, 10, 20];
  let poses = [
              // "child_pose", 
              // "swimmers", 
              "down_dog", 
              "chair_pose", 
              "crescent_lunge_left", 
              "crescent_lunge_right", 
              "plank", 
              "side_plank_right", 
              "side_plank_left", 
              // "low_cobra", 
              "namaste"];

    
  let timer = timeList[0]; // seconds

  const mstime = readable(new Date().getTime(), (set) => {
    let animationFrame;
    const next = () => {
      set(new Date().getTime());
      animationFrame = requestAnimationFrame(next);
    };
    if (window.requestAnimationFrame) {
      next();
      return () => cancelAnimationFrame(animationFrame);
    }
  });
  console.log($mstime);

  let start, time, toWait, minutes, seconds;
  let idx = 0;
  $: if (status == "play") {
    time = Math.floor(($mstime - start) / 1000);
    toWait = timer - time > 0 ? timer - time : 0;
    minutes = Math.floor(toWait / 60);
    seconds = toWait - minutes * 60;
    if (time == timer + 1 && idx+1 < timeList.length) {
      idx += 1;
      timer = timeList[idx];
      start = new Date().getTime();
      // console.log('Oke')
    }
  } else if (status == "pause") {
    start = new Date().getTime();
    timer = minutes * 60 + seconds;
  }

  $: if (idx % 2 == 1 && seconds % 5 == 0){
    takepicture()
  }

  // $: if (seconds == 10){
  //   takepicture()
  // }

  $: status = "start";
  $: error = null;
  $: errorVal = null;
  const width = 300;
  let height = 200;
  let videoSource = null;
  let canvas = null;
  let loading = false;
  
  let namaste = [[386.72223, 227.14464],
    [394.9284,  216.47475],
    [378.51605, 216.47475],
    [407.23773, 216.47475],
    [366.20673, 216.47475],
    [431.85632, 269.82422],
    [341.58813, 269.82422],
    [452.37177, 349.8484 ],
    [321.07263, 349.8484 ],
    [399.0315,  317.83868],
    [378.51605, 312.50378],
    [415.4439,  392.52795],
    [358.00055, 392.52795],
    [481.0935,  387.193  ],
    [296.4541,  387.193  ],
    [362.10364, 435.20752],
    [407.23773, 435.20752]]
  let prediction = [];
  $: rate = 0;

  onMount(async () => {
		status = "play";
    try {
      loading = true;
      const stream = await navigator.mediaDevices.getUserMedia({
        video: true,
      });
      videoSource.srcObject = stream;
      videoSource.play();
      loading = false;
      console.log(videoSource);
      start = new Date().getTime();
    } catch (error) {
      console.log(error);
    }
	});

  const obtenerVideoCamara = async () => {
    status = "play";
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

  function captureImage(canvas, player) {
    const ctx = canvas.getContext("2d");
    ctx.drawImage(player, 0, 0, canvas.width, canvas.height);
    // @ts-ignore
    return new Promise<Blob>((resolve) => {
      canvas.toBlob(resolve);
    });
  }

  // Creates interval
  const interval = setInterval(() => {
				// Get current time
				const now = new Date();

				// Format date as in mm/dd/aaaa hh:ii:ss
				// const dateTime = zeroFill((now.getMonth() + 1)) + '/' + zeroFill(now.getUTCDate()) + '/' + now.getFullYear() + ' ' + zeroFill(now.getHours()) + ':' + zeroFill(now.getMinutes()) + ':' + zeroFill(now.getSeconds());

				// Display the date and time on the screen using div#date-time
				// document.getElementById('date-time').innerHTML = dateTime;
			}, 1000);

  function takepicture() {
    const context = canvas.getContext("2d");
    canvas.width = width;
    canvas.height = height;
    context.drawImage(videoSource, 0, 0, width, height);

    // const data = canvas.toDataURL("image/png");
    // window.location.href = data;
    // console.log(data)
    
    const dataArray = new FormData();

    canvas.toBlob(function(blob){
        // link.href = URL.createObjectURL(blob);
        console.log(blob);
        dataArray.append("file", blob);
        // console.log(link.href); // this line should be here

        const getproducts = (async () => {
        const response = await fetch("http://127.0.0.1:5000/predict", {
            method: "POST",
            // headers: [["Content-Type", "multipart/form-data"]],
            body: dataArray,
        });
        let res = await response.json();
        if (response.status == 200) {
            prediction = res.prediction;
            let similar = [];

            // Pasangan keypoint untuk tiap anggota tubuh yang akan dibandingkan
            let iter = [[5, 6], [5, 7], [7, 9], [6, 8], [8, 10],
              [5, 11], [6, 12], [11, 12],
              [11, 13], [12, 14], [13, 15], [14, 16]];

            let body = [['Bahu'], ['Lengan Atas Kanan'], ['Lengan Bawah Kanan'], ['Lengan Atas Kiri'], 
              ['Lengan Bawah Kiri'], ['Bahu-Pinggul Kanan'], ['Bahu-Pinggul kiri'], ['Pinggul'],
              ['Paha Kanan'], ['Betis Kanan'], ['Paha Kiri'], ['Betis Kiri']];

            error = null;
            errorVal = null;
            
            // Hitung similaritas tiap anggota tubuh
            // iter.forEach(i => {
            //   similar.push(similarity(prediction[i[0]].concat(prediction[i[1]]), namaste[i[0]].concat(namaste[i[1]])))
            // });
            // iter.forEach(kp => {
            //   similar.push(similarity(prediction[kp[0]].concat(prediction[kp[1]]), keypoint[poses[i]][kp[0]].concat(keypoint[poses[i]][kp[1]])));
            //   // console.log(keypoint[poses[i]])
            // });

            for(let i=0;i<12;i++){
              let cosinesim = similarity(
                prediction[iter[i][0]].concat(prediction[iter[i][1]]), 
                keypoint[poses[(idx - 1) / 2]][iter[i][0]].concat(keypoint[poses[(idx - 1) / 2]][iter[i][1]])
              );
              similar.push(cosinesim);
              if (cosinesim < threshold[poses[(idx - 1) / 2]][i]){
                if (errorVal == null || threshold[poses[(idx - 1) / 2]][i] - cosinesim > errorVal){
                  error = body[i];
                  errorVal = threshold[poses[(idx - 1) / 2]][i] - cosinesim;
                }
                // console.log(body[i]);
              }
            }

            // Hitung rata-rata
            let total = similar.reduce((partialSum, a) => partialSum + a, 0);
            rate = total/12;
        }
    })();

        // fetch("http://127.0.0.1:5000/predict", {
        //     method: "POST",
        //     headers: [["Content-Type", "multipart/form-data"]],
        //     body: dataArray,
        // })
        // .then((response) => {
        //     console.log(response);
        // })
        // .catch((error) => {
        //     console.log(error);
        // });
    },'image/png')


    // fetch("http://127.0.0.1:5000/predict", {
    //   method: "POST",
    //   headers: [["Content-Type", "multipart/form-data"]],
    //   body: dataArray,
    // })
    //   .then((response) => {
    //     console.log(response);
    //   })
    //   .catch((error) => {
    //     console.log(error);
    //   });
  }
</script>

<section class="container is-fullhd">
  <div class="columns has-text-centered pt-4">
    <div class="column is-4 pl-6 pr-4">
      {#if status == "start"}
        <button on:click={obtenerVideoCamara} class="button pr my-3"
          >Mulai</button
        >
      {:else}
        <!-- svelte-ignore a11y-media-has-caption -->
        <video bind:this={videoSource} width="300" height="200" />
        <canvas id="canvas" bind:this={canvas} hidden/>
        <p class="timer mt-4">{minutes}:{seconds}</p>

        <!-- <button class="button pr my-3" on:click={takepicture}>
            Coba
        </button> -->

        {#if status == "play"}
          <button class="button pr my-3" on:click={() => (status = "pause")}>
            <PauseIcon class="pause" size={"4x"} />
          </button>
        {:else if status == "pause"}
          <button class="button pr my-3" on:click={() => (status = "play")}>
            <PlayIcon class="pause" size={"4x"} />
          </button>
        {/if}

        {#if idx % 2 == 1}
          <p class="rate is-size-2 my-2">
            Rate: <br />
            {rate.toFixed(4)}
          </p>
        {/if}
      {/if}
    </div>
    <div class="column is-8 pr-6 pl-4">
      <figure class="image px-5">
        <img src={`${poses[Math.floor(idx/2)]}.png`} alt="Instructor" />
        {#if idx % 2 == 1}
          {#if (error == null)}
            <button class="button is-fullwidth success is-size-4 px-6 my-4">
              Pertahankan posisi Anda!
            </button>
          {:else}
            <button class="button is-fullwidth alert is-size-4 px-6 my-4">
              Perhatikan posisi {error} Anda!
            </button>
          {/if}
        {:else}
          <button class="button is-fullwidth blue is-size-3 px-6 my-4 py-1">
            <b>PERSIAPAN POSISI</b>
          </button>
        {/if}
        
        </figure>
    </div>
  </div>
</section>

<style>
  video {
    border-radius: 1rem;
    -webkit-transform: scaleX(-1);
    transform: scaleX(-1);
  }

  .timer {
    color: #4b66f1;
    font-weight: 800;
    font-size: 4rem;
  }

  .pr {
    width: 5rem;
    height: 5rem;
    background-color: #14afc5;
    border-radius: 4rem;

    color: white;
    font-weight: 800;
  }

  .rate {
    color: #6f89fa;
    font-weight: 800;
  }

  .alert {
    background-color: transparent;
    border-color: #f96bbf;
    border-width: 2px;
    border-radius: 1rem;

    font-weight: 700;
    color: #f96bbf;
  }

  .success {
    background-color: transparent;
    border-color: #14afc5;
    border-width: 2px;
    border-radius: 1rem;

    font-weight: 700;
    color: #14afc5;
  }

  .blue {
    background-color: transparent;
    height: fit-content;
    border-color: #4b66f1;
    border-width: 2px;
    border-radius: 1rem;
    color: #4b66f1;
    letter-spacing: 0.2rem;
  }
</style>
