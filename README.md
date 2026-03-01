# Sleeping-mode-on
<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Bus Alarm</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<style>
body{font-family:Arial;text-align:center;padding:15px}
input,button{padding:10px;margin:5px;width:95%}
button{background:#007bff;color:white;border:none;border-radius:5px}
#map{height:300px;margin-top:10px;border-radius:10px}
#results div{padding:8px;border:1px solid #ccc;margin:5px;cursor:pointer}
#results div:hover{background:#f2f2f2}
footer{margin-top:15px;font-size:12px;color:gray}
</style>
</head>
<body>

<h2>🚌 Bus Destination Alarm</h2>

<input type="text" id="search" placeholder="Search destination">
<button onclick="searchPlace()">Search</button>

<div id="results"></div>

<input type="number" id="radius" value="2000" placeholder="Alert Distance (meters)">
<button onclick="startAlarm()">Start Alarm</button>

<div id="map"></div>

<p id="distance"></p>

<footer>Create by Sumudu Rathnayake</footer>

<audio id="alarm" src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg"></audio>

<script>
let destLat=null,destLng=null,watchId=null,map,marker,userMarker;

map=L.map('map').setView([7.8731,80.7718],8);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
maxZoom:19
}).addTo(map);

function searchPlace(){
let q=document.getElementById("search").value;

fetch("https://nominatim.openstreetmap.org/search?format=json&q="+q)
.then(r=>r.json())
.then(data=>{
let results=document.getElementById("results");
results.innerHTML="";
data.slice(0,5).forEach(place=>{
let div=document.createElement("div");
div.innerText=place.display_name;
div.onclick=function(){
destLat=parseFloat(place.lat);
destLng=parseFloat(place.lon);
results.innerHTML="";
map.setView([destLat,destLng],14);
if(marker) map.removeLayer(marker);
marker=L.marker([destLat,destLng]).addTo(map);
alert("Destination Selected!");
}
results.appendChild(div);
});
});
}

function startAlarm(){

if(destLat===null){
alert("Select destination first!");
return;
}

if(Notification.permission!=="granted"){
Notification.requestPermission();
}

let radius=parseFloat(document.getElementById("radius").value);

watchId=navigator.geolocation.watchPosition(pos=>{

let lat=pos.coords.latitude;
let lng=pos.coords.longitude;

if(userMarker) map.removeLayer(userMarker);
userMarker=L.marker([lat,lng]).addTo(map);

let d=getDistance(lat,lng,destLat,destLng);

document.getElementById("distance").innerText=
"Distance: "+Math.round(d)+" m";

if(d<=radius){

document.getElementById("alarm").play();

if(navigator.vibrate){
navigator.vibrate([500,200,500,200,500]);
}

if(Notification.permission==="granted"){
new Notification("🔔 Wake Up! Destination Near!");
}

alert("Destination Reached!");

navigator.geolocation.clearWatch(watchId);
}

});

}

function getDistance(a,b,c,d){
const R=6371e3;
const φ1=a*Math.PI/180;
const φ2=c*Math.PI/180;
const Δφ=(c-a)*Math.PI/180;
const Δλ=(d-b)*Math.PI/180;
const x=Math.sin(Δφ/2)**2+
Math.cos(φ1)*Math.cos(φ2)*
Math.sin(Δλ/2)**2;
const y=2*Math.atan2(Math.sqrt(x),Math.sqrt(1-x));
return R*y;
}
</script>

</body>
</html>
