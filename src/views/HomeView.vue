<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, onUnmounted} from "vue"
import { OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import * as THREE from "three"
import gsap from "gsap"
import * as dat from "dat.gui"

//GUI
//tra le librerie piu popolari ci sono dat.GUI, control-panel, ControlKit, Guify, Oui

//in questo esempio useremo dat.GUI

// Inizializziamo la GUI (Puoi nascondere il pannello premendo H sulla tastiera)
const gui = new dat.GUI()

//const gui = new dat.GUI({ closed:true }) //Usa questo per iniziare con il pannello GUI "chiuso" ma apribile cliccando sul pulsante che appare sullo schermo
//const gui = new dat.GUI({width:400}) //Usa questo per impostare la grandezza iniziale del pannello (puo comunque essere modificata live con drag and drop)


const parameters = {
  color: 0xff0000,
  spin: ()=>{
    gsap.to(box.rotation, {duration:1, y: box.rotation.y + Math.PI * 2})
  }
}

//COLORI, TORNARE QUI IN SEGUITO
gui
  .addColor(parameters, "color")
  .onChange(()=>{
    boxMaterial.color.set(parameters.color)
  })

//BUTTONS, PER ATTIVARE FUNZIONI
gui.add(parameters, "spin")

//Possiamo aggiungere diversi tipi di elementi al pannello GUI:
// Folder - Per organizzare gli elementi contenuti nel pannello in cartelle (struttura piu pulita quando si hanno molti elementi)
// Range - per i numeri con un valore min e un max
// Select - per scegliere da una lista di valori
// Checkbox - per i valori booleani ( veri o falsi)
// Button - per "triggerare" (attivare) funzioni
// Color - per i colori (in vari formati)
// Text - per il testo semplice (simile a input type="text")

//per aggiungere un elemento alla GUI usiamo gui.add(), ma bisogna
//aggiungerlo dopo che l'oggetto da "perfezionare" è stato creato.
//Esempio: Andiamo dopo la creazione del box e aggiungiamo alla gui un parametro da "perfezionare"


let canvasRef = ref();

var controls

//Utilizziamo i valori del viewport per impostare la grandezza del canvas
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight
}

//Aggiungiamo l'event listner per il resize del canvas
window.addEventListener("resize", ()=>{
  //update sizes for the canvas
  sizes.width = window.innerWidth
  sizes.height = window.innerHeight

  //update the camera aspect ratio
  camera.aspect = sizes.width / sizes.height
  camera.updateProjectionMatrix()

  //update renderer
  renderer.setSize(sizes.width, sizes.height)
  //HANDLE PIXEL RATIO
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})


//HANDLE FULL SCREEN (Il codice aggiuntivo è per permettere il funzionamento corretto anche con browser Safari)
window.addEventListener("dblclick",()=>{
  const fullscreenElement = document.fullscreenElement ||  document.webkitFullscreenElement
  if(!fullscreenElement){
    if(canvasRef.value.requestFullscreen){
      canvasRef.value.requestFullscreen()
    }
    else if(canvasRef.value.webkitRequestFullscreen){
      canvasRef.value.webkitRequestFullscreen()
    }
    
  }
  else{
    if(document.exitFullscreen)
    {
      document.exitFullscreen()
    }
    else if(document.webkitExitFullscreen){
      document.webkitExitFullscreen()
    }
  }
})

let scene = new THREE.Scene()

let renderer


// BOX
const boxGeometry = new THREE.BoxGeometry(1,1,1,2,2,2)
const boxMaterial = new THREE.MeshBasicMaterial({color: 0xff0000}) 
const box = new THREE.Mesh(boxGeometry,boxMaterial)
box.position.set(0 , 0, 0)
scene.add(box)

//Aggiungiamo la posizione del box alla GUI
gui.add(box.position, "x",-3, 3, 0.01) //parametri: oggetto e proprietà da mettere a punto, quale attributo della proprietà modificare (e cosa appare a schermo), valore minimo, valore max, step (per rendere piu o meno preciso lo slider)

//possiamo anch aggiungere min max e step usando dei metodi, in questo modo: (cosi è possibile aggiungere anche metodi che non sono presenti nel "costruttore")
gui
  .add(box.position, "y")
  .min(-3)
  .max(3)
  .step(0.01)
  //ad esempio possiamo cambiare la label sulla gui in questo modo
  .name("asse y")

//o anche aggiungere i metodi dopo il costruttore in questo modo:
gui.add(box.position, "z",-3, 3, 0.01).name("asse z")


//Possiamo controllare booleani, ad esempio la proprietà visible:
gui.add(box, "visible")

// o il wireframe
gui.add(box.material, "wireframe")

//colori
// per i colori è un po piu complicato, perche con gui.add gui non 
//sa che noi stiamo parlando di colori, ma lo interpreta come stringa o 
//numero, mentre in threejs i colori sono in una classe (three.color) di threjs,
//quindi dobbiamo usare gui.addColor.
// Adesso però abbiamo un altro problema, non possiamo accedere ai colori direttamente dal materiale,
//perche come dicevamo prima i colori in threejs non sono stringhe, quindi dovremo creare un oggetto
//temporaneo per poter "storare" i valori dei colori. lo creiamo a inizio pagina, dopo aver istanziato GUI

// CAMERA
let camera = new THREE.PerspectiveCamera(
  75, 
  sizes.width / sizes.height, 
  0.1,
  100  
);
camera.position.set(0,0,3)
camera.lookAt(box.position)
scene.add(camera);

const clock = new THREE.Clock()

//Animate Loop
const animate = () =>{
  const elapsedTime = clock.getElapsedTime()

  controls.update()
  
  renderer.render(scene, camera)
}

onMounted(() => {
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value
  });
 
  renderer.setSize(sizes.width, sizes.height)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio,2)) // in questo modo il max valore di pixel ratio utilizzaro è 2 (altrimenti devi renderizzare troppi pixel, dispiego enorme di potenza gpu)
  renderer.render(scene, camera)
  renderer.setAnimationLoop(animate)

  //CONTROLS
  controls = new OrbitControls(camera, canvasRef.value)
  controls.enableDamping = true; // permette uno effetto smooth una volta che rilasciamo l'elemento, rallenta fino a fermarsi
});

onUnmounted(() => {
  renderer.setAnimationLoop(null)
});

</script>

<style>
canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>