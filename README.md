# threejsGalaxyGenerator

This threejs project creates a spiral galaxy. It can be used to create other types of galaxies and space objects with edits.

A general overview of the pertinent code is given below:

To start, the style sheet, threejs library, orbitControls and dat.gui interface are imported.
```JavaScript
import './style.css'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import * as dat from 'dat.gui'
```

Next the base of the scene was setup, including dat.GUI.
```JavaScript
const gui = new dat.GUI()
const canvas = document.querySelector('canvas.webgl')
const scene = new THREE.Scene()

Next the size of the scene was set
```JavaScript
const sizes = {
    width: window.innerWidth,
    height: window.innerHeight
}
```

Then the camera was set up along with the controls.
```JavaScript
const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100)
camera.position.x = 3
camera.position.y = 3
camera.position.z = 3
scene.add(camera)
```

Next the renderer was setup.
```JavaScript
const renderer = new THREE.WebGLRenderer({
    canvas: canvas
})
renderer.setSize(sizes.width, sizes.height)
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
```

After this an eventListener was setup to listen for screen resizes.
```JavaScript
window.addEventListener('resize', () =>
{
    // Update sizes
    sizes.width = window.innerWidth
    sizes.height = window.innerHeight

    // Update camera
    camera.aspect = sizes.width / sizes.height
    camera.updateProjectionMatrix()

    // Update renderer
    renderer.setSize(sizes.width, sizes.height)
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
})
```

Next the animate functionality was setup.
```JavaScript
const clock = new THREE.Clock()

const tick = () =>
{
    const elapsedTime = clock.getElapsedTime()

    // Update controls
    controls.update()

    // Render
    renderer.render(scene, camera)

    // Call tick again on the next frame
    window.requestAnimationFrame(tick)
}

tick()
```

Then a test object was added to the scene.
```JavaScript
const cube = new THREE.Mesh(
    new THREE.BoxGeometry(1, 1, 1),
    new THREE.MeshBasicMaterial()
)
scene.add(cube);
```

The scene was rendered and if all is well the test cube is deleted and a generateGalaxy() function is created and then called to instantiate a galaxy upon page loads.
```JavaScript
const generateGalaxy = () => {
  console.log("generate the galaxy");
};
generateGalaxy();
```

Next a parameters object was created to contain the parameters of the galaxy.
```JavaScript
const parameters = {};
parameters.count = 1000;
parameters.size = 0.02;
```

Next random particles were created based on the count parameter above.
```JavaScript
const generateGalaxy = () => {
  const geometry = new THREE.BufferGeometry();
  const positions = new Float32Array(parameters.count * 3);
  for (let i = 0; i < parameters.count; i++) {
    const i3 = i * 3;
    positions[i3 + 0] = Math.random();
    positions[i3 + 1] = Math.random();
    positions[i3 + 2] = Math.random();
  }
//   console.log(positions);
};
```

The above generates a positions array of 1000 points with 3 vertices (x,y,z). Next an attribute is added to set the positions to the geometry.
```JavaScript
  geometry.setAttribute(
      'position',
      new THREE.BufferAttribute(positions, 3)
  )
```

After the initial geometry was set up the material was constructed.
```JavaScript
  const material = new THREE.PointsMaterial({
    size: parameters.size,
    sizeAttenuation: true,
    depthWrite: false,
    blending: THREE.AdditiveBlending,
  });
```

Then the points were added and the scene rendered which should show a cube of particles.
```JavaScript
  const points = new THREE.Points(geometry, material);
  scene.add(points);
};
```

***To spread the cube out and center it to the screen the following code would be needed***
```JavaScript
  for (let i = 0; i < parameters.count; i++) {
    const i3 = i * 3;
    positions[i3 + 0] = (Math.random() - 0.5) * 3;
    positions[i3 + 1] = (Math.random() - 0.5) * 3;
    positions[i3 + 2] = (Math.random() - 0.5) * 3;
  }
```

Once the base was finished tweaks were added.
```JavaScript
gui.add(parameters, "count").min(100).max(1000000).step(100).name('starsCount');
gui.add(parameters, "size").min(0.001).max(0.1).step(0.001).name('starsSize');
```

To enable the tweaks to work the generateGalaxy() had to be recalled by using an event listener.
```JavaScript
gui
  .add(parameters, "count")
  .min(100)
  .max(1000000)
  .step(100)
  .onFinishChange(generateGalaxy)
  .name("starsCount");
gui
  .add(parameters, "size")
  .min(0.001)
  .max(0.1)
  .step(0.001)
  .onFinishChange(generateGalaxy)
  .name("starsSize");
```

The above code allows the datGUI panel components to work but the old renders are not removed. This issue was corrected next by removing the gemotry, material and points outside of the generateGalaxy() function.
```JavaScript
let geometry = null;
let material = null;
let points = null;

