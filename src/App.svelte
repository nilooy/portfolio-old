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

  var vertices = [];

  for (var i = 0; i < 10000; i++) {
    var x = THREE.MathUtils.randFloatSpread(2000);
    var y = THREE.MathUtils.randFloatSpread(2000);
    var z = THREE.MathUtils.randFloatSpread(2000);

    vertices.push(x, y, z);
  }

  // moon
  var moonGeometry = new THREE.SphereGeometry(1, 3);
  var materialMoon = new THREE.MeshPhongMaterial({
    color: 0xf0ff00,
    wireframe: true
  });

  var meshMoon = new THREE.Mesh(moonGeometry, materialMoon);
  meshMoon.position.set(-3.2, 1.5, 0);
  meshMoon.scale.set(0.2, 0.2, 0.2);
  scene.add(meshMoon);

  var geometry = new THREE.BufferGeometry();
  geometry.setAttribute(
    "position",
    new THREE.Float32BufferAttribute(vertices, 3)
  );

  var material = new THREE.PointsMaterial({ color: 0x888888 });

  var points = new THREE.Points(geometry, material);

  scene.add(points);

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
  /* .banner-content h1 {
    font-size: 300px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 100;
  }
  .banner-content h2 {
    font-size: 100px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    font-weight: 100;
  } */

  .glow {
    font-size: 80px;
    color: #fff;
    text-align: center;
    -webkit-animation: glow 1s ease-in-out infinite alternate;
    -moz-animation: glow 1s ease-in-out infinite alternate;
    animation: glow 1s ease-in-out infinite alternate;
  }

  @-webkit-keyframes glow {
    from {
      text-shadow: 0 0 10px #fff, 0 0 20px #fff, 0 0 30px #50a00f,
        0 0 40px #50a00f, 0 0 50px #50a00f, 0 0 60px #50a00f, 0 0 70px #50a00f;
    }

    to {
      text-shadow: 0 0 20px #fff, 0 0 30px #33d31e, 0 0 40px #33d31e,
        0 0 50px #33d31e, 0 0 60px #33d31e, 0 0 70px #33d31e, 0 0 80px #33d31e;
    }
  }

  .glitch {
    position: relative;
    color: white;
    font-size: 6em;
    letter-spacing: 0.2em;
    /* Animation provies a slight random skew. Check bottom of doc
  for more information on how to random skew. */
    animation: glitch-skew 1s infinite linear alternate-reverse;
  }

  h1.glitch {
    font-size: 20em;
  }

  .glitch::before {
    content: attr(data-text);
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    left: 2px;
    text-shadow: -2px 0 #ff00c1;
    /* Creates an initial clip for our glitch. This works in
  a typical top,right,bottom,left fashion and creates a mask
  to only show a certain part of the glitch at a time. */
    clip: rect(44px, 450px, 56px, 0);
    /* Runs our glitch-anim defined below to run in a 5s loop, infinitely,
  with an alternating animation to keep things fresh. */
    animation: glitch-anim 5s infinite linear alternate-reverse;
  }
  .glitch::after {
    content: attr(data-text);
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    left: -2px;
    text-shadow: -2px 0 #00fff9, 2px 2px #ff00c1;
    animation: glitch-anim2 1s infinite linear alternate-reverse;
  }

  /* Creates an animation with 20 steaps. For each step, it calculates 
a percentage for the specific step. It then generates a random clip
box to be used for the random glitch effect. Also adds a very subtle
skew to change the 'thickness' of the glitch.*/
  @keyframes glitch-anim {
    0% {
      clip: rect(51px, 9999px, 24px, 0);
      transform: skew(0.92deg);
    }
    5% {
      clip: rect(56px, 9999px, 99px, 0);
      transform: skew(0.27deg);
    }
    10% {
      clip: rect(39px, 9999px, 50px, 0);
      transform: skew(1deg);
    }
    15% {
      clip: rect(99px, 9999px, 100px, 0);
      transform: skew(0.29deg);
    }
    20% {
      clip: rect(39px, 9999px, 36px, 0);
      transform: skew(0.35deg);
    }
    25% {
      clip: rect(93px, 9999px, 79px, 0);
      transform: skew(0.34deg);
    }
    30% {
      clip: rect(26px, 9999px, 23px, 0);
      transform: skew(0.76deg);
    }
    35% {
      clip: rect(9px, 9999px, 30px, 0);
      transform: skew(0.76deg);
    }
    40% {
      clip: rect(53px, 9999px, 24px, 0);
      transform: skew(0.21deg);
    }
    45% {
      clip: rect(40px, 9999px, 61px, 0);
      transform: skew(0.26deg);
    }
    50% {
      clip: rect(51px, 9999px, 31px, 0);
      transform: skew(0.69deg);
    }
    55% {
      clip: rect(84px, 9999px, 37px, 0);
      transform: skew(0.09deg);
    }
    60% {
      clip: rect(27px, 9999px, 36px, 0);
      transform: skew(0.14deg);
    }
    65% {
      clip: rect(84px, 9999px, 97px, 0);
      transform: skew(0.42deg);
    }
    70% {
      clip: rect(24px, 9999px, 61px, 0);
      transform: skew(0.08deg);
    }
    75% {
      clip: rect(32px, 9999px, 75px, 0);
      transform: skew(0.51deg);
    }
    80% {
      clip: rect(97px, 9999px, 85px, 0);
      transform: skew(0.91deg);
    }
    85% {
      clip: rect(14px, 9999px, 18px, 0);
      transform: skew(0.03deg);
    }
    90% {
      clip: rect(74px, 9999px, 15px, 0);
      transform: skew(0.56deg);
    }
    95% {
      clip: rect(84px, 9999px, 25px, 0);
      transform: skew(0.35deg);
    }
    100% {
      clip: rect(21px, 9999px, 37px, 0);
      transform: skew(0.33deg);
    }
  }
  @keyframes glitch-anim2 {
    0% {
      clip: rect(55px, 9999px, 33px, 0);
      transform: skew(0.42deg);
    }
    5% {
      clip: rect(64px, 9999px, 64px, 0);
      transform: skew(0.88deg);
    }
    10% {
      clip: rect(22px, 9999px, 37px, 0);
      transform: skew(0.92deg);
    }
    15% {
      clip: rect(20px, 9999px, 15px, 0);
      transform: skew(0.02deg);
    }
    20% {
      clip: rect(66px, 9999px, 20px, 0);
      transform: skew(0.64deg);
    }
    25% {
      clip: rect(50px, 9999px, 7px, 0);
      transform: skew(0.14deg);
    }
    30% {
      clip: rect(45px, 9999px, 69px, 0);
      transform: skew(0.05deg);
    }
    35% {
      clip: rect(9px, 9999px, 77px, 0);
      transform: skew(0.61deg);
    }
    40% {
      clip: rect(23px, 9999px, 82px, 0);
      transform: skew(0.5deg);
    }
    45% {
      clip: rect(34px, 9999px, 34px, 0);
      transform: skew(0.27deg);
    }
    50% {
      clip: rect(79px, 9999px, 66px, 0);
      transform: skew(0.55deg);
    }
    55% {
      clip: rect(3px, 9999px, 26px, 0);
      transform: skew(0.32deg);
    }
    60% {
      clip: rect(50px, 9999px, 75px, 0);
      transform: skew(0.03deg);
    }
    65% {
      clip: rect(73px, 9999px, 41px, 0);
      transform: skew(0.53deg);
    }
    70% {
      clip: rect(78px, 9999px, 10px, 0);
      transform: skew(0.04deg);
    }
    75% {
      clip: rect(12px, 9999px, 4px, 0);
      transform: skew(0.24deg);
    }
    80% {
      clip: rect(27px, 9999px, 5px, 0);
      transform: skew(0.87deg);
    }
    85% {
      clip: rect(16px, 9999px, 86px, 0);
      transform: skew(0.82deg);
    }
    90% {
      clip: rect(15px, 9999px, 84px, 0);
      transform: skew(0.62deg);
    }
    95% {
      clip: rect(88px, 9999px, 88px, 0);
      transform: skew(0.68deg);
    }
    100% {
      clip: rect(79px, 9999px, 6px, 0);
      transform: skew(0.08deg);
    }
  }
  @keyframes glitch-skew {
    0% {
      transform: skew(-1deg);
    }
    10% {
      transform: skew(-1deg);
    }
    20% {
      transform: skew(-1deg);
    }
    30% {
      transform: skew(-1deg);
    }
    40% {
      transform: skew(5deg);
    }
    50% {
      transform: skew(-3deg);
    }
    60% {
      transform: skew(-3deg);
    }
    70% {
      transform: skew(2deg);
    }
    80% {
      transform: skew(5deg);
    }
    90% {
      transform: skew(-4deg);
    }
    100% {
      transform: skew(2deg);
    }
  }
</style>

<div class="banner-content">
  <h1 class="glitch" data-text="Hi!">Hi!</h1>
  <h2 class="glitch" data-text="I'm Niloy">I'm Niloy</h2>
</div>

<h2>asdasd</h2>
