

# Import 3d models

```ts
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';  
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader';  
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader';


const GLTF_MODEL = await useLoader(GLTFLoader, '/models/house_web.gltf');
const GLB_MODEL = await useLoader(GLTFLoader, '/models/house_web.glb');
const FBX_MODEL = await useLoader(FBXLoader, '/models/house_web.fbx');
const OBJ_MODEL = await useLoader(OBJLoader, '/models/house_web.obj');
```

# Create initial scene

## camera.ts

```ts
import * as THREE from "three";  
  
type Options = {  
	fov?: number;  
	aspect?: number;  
	near?: number;
	far?: number;  
	position?: { x?: number; y?: number; z?: number; }  
}  
export default class Camera {  
	instance: THREE.PerspectiveCamera;  
  
	constructor(opt: {} = {}) {  
		this.instance = this.init(opt);  
	}  
  
	init(opt: Options = {}) {  
		const camera = new THREE.PerspectiveCamera(  
			opt.fov ?? 70,  
			opt.aspect ?? innerWidth / innerHeight,  
			opt.near ?? 1,  
			opt.far ?? 1500,  
		)  
	  
		camera.position.x = opt.position?.x ?? 0;  
		camera.position.y = opt.position?.y ?? 0;  
		camera.position.z = opt.position?.z ?? 50;  
		  
		return camera;  
	}  
  
	get camera() {  
		return this.instance;  
	}  
}
```

## scene.ts

```ts
import * as THREE from "three";  
  
export default class Scene {  
	instance: THREE.Scene;  
	  
	constructor(opt: {} = {}) {  
		this.instance = this.init(opt);  
	}  
	  
	init(opt: {} = {}) {  
		const scene = new THREE.Scene()  
		return scene;  
	}  
  
	get scene() {  
		return this.instance;  
	}  
}
```

## renderer.ts

```ts
import * as THREE from "three";  
  
export default class Renderer {  
	instance: THREE.Renderer;  
	  
	constructor(opt: {} = {}) {  
		this.instance = this.init(opt);  
	}  
	  
	init(opt: {} = {}) {  
		const renderer = new THREE.WebGLRenderer({antialias: true})  
		  
		renderer.shadowMap.enabled = true  
		renderer.shadowMap.type = THREE.PCFSoftShadowMap  
		renderer.setSize(window.innerWidth, window.innerHeight)  
		renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));  
		  
		return renderer;  
	}  
	  
	get renderer() {  
		return this.instance;  
	}  
}
```

## ambient.ts

```ts
import * as THREE from "three";  
  
export default class Ambient {  
	instance: THREE.AmbientLight;  
	  
	constructor(opt: {} = {}) {  
		this.instance = this.init(opt);  
	}  
	  
	init(opt: {} = {}) {  
		const ambientLight = new THREE.AmbientLight('#ffffff', 0.5)  
		  
		return ambientLight;  
	}  
	  
	get light() {  
		return this.instance;  
	}  
}
```

## directional.ts

```ts
import * as THREE from "three";  
  
export default class Directional {  
	instance: THREE.DirectionalLight;  
	  
	constructor(opt: {} = {}) {   
	this.instance = this.init(opt);  
}  
  
init(opt: {} = {}) {  
	const directionalLight = new THREE.DirectionalLight('#ff0000', 1);  
	directionalLight.position.y = 20  
	directionalLight.position.z = 20  
	  
	directionalLight.castShadow = true  
	  
	directionalLight.shadow.mapSize.width = 2048  
	directionalLight.shadow.mapSize.height = 2048  
	directionalLight.shadow.camera.far = 50  
	directionalLight.shadow.camera.near = 1  
	directionalLight.shadow.camera.top = 20  
	directionalLight.shadow.camera.right = 20  
	directionalLight.shadow.camera.bottom = -20  
	directionalLight.shadow.camera.left = -20  
	  
	return directionalLight;  
}  
  
	get light() {  
		return this.instance;  
	}  
}
```


## model.ts

```ts
import * as THREE from "three";  
import {GLTFLoader} from 'three/examples/jsm/loaders/GLTFLoader';  
  
export default class House {  
	instance: THREE.Mesh;  
  
	constructor(opt: {} = {}) {  
		this.instance = this.init(opt);  
	}  
  
	async init(opt: {} = {}) {  
		return new Promise((resolve, reject) => {  
			const loader = new GLTFLoader();  
			  
			loader.load('src/assets/models/house/house.gltf', (gltf) => {  
				const house = gltf.scene;  
		
				house.children[12].material = new THREE.MeshStandardMaterial({
					color: '#1be804'
				});  
				
				house.children[12].material.metalness = 0.69;  
		
				house.position.z = 20;  
				house.rotation.y = 0.3;  
				house.rotation.x = 0.3;  
				resolve(house);  
			});  
		})  
	}  
  
	get model() {  
		return this.instance;  
	}  
}  
```
## main.ts

```js
import Camera from "./modules/camera";  
import Scene from './modules/scene';
import Renderer from "./modules/renderer";
import AmbientLight from './modules/light/ambient';  
import DirectionalLight from "./modules/light/directional";

import House from "./models/house";

/** Cцена */  
const camera = new Camera().camera;  
const scene = new Scene().scene;  
const renderer = new Renderer().renderer;  
const ambientLight = new AmbientLight().light;  
const directionalLight = new DirectionalLight().light;

function tick(): void {  
	renderer.render(scene, camera)  
	  
	window.requestAnimationFrame(tick)  
}

function resizeHandler() {  
	camera.aspect = window.innerWidth / window.innerHeight  
	camera.updateProjectionMatrix()  
	  
	renderer.setSize(window.innerWidth, window.innerHeight)  
	renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))  
}

async function addHouse() {  
	house = await new House().model;  
	  
	scene.add(house);  
}

async function main() {
	renderer.render(scene, camera);  
	
	scene.add(ambientLight);  
	scene.add(directionalLight);  
	  
	document.body.appendChild(renderer.domElement);
	
	window.addEventListener('resize', resizeHandler, false);  
	
	tick();  
	
	await addHouse();
}  
  
main();
```