<script>
  import * as THREE from "three";
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
  import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
  import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
  import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass.js";
  import { TweenMax } from "gsap";
  import { getMousePos, moveJoint } from "./moveChar";

  var composer,
    renderer,
    mixer,
    neck,
    waist,
    mixamorigRightHand,
    mixamorigRightForeArm,
    mixamorigRightArm,
    mixamorigLeftLeg;

  var params = {
    exposure: 1,
    bloomStrength: 1,
    bloomThreshold: 0.4,
    bloomRadius: 2
  };

  var scene = new THREE.Scene();
  var camera = new THREE.PerspectiveCamera(
    40,
    window.innerWidth / window.innerHeight,
    1,
    1000
  );

  // Add lights
  let hemiLight = new THREE.HemisphereLight(0xffffff, 0xffffff, 0.61);
  hemiLight.position.set(0, 50, 0);
  // Add hemisphere light to scene
  scene.add(hemiLight);

  let d = 8.25;
  let dirLight = new THREE.DirectionalLight(0xffffff, 0.54);
  dirLight.position.set(-8, 12, 8);
  dirLight.castShadow = true;
  dirLight.shadow.mapSize = new THREE.Vector2(1024, 1024);
  dirLight.shadow.camera.near = 0.1;
  dirLight.shadow.camera.far = 1500;
  dirLight.shadow.camera.left = d * -1;
  dirLight.shadow.camera.right = d;
  dirLight.shadow.camera.top = d;
  dirLight.shadow.camera.bottom = d * -1;
  // Add directional Light to scene
  scene.add(dirLight);

  var renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const stacy_mtl = new THREE.MeshPhongMaterial({
    color: 0x00ff00,
    skinning: true,
    wireframe: true
  });

  var renderScene = new RenderPass(scene, camera);

  var bloomPass = new UnrealBloomPass(
    new THREE.Vector2(window.innerWidth, window.innerHeight),
    0.5,
    0.4,
    0.85
  );
  bloomPass.threshold = params.bloomThreshold;
  bloomPass.strength = params.bloomStrength;
  bloomPass.radius = params.bloomRadius;

  composer = new EffectComposer(renderer);
  composer.addPass(renderScene);
  composer.addPass(bloomPass);

  var loader = new GLTFLoader();

  loader.load(
    "fly.glb",
    function(gltf) {
      var obj = gltf.scene;

      obj.position.set(0, 0, 0);
      obj.position.x = 1.3;
      obj.position.y = -1.5;

      obj.traverse(o => {
        if (o.isMesh) {
          o.castShadow = true;
          o.receiveShadow = true;
          o.material = stacy_mtl;
        }

        console.log(o.name);
        if (o.isBone && o.name === "mixamorigNeck") {
          neck = o;
        }
        if (o.isBone && o.name === "mixamorigSpine") {
          waist = o;
        }
        if (o.isBone && o.name === "mixamorigRightHand") {
          mixamorigRightHand = o;
        }
        if (o.isBone && o.name === "mixamorigRightForeArm") {
          mixamorigRightForeArm = o;
        }
        if (o.isBone && o.name === "mixamorigRightArm") {
          mixamorigRightArm = o;
          mixamorigRightArm.rotation.z = THREE.MathUtils.degToRad(-60);
        }
        if (o.isBone && o.name === "mixamorigLeftLeg") {
          mixamorigLeftLeg = o;
        }
      });

      //   mixamorigRightForeArm.rotation.y = THREE.Math.degToRad(10);
      //   mixamorigRightForeArm.rotation.x = THREE.Math.degToRad(29);
      //   mixamorigRightForeArm.rotation.z = THREE.Math.degToRad(-149);

      TweenMax.from(composer.passes[1], 2, {
        radius: 2,
        yoyo: true,
        repeat: -1,
        ease: "Power2.easeInOut"
      });

      TweenMax.from(composer.passes[1], 2, {
        strength: 2,
        yoyo: true,
        repeat: -1,
        ease: "Power2.easeInOut"
      });

      TweenMax.from(mixamorigRightForeArm.rotation, 0.5, {
        z: THREE.Math.degToRad(-109),
        yoyo: true,
        repeat: -1,
        ease: "power1.none"
      });

      TweenMax.to(mixamorigLeftLeg.rotation, 0.5, {
        x: THREE.Math.degToRad(-30),
        yoyo: true,

        ease: "power1.none"
      });

      TweenMax.from(obj.position, 3, {
        y: 1.9,
        ease: "expo.out"
      });

      scene.add(obj);
    },
    undefined,
    function(error) {
      console.error(error);
    }
  );

  document.addEventListener("mousemove", function(e) {
    var mousecoords = getMousePos(e);
    if (neck && waist) {
      moveJoint(mousecoords, neck, 50);
      moveJoint(mousecoords, waist, 30);
    }
  });

  camera.position.z = 5;

  var animate = function() {
    requestAnimationFrame(animate);

    // renderer.render(scene, camera);
    composer.render();
  };

  animate();
</script>

<style>
  h1,
  h2 {
    margin: 0;
    padding: 0;
  }

  canvas {
    width: 100%;
    height: 100%;
  }

  .banner-content {
    position: absolute;
    color: white;
    text-align: left;
    top: 20%;
    left: 200px;
  }
  .banner-content h1 {
    font-size: 300px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 100;
  }
  .banner-content h2 {
    font-size: 100px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 100;
  }
</style>

<div class="banner-content">
  <h1>Hi!</h1>
  <h2>I'm Niloy</h2>
</div>

<h2>asdasd</h2>
