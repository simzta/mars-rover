<script>
    import 'C:/Users/Serafima/Documents/Projects/C2C/mars-rover/src/styles/styles.css';

    /* Define Rover Options */
    // Allows user to choose a rover, present aray of dictionaries present rover choices.
    const roverOptions = [

        { value: 'all', name: 'All Rovers'}, // "value" is what we use in our code, "name" is what the user sees
        { value: 'perseverance', name: 'Perseverance' },
        { value: 'curiosity', name: 'Curiosity'},
        { value: 'opportunity', name: 'Opportunity'},
        { value: 'spirit', name: 'Spirit'}
    ];

    /* Define Camera Options */
    // Allows user to choose a camera, user filters photo results by cameras.  
    const cameraOptions = [

        { value: 'all', name: 'All Cameras' },
        { value: 'FHAZ', name: 'Front Hazard Avoidance Camera' },
        { value: 'RHAZ', name: 'Rear Hazard Avoidance Camera' },
        { value: 'MAST', name: 'Mast Camera' },
        { value: 'CHEMCAM', name: 'Chemistry and Camera Complex' },
        { value: 'MAHLI', name: 'Mars Hand Lens Imager' },

        { value: 'MARDI', name: 'Mars Descent Imager' },
        { value: 'NAVCAM', name: 'Navigation Camera' },
        { value: 'PANCAM', name: 'Panoramic Camera' },
        { value: 'MINITES', name: 'Miniature Thermal Emission Spectrometer' } 
    ]; 

    // Below are the default values for the rover and camera.
    let selectedRover = 'all';
    let selectedCamera = 'all';
    let startDate = '';
    let endDate = '';
    let photos = []; // Declareing photos as a  variable

    function getLast7Days() {
        
        const today = new Date();
        const lastWeek = new Date(today);
        lastWeek.setDate(today.getDate() - 6);

        return {
            start: lastWeek.toISOString().split('T')[0],
            end: today.toISOString().split('T')[0]
        };
    }

    async function fetchPhotos() {

        let rovers = selectedRover === 'all'
            ? ['perseverance', 'curiosity', 'opportunity', 'spirit']
            : [selectedRover];

        // Resets the photo array
        photos = [];

        if (!startDate && !endDate) {
            for (const rover of rovers) {
                let url = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/latest_photos?api_key=${import.meta.env.VITE_NASA_API_KEY}`;

                if (selectedCamera !== 'all') {
                    url += `&camera=${selectedCamera}`;
                }

                const res = await fetch(url);
                const data = await res.json();
                photos = photos.concat(data.latest_photos || []); // Concatenate the latest photos
            }
     
        }

        let { start, end } = { start: startDate, end: endDate };
        for (const rover of rovers) {

            let url = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/photos?api_key=${import.meta.env.VITE_NASA_API_KEY}`;
            url += `&earth_date=${end}`;

            if (selectedCamera !== 'all') {
                url += `&camera=${selectedCamera}`;
            }

            const res = await fetch(url);
            const data = await res.json();
            photos = photos.concat(data.photos || []); 
        }
    }
        

    // THE FOLLOWING CODE IS HTML--------------------------------------------------------------------------------------------------------------------

</script>
    <title>NASA Mars Rover Picture Collecter</title>

    <h1>NASA Mars Rover Image Search</h1>
    <h2>PHOTOS</h2>

<!-- NO TIME FOR THIS, BUT I THINK IT WOULD BE GOOD TO HAVE A BACK BUTTON TO GO BACK TO THE MAIN MENU.
     <h2>
    <nav>
      <a href="src/routes/MarsRoverHomePage.svelte"> Back to Main Menu</a>
    </nav>
</h2> --> 



<form on:submit|preventDefault={fetchPhotos}>

    <!-- CREATES THE SECTION WHERE THE PHOTOS WILL BE SHOWN -->
    <section class="image-results-container">
        <!-- To allow the alteration of each individual photo -->
        <div class= "image-results">
            {#if photos.length > 0}

                {#each photos as photo}
                    <img src={photo.img_src} alt={`Photo taken by ${photo.rover.name} rover on ${photo.earth_date}`} />
                {/each}

            {:else}

                <p>NO PHOTOS!</p>

            {/if}
        </div>
    </section>


    
  <label>
    
        Rover:
        <select bind:value={selectedRover}>
            {#each roverOptions as rover}
                <option value={rover.value}>{rover.name}</option>
            {/each}
        </select>

  </label>
  <br />



  <label>

        Camera:
        <select bind:value={selectedCamera}>
            {#each cameraOptions as camera}
                <option value={camera.value}>{camera.name}</option>
            {/each}
        </select>

  </label>
  <br />



  <label>

        Start Date:
        <input type="date" bind:value={startDate} />

  </label>
  <br />



  <label>

        End Date:
        <input type="date" bind:value={endDate} />

  </label>
  <br />



  <button type="submit">Search</button>

  <!-- INFORMATION BUTTON -->
  <div class="info-container">

    <button class="info-button">i</button>
    <p class="info-text">If you leave everything blank, you'll get the most recent photos from all rovers and all cameras (last 7 days).</p>
    
  </div>

</form>

