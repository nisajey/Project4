<script>
import $ from 'jquery'
import { onUpdated } from 'vue';
import axios from 'axios'
import { ProgressCircle } from 'vue-progress-circle'
import "./css/default.css";

export default {
    data() {
        return {
            view: 'map',
            codes: [],
            neighborhoods: [],
            incidents: [],
            leaflet: {
                map: null,
                center: {
                    lat: 44.955139,
                    lng: -93.102222,
                    address: ""
                },
                zoom: 12,
                bounds: {
                    nw: { lat: 45.008206, lng: -93.217977 },
                    se: { lat: 44.883658, lng: -92.993787 }
                },
                neighborhood_markers: [
                    { location: [44.942068, -93.020521], marker: null, onMap: true },
                    { location: [44.977413, -93.025156], marker: null, onMap: true },
                    { location: [44.931244, -93.079578], marker: null, onMap: true },
                    { location: [44.956192, -93.060189], marker: null, onMap: true },
                    { location: [44.978883, -93.068163], marker: null, onMap: true },
                    { location: [44.975766, -93.113887], marker: null, onMap: true },
                    { location: [44.959639, -93.121271], marker: null, onMap: true },
                    { location: [44.947700, -93.128505], marker: null, onMap: true },
                    { location: [44.930276, -93.119911], marker: null, onMap: true },
                    { location: [44.982752, -93.147910], marker: null, onMap: true },
                    { location: [44.963631, -93.167548], marker: null, onMap: true },
                    { location: [44.973971, -93.197965], marker: null, onMap: true },
                    { location: [44.949043, -93.178261], marker: null, onMap: true },
                    { location: [44.934848, -93.176736], marker: null, onMap: true },
                    { location: [44.913106, -93.170779], marker: null, onMap: true },
                    { location: [44.937705, -93.136997], marker: null, onMap: true },
                    { location: [44.949203, -93.093739], marker: null, onMap: true }
                ]
            },
            crimeTypes: {
                "Narcotics": "other",
                "Burglary": "propCrime",
                "Graffiti": "propCrime",
                "Theft": "propCrime",
                "Auto Theft": "propCrime",
                "Agg. Assault Dom.": "violantCrime",
                "Agg. Assault": "violantCrime",
                "Vandalism": "propCrime",
                "Simple Asasult Dom.": "violantCrime",
                "Robbery": "propCrime",
                "Discharge": "violantCrime",
                "Arson": "propCrime",
                "Rape": "violantCrime",
                "Homicide": "violantCrime",
                "Proactive Police Visit": "other",
                "Community Engagement Event": "other",
                "Simple Assault Dom.": "violantCrime",
                "Other": "other",
                "Proactive Foot Patrol": "other",
                "Criminal Damage": "propCrime",
                "Community Event": "other",
                "THEFT": "propCrime",
                "Simple Assault Dom": "violantCrime",
                "Agg. Assault Dom": "violantCrime",
            }
        };
    },
    methods: {
        viewMap(event) {
            this.view = 'map';
        },
        updateCenter() {
            /*If it contains any letters then it must be a street address*/
            if(/[a-zA-Z]/.test(this.address)){
                let url = "https://nominatim.openstreetmap.org/?street='"+ this.address + "'&city='St.Paul'&state='Minnesota'&format=json&limit=1";
                console.log(url)
                this.getJSON(url)
                    .then((data) => {
                    console.log(data)
                
                    // keep in bounds
                    if (data[0].lat > 45.008206) {
                        data[0].lat = 45.008206;
                    }else if (data[0].lat < 44.883658) {
                        data[0].lat = 44.883658;
                    }

                    if (data[0].lon < -93.217977) {
                        data[0].lon = -93.217977;
                    }else if (data[0].lon > -92.993787) {
                        data[0].lon = -92.993787;
                    }
                    // Zoom on location
                    this.leaflet.map.flyTo(new L.LatLng(data[0].lat, data[0].lon), 17);
                }).catch((err) => { console.log(err);});
                //else it must be coordinates 
            }else {
                let latlong = this.address.replace(",", "").trim().split(' ');
                console.log(latlong);
                if (latlong[0]> 45.008206) {
                    latlong[0] = 45.008206;
                    }else if (latlong[0] < 44.883658) {
                        latlong[0] = 44.883658;
                    }

                    if (latlong[1] < -93.217977) {
                        latlong[1] = -93.217977;
                    }else if (latlong[1] > -92.993787) {
                        latlong[1] = -92.993787;
                    }
                    // Zoom on location
                    this.leaflet.map.flyTo(new L.LatLng(latlong[0], latlong[1]), 17);
            }
        },
        parseEnteredData() {

            let all_fields_filled = true;
            

            let user_case_number = document.getElementById("case_number").value;
            let user_date = document.getElementById("date").value;
            let user_time = document.getElementById("time").value;
            let user_code = parseInt(document.getElementById("code").value);
            let user_incident = document.getElementById("incident").value;
            let user_police_grid = parseInt(document.getElementById("police_grid").value);
            let user_neighborhood_number = parseInt(document.getElementById("neighborhood_number").value);
            let user_block = document.getElementById("block").value;
            
            let user_fields_collection = [user_case_number, user_date, user_time, user_code, user_incident, user_police_grid, user_neighborhood_number, user_block];
            
            let userData = {
                case_number: user_case_number,
                date: user_date,
                time: user_time,
                code: user_code,
                incident: user_incident,
                police_grid: user_police_grid,
                neighborhood_number: user_neighborhood_number,
                block: user_block
            };

            console.log(user_fields_collection[user_fields_collection.length-1]);
            
            for (let i=0; i<user_fields_collection.length; i++) {
                if (user_fields_collection[i] === "") {
                    all_fields_filled = false;
                    break;
                }
            }

            if (all_fields_filled) {
                console.log(userData);
                console.log("Beginning uploadJSON call");
                this.uploadJSON('PUT', 'http://127.0.0.1:8001/new-incident', userData)
                .then((err, data) => {
                    console.log("Ending uploadJSON call.");
                    if (err) {
                        console.log(err)
                    }
                    if (data) {
                        console.log(data);
                    }
                });
            } else {
                alert("You must fill in all fields!");
            }
            
        },
        viewNewIncident(event) {
            this.view = 'new_incident';
        },

        viewAbout(event) {
            this.view = 'about';
        },
        getData(){

let url_incidents = "http://127.0.0.1:8001/incidents";
let url_neighborhood = "http://127.0.0.1:8001/neighborhoods";
let url_code = "http://127.0.0.1:8001/codes"
Promise.all([this.getJSON(url_code),this.getJSON(url_neighborhood),this.getJSON(url_incidents)])
.then((results) => {
    this.codes = JSON.parse(JSON.stringify(results[0]));
    this.neighborhoods = JSON.parse(JSON.stringify(results[1]));
    this.incidents = JSON.parse(JSON.stringify(results[2])); 
    this.placeNeighborhoodMarker();
})
.catch((error) =>{
    alert("Table data not loaded");
    console.log(error);
})

},
placeNeighborhoodMarker(){
for (let i = 0; i < this.leaflet.neighborhood_markers.length; i++) {
    let marker = new L.Marker([this.leaflet.neighborhood_markers[i].location[0], this.leaflet.neighborhood_markers[i].location[1]]);
    let num = i+1;
    let url = 'http://localhost:8001/count?neighborhood=' + num;
    this.getJSON(url).then((results) => {
        console.log(results);
        marker.bindPopup(results[0].count + " incidents occured.");
        this.leaflet.neighborhood_markers[i].marker = marker;
        marker.addTo(this.leaflet.map);
    })
    .catch((error) =>{
    alert("Neighborhhod Marker Error:");
    console.log(error);
})

}
},
        getJSON(url) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    dataType: 'json',
                    url: url,
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        },


        uploadJSON(method, url, data) {
            return new Promise((resolve, reject) => {
                $.ajax({
                    type: method,
                    url: url,
                    contentType: 'application/json; charset=utf-8',
                    data: JSON.stringify(data),
                    dataType: 'text',
                    success: (response) => {
                        resolve(response);
                    },
                    error: (status, message) => {
                        reject({ status: status.status, message: status.statusText });
                    }
                });
            });
        },
        async deleteIncident(case_number, index) {

            // here we are trying to delete a incident
            // based on the case_number
            await axios
                .delete("http://localhost:8001/remove-incident?case_number=" + case_number)
                .then(response => {
                    if (response.status == 200) {
                        alert("Success! Case Number " + case_number + " Was Deleted.");
                        this.codes.pop(index);
                    } else {
                        alert("Error: Case Number " + case_number + " Was Deleted.");
                    }
                });

        }
    },
    async mounted() {

        // neighborhoods
        await axios
            .get("http://localhost:8001/neighborhoods")
            .then(response => {
                this.neighborhoods = response.data;
                console.log(this.neighborhoods)
            });

        this.leaflet.map = L.map('leafletmap').setView([this.leaflet.center.lat, this.leaflet.center.lng], this.leaflet.zoom);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
            minZoom: 11,
            maxZoom: 18
        }).addTo(this.leaflet.map);
        this.leaflet.map.setMaxBounds([[44.883658, -93.217977], [45.008206, -92.993787]]);

        let district_boundary = new L.geoJson();
        district_boundary.addTo(this.leaflet.map);

        this.getJSON('/data/StPaulDistrictCouncil.geojson').then((result) => {
            // St. Paul GeoJSON
            $(result.features).each((key, value) => {
                district_boundary.addData(value);
            });
        }).catch((error) => {
            console.log('Error:', error);
        });

        // codes
        await axios
            .get("http://localhost:8001/codes")
            .then(response => {
                this.codes = response.data;
            });


        // incidents
        await axios
            .get("http://localhost:8001/incidents")
            .then(response => {
                this.incidents = response.data;
            });
        this.getData();
        console.log("Finished calling all the APIs");

    }
}

