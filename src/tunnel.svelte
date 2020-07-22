<script>
  import * as THREE from "three";
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
  import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
  import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
  import { UnrealBloomPass } from "three/examples/jsm/postprocessing/UnrealBloomPass.js";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
  import { TweenMax } from "gsap";
  import { onMount } from "svelte";

  onMount(async () => {
    //Get window size
    var ww = window.innerWidth,
      wh = window.innerHeight;

    //Create a WebGL renderer
    var renderer = new THREE.WebGLRenderer({
      canvas: document.getElementById("render")
    });
    renderer.setSize(ww, wh);

    //Create an empty scene
    var scene = new THREE.Scene();
    scene.fog = new THREE.Fog(0x000000, 30, 150);

    //Create a perpsective camera
    var camera = new THREE.PerspectiveCamera(45, ww / wh, 0.1, 150);
    camera.position.y = 400;
    camera.position.z = 400;

    //Array of points
    var points = [
      [68.5, 185.5],
      [1, 262.5],
      [270.9, 281.9],
      [345.5, 212.8],
      [178, 155.7],
      [240.3, 72.3],
      [153.4, 0.6],
      [52.6, 53.3],
      [68.5, 185.5]
    ];

    //Convert the array of points into vertices
    for (var i = 0; i < points.length; i++) {
      var x = points[i][0];
      var y = Math.random() * 100;
      var z = points[i][1];
      points[i] = new THREE.Vector3(x, y, z);
    }
    //Create a path from the points
    var path = new THREE.CatmullRomCurve3(points);
    path.closed = true;

    // Define the precision of the finale tube, the amount of divisions
    var tubeDetail = 1600;
    // Define the precision of the circles
    var circlesDetail = 40;

    // Define the radius of the finale tube
    var radius = 4;
    // Get all the circles that will compose the tube
    var frames = path.computeFrenetFrames(tubeDetail, true);

    // Create an empty Geometry where we will put the particles
    var geometry = new THREE.Geometry();

    // Define a basic color
    var color = new THREE.Color(0x000000);

    // First loop through all the circles
    for (var i = 0; i < tubeDetail; i++) {
      // Get the normal values for each circle
      var normal = frames.normals[i];
      // Get the binormal values
      var binormal = frames.binormals[i];

      // Calculate the index of the circle (from 0 to 1)
      var index = i / tubeDetail;
      // Get the coordinates of the point in the center of the circle
      var p = path.getPointAt(index);

      // Loop for the amount of particles we want along each circle
      for (var j = 0; j < circlesDetail; j++) {
        // Clone the position of the point in the center
        var position = p.clone();

        // Calculate the angle for each particle along the circle (from 0 to Pi*2)
        var angle = (j / circlesDetail) * Math.PI * 2 + index * Math.PI * 5;
        // Calculate the sine of the angle
        var sin = Math.sin(angle);
        // Calculate the cosine from the angle
        var cos = -Math.cos(angle);

        // Calculate the normal of each point based on its angle
        var normalPoint = new THREE.Vector3(0, 0, 0);
        normalPoint.x = cos * normal.x + sin * binormal.x;
        normalPoint.y = cos * normal.y + sin * binormal.y;
        normalPoint.z = cos * normal.z + sin * binormal.z;
        // Multiple the normal by the radius
        normalPoint.multiplyScalar(radius);

        // We add the normal values for each point
        position.add(normalPoint);
        var color = new THREE.Color("hsl(" + index * 360 * 4 + ", 100%, 50%)");
        geometry.colors.push(color);
        geometry.vertices.push(position);
      }
    }

    // Material for the points
    var material = new THREE.PointsMaterial({
      size: 0.2,
      vertexColors: THREE.VertexColors
    });

    var tube = new THREE.Points(geometry, material);
    //Add tube into the scene
    scene.add(tube);

    var percentage = 0;

    function render() {
      percentage += 0.0005;
      var p1 = path.getPointAt(percentage % 1);
      var p2 = path.getPointAt((percentage + 0.01) % 1);
      camera.position.set(p1.x, p1.y, p1.z);
      camera.lookAt(p2);

      //Render the scene
      renderer.render(scene, camera);

      requestAnimationFrame(render);
    }
    requestAnimationFrame(render);
  });
</script>

<canvas id="render" />
