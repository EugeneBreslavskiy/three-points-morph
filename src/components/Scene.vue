<template>
  <section class="scene" ref="scene" @click="increaseBlend()"></section>
</template>

<script>
    import * as THREE from 'three';
    import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader';
    import { VTKLoader } from 'three/examples/jsm/loaders/VTKLoader';
    import { PLYLoader } from 'three/examples/jsm/loaders/PLYLoader';
    import { TweenMax } from 'gsap';
    const OrbitControls = require('three-orbitcontrols');
  
    export default {
        name: 'Scene',
        data() {
            return {
                container: null,
                camera: null,
                controls: null,
                scene: null,
                renderer: null,
                models: [
                    {
                        url: '/static/models/whale/whale.ply',
                    },
                    {
                        url: '/static/models/moose/moose.ply'
                    }
                ],
                positions: [],
                points: null,
                blend: 0
            }
        },
        methods: {
            init() {
                this.container = this.$refs.scene;
                
                this.scene = new THREE.Scene();
                
                this.createCamera();
                this.createControls();
                this.createLights();
                this.createRenderer();
                this.loadModels();
                
                this.renderer.setAnimationLoop(() => {
                    this.update();
                    this.render();
                })

                window.addEventListener( 'resize', this.resizeWindowHandler, false );
            },
            createCamera() {
                this.camera = new THREE.PerspectiveCamera( 45, window.innerWidth / window.innerHeight, 1, 2000 );
                this.camera.position.set( 0, 0, 250 );
            },
            createControls() {
                this.controls = new OrbitControls( this.camera, this.container );
                this.controls.enableZoom = true;
                this.controls.enablePan = false;
                this.controls.autoRotate = false;
                this.controls.autoRotateSpeed = 1;
                this.controls.enableDamping = true;
                this.controls.dampingFactor = 0.5;
            },
            createLights() {
                const ambientLight = new THREE.AmbientLight( 0x404040, 2 );
                const directionalLight = new THREE.DirectionalLight( 0xffffff, 5 );
                directionalLight.position.set( 10, 10, 10 );
                
                this.scene.add( ambientLight, directionalLight );
            },
            createRenderer() {
                this.renderer = new THREE.WebGLRenderer( { antialias: true, alpha: true } );
                this.renderer.setSize( window.innerWidth, window.innerHeight );
                this.renderer.setPixelRatio( window.devicePixelRatio );
                this.renderer.setClearColor( 0xffffff, 0 ); 

                this.container.appendChild( this.renderer.domElement );
            },
            createObject(current, next) {
                let pointsGeometry = new THREE.BufferGeometry();
                pointsGeometry.addAttribute('position', new THREE.Float32BufferAttribute( current, 3 ));
                pointsGeometry.addAttribute('pos', new THREE.Float32BufferAttribute( next, 3 ));

                let pointsMaterial = new THREE.ShaderMaterial(
                    {
                        uniforms: {
                            blend: {
                                value: 0
                            }
                        },
                        vertexShader: `
                            uniform float blend;
                            
                            attribute vec3 pos;

                            void main() {
                                vec3 result = pos * blend + (1.0 - blend) * position;

                                gl_Position = projectionMatrix * viewMatrix * modelMatrix * vec4( result, 1.0 );
                            }
                            
                        `,
                        fragmentShader: `
                            uniform float blend;
                            void main() {
                                gl_FragColor = vec4( 0.0, 0.0, 0.0, 0.75 );
                            }
                        `
                    }
                );

                let points = new THREE.Points( pointsGeometry, pointsMaterial );

                this.points = points;

                this.scene.add( points );
            },
            loadModels() {
                let loader = new PLYLoader();

                loader.load(
                    this.models[0].url,
                    ( object ) => {
                        let vertices = object.attributes.position.array;

                        //vertices.splice(12000);

                        //console.log(vertices);

                        //console.log(`whale length ${ vertices.length }`);

                        let vert = Array.from(vertices).slice(0, 120000);

                        this.positions.push( vert );

                        loader.load(
                            this.models[1].url,
                            ( model ) => {
                                let vertices_2 = model.attributes.position.array;

                                let vert_2 = Array.from(vertices_2).slice(0, 120000);

                                // console.log(`moose length ${ vertices_2.length }`);

                                this.positions.push( vert_2 );

                                this.createObject(vert, vert_2);
                            }
                        )
                    }
                )
            },
            update() {
                this.controls.update();
            },
            render() {
                this.renderer.render( this.scene, this.camera );
            },
            resizeWindowHandler() {
                this.camera.aspect = window.innerWidth / window.innerHeight;
                this.camera.updateProjectionMatrix();

                this.renderer.setSize( window.innerWidth, window.innerHeight );
            },
            increaseBlend() {
                if (this.points.material.uniforms.blend.value === 0) {
                    TweenMax.to(this.points.material.uniforms.blend, 1.25, {
                        value: 1,
                        ease: 'expo'
                    })
                } else {
                    TweenMax.to(this.points.material.uniforms.blend, 1.25, {
                        value: 0,
                        ease: 'expo'
                    })
                }
            }
        },
        mounted() {
            this.init();
        }
    }
</script>

<style lang="sass">
  
  .scene
    width: 100%
    height: 100%
    position: absolute

</style>