const generateGalaxy = () => {
  geometry = new THREE.BufferGeometry();
 ...
  ...
    ...
    ...
    ...
    ...
  }
  //   console.log(positions);
  ...
  material = new THREE.PointsMaterial({
    ...
    ...
    ...
    ...
  });
  
  points = new THREE.Points(geometry, material);
  ...
};
```

The above code creates the geometry, material and points as empty objects and when generateGalaxy() is called those 3 variables are changed. Now, before creating new particles a test can be done to check if those variables already have a value. If so, they will be destroyed.
```JavaScript
const generateGalaxy = () => {

    /**
     * Destroy Particles
     */
    if(points !== null){
        geometry.dispose();
        material.dispose();
        scene.remove(points)
    }
    ...
 ```
 
 The parameters were updated.
 ```JavaScript
 const parameters = {};
parameters.count = 100000;
parameters.size = 0.01;
```

Next the shape of the galaxy was started upon starting with the radius.
```JavaScript 
parameters.radius = 5;

gui
  .add(parameters, "radius")
  .min(0.01)
  .max(20)
  .step(0.01)
  .onFinishChange(generateGalaxy)
  .name("galaxyRadius");
```

Next a vertex was created and the particles made to be randomly rendered upon that vertex, this will eliminate the cube.
```JavaScript
   const radius = Math.random() * parameters.radius;

    positions[i3 + 0] = radius;
    positions[i3 + 1] = 0;
    positions[i3 + 2] = 0;
  }
```

Next other vertices are added to form branches off a single central vertex.
```JavaScript
parameters.branches = 3;
gui
  .add(parameters, "branches")
  .min(2)
  .max(18)
  .step(1)
  .onFinishChange(generateGalaxy)
  .name("galaxyRadius");
```

Next the particles were positioned randomly on the various branches starting with determining the angles of the subsequent vertices.
```JavaScript
    const branchAngle =
      ((i % parameters.branches) / parameters.branches) * Math.PI * 2;
```

Next particles were positioned on a circle using the above angle.
```JavaScript
    positions[i3 + 0] = Math.cos(branchAngle) * radius;
    positions[i3 + 1] = 0;
    positions[i3 + 2] = Math.sin(branchAngle) * radius;
```

This gives 3 vertex with particles along each in randomized position. Next the spin is created to add rotation.
```JavaScript
parameters.spin = 1;
gui
  .add(parameters, "spin")
  .min(-5)
  .max(5)
  .step(0.001)
  .onFinishChange(generateGalaxy)
  .name("galaxySpin");
```

Next it is set up so that particles spin rotation increases the further they are from the center, just as what happens in a real galaxy with the further most stars spinning in a much wider orbit than the stars close the the galaxies center.
```JavaScript
    const spinAngle = radius * parameters.spin;

    const branchAngle =
      (((i % parameters.branches) + spinAngle) / parameters.branches) *
      Math.PI *
      2;
```

This creates the spiral shape. Next randomness is added to disperse the particles off of the vertex which up until now they have been confined upon.
```JavaScript
parameters.randomness = 0.2;
gui
  .add(parameters, "randomness")
  .min(0)
  .max(2)
  .step(0.001)
  .onFinishChange(generateGalaxy)
  .name("particleRandomness");
  
      const randomX = Math.random() * parameters.randomness;
    const randomY = Math.random() * parameters.randomness;
    const randomZ = Math.random() * parameters.randomness;

    positions[i3 + 0] = Math.cos(branchAngle) * radius + randomX;
    positions[i3 + 1] = 0 + randomY;
    positions[i3 + 2] = Math.sin(branchAngle) * radius + randomZ;
  }
```

The above code works to fatten the vertices into tubes but the particles are still not dispersed properly to tweaks are made to allow for star disperal to follow an exponential curve up to the value of 1 as far as distance off the vertex is concerned.
```JavaScript
parameters.randomnessPower = 3;

gui
  .add(parameters, "randomnessPower")
  .min(1)
  .max(10)
  .step(0.001)
  .onFinishChange(generateGalaxy)
  .name("particleRandomness");

    const randomX =
      Math.pow(Math.random(), parameters.randomnessPower) *
      (Math.random() < 0.5 ? 1 : -1);
    const randomY =
      Math.pow(Math.random(), parameters.randomnessPower) *
      (Math.random() < 0.5 ? 1 : -1);
    const randomZ =
      Math.pow(Math.random(), parameters.randomnessPower) *
      (Math.random() < 0.5 ? 1 : -1);
```



Next colors were added to the galaxy to have a differentation between the center and the outer rim.
```JavaScript
parameters.insideColor = "#ff6030";
parameters.outsideColor = "#ff6030";

gui
  .addColor(parameters, "insideColor")
  .onFinishChange(generateGalaxy)
  .name("insideColor");
gui
  .addColor(parameters, "outsideColor")
  .onFinishChange(generateGalaxy)
  .name("outsideColor");
```

To make the colors work the material needs to be updated and a new attributed added.
```JavaScript
    vertexColors: true,
    
  const colors = new Float32Array(parameters * 3)
  
    colors[i3 + 0] = 1;
    colors[i3 + 1] = 0;
    colors[i3 + 2] = 0;
    
  geometry.setAttribute("color", new THREE.BufferAttribute(colors, 3));
```

At this point the spiral galaxy should be rendered on the screen with color added. Next a color instance was created for the inside and outside colors.
```JavaScript
  const colorInside = new THREE.Color(parameters.insideColor);
  const colorOutside = new THREE.Color(parameters.outsideColor);
```

Now the colors are blended depending on the radius.
```JavaScript
    const mixedColor = colorInside.clone();
    mixedColor.lerp(colorOutside, radius / parameters.radius);

    colors[i3 + 0] = mixedColor.r;
    colors[i3 + 1] = mixedColor.g;
    colors[i3 + 2] = mixedColor.b;
```

***End Walkthrough