</script>

<template>
    <div class="section">
        <section class="section has-background-primary">
            <div class="container">
                <h1 class="title is-1">
                    St. Paul Crime
                </h1>
                <p class="subtitle ">
                    Interactive Map and Database
                </p>
            </div>
        </section>
        <nav class="navbar" role="navigation" aria-label="main navigation">
            <div class="navbar-brand">
                <!-- navbar items and navbar burger... -->
            </div>
            <div class="navbar-menu">
                <div class="navbar-start">
                    <div class="navbar-item has-text-primary "
                        v-on:click="((view === 'map') ? 'selected' : 'unselected')" @click="viewMap">Map</div>
                    <div class="navbar-item has-text-primary"
                        v-on:click="((view === 'new_incident') ? 'selected' : 'unselected')" @click="viewNewIncident">
                        New Incident</div>
                </div>
                <div class="navbar-end">
                    <div class="navbar-item has-text-primary"
                        v-on:click="((view === 'about') ? 'selected' : 'unselected')" @click="viewAbout">About</div>
                </div>
            </div>
        </nav>

        <div class="content" v-show="view === 'map'">
            <div class="content">
                <div class="content">
                    <div id="leafletmap" class="content"></div>
                </div>
                <input id="location_search" class="input is-hovered" type="text" placeholder="Location: Street Address or Coordinate" v-model="address"/>
                <button class="button is-primary is-large" v-on:click="updateCenter"> Go </button>

                <center v-if="!incidents.length">
                    loading ...
                </center>

                <ul class="legend">
                    <li><span class="violantCrime"></span> Violant Crime</li>
                    <li><span class="propCrime"></span> Property Crime</li>
                    <li><span class="other"></span> Other</li> 
                </ul>

                <table v-if="incidents.length">
                    <tr>
                        <th>Crime</th>
                        <th>Neighborhood</th>
                        <th>Code</th>
                        <th>Address</th>
                        <th>Action</th>
                        <!-- <th>Date/Time</th> -->
                    </tr>
                    <tr v-for="(incidentx, index) in incidents" :class="this.crimeTypes[incidentx.incident]">
                        <td>{{ incidentx.incident }}</td>
                        <td>{{ neighborhoods[incidentx.neighborhood_number - 1].neighborhood_name }}</td>
                        <td>{{ incidentx.code }}</td>
                        <td>{{ incidentx.block }}</td>
                        <td><button class="button is-primary is-large"
                                v-on:click="deleteIncident(incidentx.case_number, index)"> Delete
                            </button></td>
                        <!-- <td>{{incidentx.date_time}}</td>  -->
                    </tr>
                </table>

            </div>
        </div>

        <div v-if="view === 'new_incident'">
            <!-- Replace this with your actual form: can be done here or by making a new component -->
            <div class="grid-container">
                <div class="grid-x grid-padding-x">
                    <h1 class="cell auto">New Incident Form</h1>
                </div>
                <div class="columns">
                    <input class="input is-hovered" id="case_number" type="text" placeholder="Case Number">
                    <input class="input is-hovered" id="date" type="text" placeholder="Date (2000-01-31)">
                    <input class="input is-hovered" id="time" type="text" placeholder="Time (00:00:00)">
                </div>
                
                <div class="columns">
                    <input class="input is-hovered" id="code" type="text" placeholder="Code">
                    <input class="input is-hovered" id="incident" type="text" placeholder="Incident">
                    <input class="input is-hovered" id="police_grid" type="text" placeholder="Police Grid">
                </div>
                <div class="columns">
                    <input class="input is-hovered" id="neighborhood_number" type="text" placeholder="Neighborhood Number">
                    <input class="input is-hovered" id="block" type="text" placeholder="Block">
                </div>

                <button class="button is-primary is-large" v-on:click="parseEnteredData"> Go </button>
            </div>
        </div>




        <div v-if="view === 'about'">
            <!-- Replace this with your actual about the project content: can be done here or by making a new component -->
            <div class="grid-container">
                <div class="grid-x grid-padding-x">
                    <h1 class="cell auto">About the Project</h1>
                </div>
            </div>

            <div class="section">
                <article class="media">
                    <figure class="media-left">
                        <p class="image is-flex is-align-items-center is-justify-content-center is-128x128">
                            <img class="is-rounded boarder" src="../images/jonathan_image.jpg" alt="Image of Jonathan">
                        </p>
                    </figure>

                    <div class="media-content">
                        <div class="content">
                            <p>
                                <strong>Jonathan Benusa</strong>
                                <br>
                                I'm a senior with a major in Computer Science and minor in Electrical Engineering and
                                Catholic Studies.
                            </p>
                        </div>
                        <nav class="level is-mobile">
                            <div class="level-left">
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-reply"></i></span>
                                </a>
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-retweet"></i></span>
                                </a>
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-heart"></i></span>
                                </a>
                            </div>
                        </nav>
                    </div>
                </article>

                <article class="media">
                    <figure class="media-left">
                        <p class="image is-flex is-align-items-center is-justify-content-center is-128x128">
                            <img class="is-rounded boarder" src="../images/Anisa_Image.jpeg" alt="Image of Anisa">
                        </p>
                    </figure>

                    <div class="media-content">
                        <div class="content">
                            <p>
                                <strong>Anisa Jeylani</strong>
                                <br>
                                I am a junior with a major in Computer Science and Minor in Statistics.
                            </p>
                        </div>
                        <nav class="level is-mobile">
                            <div class="level-left">
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-reply"></i></span>
                                </a>
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-retweet"></i></span>
                                </a>
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-heart"></i></span>
                                </a>
                            </div>
                        </nav>
                    </div>
                </article>

                <article class="media">
                    <figure class="media-left">
                        <p class="image is-flex is-align-items-center is-justify-content-center is-128x128">
                            <img class="is-rounded boarder" src="../images/jackson_image.jpg" alt="Image of Jackson">
                        </p>
                    </figure>

                    <div class="media-content">
                        <div class="content">
                            <p>
                                <strong>Jackson Meyer </strong>
                                <br>
                                I am a junior with a major in Computer Science and a minor in Philosophy.
                            </p>
                        </div>
                        <nav class="level is-mobile">
                            <div class="level-left">
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-reply"></i></span>
                                </a>
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-retweet"></i></span>
                                </a>
                                <a class="level-item">
                                    <span class="icon is-small"><i class="fas fa-heart"></i></span>
                                </a>
                            </div>
                        </nav>
                    </div>
                </article>
            </div>

            <div class="section">
                <p>Here is a list of the tools we used and a brief description of each:</p>
                <ul>
                    <li><strong>Nominatim</strong> - This API allows one to search for locations or retrieve their names based on their coordinates.</li>
                    <li><strong>Leaflet</strong> - This API allows one to render a map on their website and customize it how they see fit.</li>
                    <li><strong>VueJS</strong> - This JavaScript framework allows one to make websites easily and fast.</li>
                    <li><strong>St. Paul Crime</strong> - This API was created by this group in the previous project. It will retrieve data from a crime database.</li>
                    <li><strong>Bulma</strong> - This CSS framework allows for easy customization of HTML elements.</li>
                    <li><strong>Foundations</strong> - This CSS framework also allows for simple editing of HTML elements.</li>
                </ul>
                <br/><br/>
                <p>Additionally, here is a list of six interesting things we discovered using our application:</p>
                <ol>
                    <li>We learned that there are the most incidents in downtown St. Paul.</li>
                    <li>It seems like there is a lot of violent crimes in St. Paul.</li>
                    <li>St. Thomas is located in a couple of the neighborhoods with less incidents.</li>
                    <li>There are a lot of incidents around Frogtown.</li>
                    <li>The Leaflet API makes things easy to use and runs smoothly.</li>
                    <li>THe Nominatim API is really good as well. They work together very well.</li>
                </ol>
                <br/>
                <a href="https://stthomas.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=5561e8e4-f500-48ed-baf2-af7000dca736&start=0" target="_blank">Demo Video</a>
            </div>
        </div>

        

    </div>
</template>

<style>
#leafletmap {
    height: 500px;
}

.selected {
    background-color: rgb(10, 100, 126);
    color: white;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}

.unselected {
    background-color: rgb(200, 200, 200);
    color: black;
    border: solid 1px white;
    text-align: center;
    cursor: pointer;
}
</style>
