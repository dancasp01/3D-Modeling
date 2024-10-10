
<script setup>
//--------------Start of Script----------------//
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/Addons.js';
import { ref } from 'vue';

const scene = new THREE.Scene()
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.2, 2000)

const renderer = new THREE.WebGLRenderer()

renderer.setSize( window.innerWidth, window.innerHeight );
document.body.appendChild( renderer.domElement );

//--------------Textures----------------//

const earthTexture = new THREE.TextureLoader().load('erf.jpg')
const sunTexture = new THREE.TextureLoader().load('sun.jpg')
const moonTexture = new THREE.TextureLoader().load('moon.jpg')
const saturnTexture = new THREE.TextureLoader().load('saturn.jpg')
const saturnRingTexture = new THREE.TextureLoader().load('saturnRing.jpg')
const normalTexture = new THREE.TextureLoader().load('crazymoon.jpg')
const followJeff = ref(false)

//--------------Creation of Jeff----------------//

const jeffGeometry = new THREE.SphereGeometry(6,64,64);
const jeff = new THREE.Mesh(
    jeffGeometry,
    new THREE.MeshStandardMaterial({
        map: earthTexture,
        normalMap: normalTexture,
    })
);
jeff.position.set(60,0,-20)

//--------------Creation of sun----------------//

const sunGeometry = new THREE.SphereGeometry(12,128,128);
const sun = new THREE.Mesh(
    sunGeometry,
    new THREE.MeshStandardMaterial({
        map: sunTexture,
        normalMap: normalTexture,
    })
);
// sun.position.set(60,0,-20)


//--------------Creation of moon----------------//

const moonGeometry = new THREE.SphereGeometry(1.5,16,16);
const moon = new THREE.Mesh(
    moonGeometry,
    new THREE.MeshStandardMaterial({
        map: moonTexture,
        normalMap: normalTexture,
    })
);
moon.position.set(140,4,-22)

//--------------Creation of Saturn and its ring----------------//

const saturnGeometry = new THREE.SphereGeometry(6,64,64);
const saturn = new THREE.Mesh(
    saturnGeometry,
    new THREE.MeshStandardMaterial({
        map: saturnTexture,
        // normalMap: normalTexture,
    })
);
saturn.position.set(0,0,-20)

const ringGeometry = new THREE.RingGeometry(8, 12, 64);
const ringMaterial = new THREE.MeshBasicMaterial({

    map: saturnRingTexture,
    side: THREE.DoubleSide
});
const ring = new THREE.Mesh(ringGeometry, ringMaterial);
ring.rotation.x = Math.PI / 2;
ring.rotateX(-.05)
saturn.add(ring);

scene.add(jeff,sun,moon,saturn)

//--------------Lighting Grid & Controls----------------//

const ambient = new THREE.AmbientLight(0xffffff, 2.5);
// const gridHelper = new THREE.GridHelper(200, 50);

const controls = new OrbitControls(camera, renderer.domElement);

//--------------Star Creation & Removal----------------//

const stars = ref(300)
const currStars = ref(300)

function addStar(){
    const geometry = new THREE.SphereGeometry(.25)
    const material  = new THREE.MeshStandardMaterial({color: 0xffffff})
    const star = new THREE.Mesh(geometry,material)

    const [x,y,z] = Array(3).fill().map(() => THREE.MathUtils.randFloatSpread(200));

    star.position.set(x,y,z)
    scene.add(star)
}

Array(stars.value).fill().forEach(addStar) //Initial Star Creation

function removeStar(){
    const star = scene.children.find(child => child.id > 14)//14 is the first star, Jeff is somewhere below 14
    if (star === jeff) {
        alert('Jeff is not a star', jeff.id.toString())
    }
    scene.remove(star)
    // scene.add(ambient)
}

function checkStars(){
    while (stars.value != currStars.value){
    if (stars.value < currStars.value){
        addStar()
        stars.value++
    } else if (stars.value > currStars.value){
        removeStar()
        stars.value--
    }
}
scene.add(ambient);
return stars.value
}

//--------------Camera Positioning----------------//

camera.position.z = 15;

function getFPS() {
    let lastFramTime = performance.now();
    const now =  performance.now();
    const delta = (now - lastFramTime)/1000;
    lastFramTime = now;
    return Math.round(1/delta);
}

//--------------Animation----------------//

let speedX = ref(0)
let speedY = ref(0.01)

const jeffOrbitSpeed = ref(10)
const moonOrbitSpeed = ref(1)
const saturnOrbitSpeed = ref(.5)

function animate() {
    jeff.rotateX(speedX.value)
    jeff.rotateY(speedY.value)

    jeff.rotateY(-.0026)
    const jeffTime = Date.now() * jeffOrbitSpeed.value / 100000;
    jeff.position.x = Math.sin(jeffTime) * 50;
    jeff.position.z = Math.cos(jeffTime) * 50;


    moon.rotateY(.04)
    const moonTime = Date.now() * moonOrbitSpeed.value / 10000;
    moon.position.x = Math.sin(moonTime) * 10 + jeff.position.x;
    moon.position.z = Math.cos(moonTime) * 10 + jeff.position.z;

    saturn.rotateY(.04)
    const saturnTime = Date.now() * saturnOrbitSpeed.value / 10000;
    saturn.position.x = Math.sin(saturnTime) * 85;
    saturn.position.z = Math.cos(saturnTime) * 85;

    sun.rotateY(.001)

    controls.update();
	renderer.render( scene, camera);
}
renderer.setAnimationLoop( animate );
scene.add(ambient);

//--------------End of Script----------------//
</script>

<template>
    <div id="info">
        <p>Frames Per Second: {{ getFPS() }}</p>
        <p>Earth X - Rotation: {{ speedX }}</p>
        <input type="range" id="speedX" v-model="speedX" min="-.3" max=".3" step=".0001"/>
        <p>Earth Y - Rotation: {{ speedY }}</p>
        <input type="range" id="speedY" v-model="speedY" min="-.3" max=".3" step=".0001"/>
        <p>Stars: {{ checkStars() }}</p>
        <input type="range" id="speedY" v-model="currStars" min="0" max="2000" step="1"/>
        <p>Earth Orbit Speed: {{ jeffOrbitSpeed }}</p>
        <input type="range" id="speedY" v-model="jeffOrbitSpeed" min="-100" max="100" step="5"/>
        <p>Moon Orbit Speed: {{ moonOrbitSpeed }}</p>
        <input type="range" id="moonSpeed" v-model="moonOrbitSpeed" min="-100" max="100" step="5"/>
        <p>Follow Earth</p>
        <input type="checkbox" id="followJeff" v-model="followJeff"/>

        <p>Camera - X: {{ camera.position.x }}</p>
        <p>Camera - Y: {{ camera.position.y}}</p>
        <p>Camera - Z: {{ camera.position.z}}</p>

        <input type="button" id="reset" value="Reset Camera" @click="camera.position.set(0,0,15)"/>
    </div>
</template>

<style>
body {
    margin: 0;
    padding: 0;
    overflow-x: hidden; /* Hide horizontal scrollbar */
    overflow-y: auto;   /* Only show vertical scrollbar when necessary */
    width: 100%;
    height: 100%;
}
#info {
    background: black;
    color:white;
	position: absolute;
	top: 10px;
	text-align: center;
	z-index: 100;
	display:block;
    font-size: larger;
    caret-color: transparent;
}
</style>