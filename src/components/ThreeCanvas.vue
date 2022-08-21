<template>
    <div tabindex="0" @keydown="handleKey">
        <canvas ref="c"></canvas>
    </div>
</template>

<script type="text/javascript">
import * as THREE from 'three'
import { GLTFLoader } from '../../node_modules/three/examples/jsm/loaders/GLTFLoader'

export default {
    name: "ThreeCanvas",
    props: {
        gltfModel: Object
    },
    data() {
        return {}
    },
    mounted() {
        const canvas = this.$refs.c

        this.gltfLoader = new GLTFLoader()
        //const url = "frogman.gltf"

        this.renderer = new THREE.WebGLRenderer({ canvas})
        this.initCamera()       
        //this.initiateBoxGeometry()
        this.scene = new THREE.Scene()
        //this.instantiateCubes()
        this.createLight()

        window.requestAnimationFrame(this.render)
    },
    
    methods: {
        dumpObject(obj, lines = [], isLast = true, prefix) {
                  const localPrefix = isLast ? '└─' : '├─';
                  lines.push(`${prefix}${prefix ? localPrefix : ''}${obj.name || '*no-name*'} [${obj.type}]`);
                  const newPrefix = prefix + (isLast ? '  ' : '│ ');
                  const lastNdx = obj.children.length - 1;
                  obj.children.forEach((child, ndx) => {
                    const isLast = ndx === lastNdx;
                    this.dumpObject(child, lines, isLast, newPrefix);
                  });
                  return lines;
        },
        loadModel(url) {
            this.gltfLoader.load(url, (gltf) => {

            const root = gltf.scene
            this.scene.add(root)
            })
        },
        initCamera() {
            const fov = 75
            const aspect = 2
            const near = 0.001
            const far = 5

            this.camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
            this.camera.position.z = 4
        },
        initiateBoxGeometry() {
            const boxWidth = 1
            const boxHeight = 1
            const boxDepth = 1
            this.geometry = new THREE.BoxGeometry(boxWidth, boxHeight, boxDepth)
        },
        instantiateCubes() {
            this.cubes = [
            this.makeInstance(this.geometry, 0x88aa88, -5),
            this.makeInstance(this.geometry, 0xaaaa88, 5),
        ]
        },
        handleKey(event) {
            switch(event.key) {
                case 'w': 
                    this.camera.translateZ(-0.5)
                    break;
                case 's': 
                    this.camera.translateZ(0.5)
                    break;
                case 'a': 
                    this.camera.translateX(-0.5)
                    break;
                case 'd': 
                    this.camera.translateX(0.5)
                    break;
                case 'q': 
                    this.camera.rotateY(0.5)
                    break;
                case 'e': 
                    this.camera.rotateY(-0.5)
                    break;
                case 'z': 
                    this.camera.translateY(0.5)
                    break;
                case 'h': 
                    this.camera.translateY(-0.5)
                    break;
            }
            this.renderer.render(this.scene, this.camera)
        },
        makeInstance(geometry, color, x) {
            const material = new THREE.MeshPhongMaterial({color})
            const cube = new THREE.Mesh(geometry, material)
            this.scene.add(cube)
            cube.position.x = x
            return cube
        },
        createLight() {
            const color = 0xFFFFFF
            const intensity = 1
            const light = new THREE.DirectionalLight(color, intensity)
            light.position.set(-1, 2, 4)
            this.scene.add(light)
        },
        renderCubes(time) {
            this.cubes.forEach((cube, ndx) => {
                const speed = 1 + ndx * .1
                const rot = time * speed
                cube.rotation.x = rot
                cube.rotation.y = rot
            })
        },
        //render(time) {
        render() {
            //time *= 0.001

            //renderCubes(time)

            this.renderer.render(this.scene, this.camera)

            window.requestAnimationFrame(this.render)
        }
    },
    watch: {
        gltfModel: {
            handler(newValue) {
                const blob = new Blob([newValue], { type: "text/plain;charset=utf-8"})
                this.loadModel(URL.createObjectURL(blob))
                
            }
        }
    }

}
</script>