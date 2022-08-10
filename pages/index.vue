<template lang="pug">
div
  canvas
</template>

<script setup lang="ts">
import {
  WebGLRenderer,
  Color,
  Scene,
  Camera,
  MeshNormalMaterial,
  Mesh,
  CylinderBufferGeometry,
  Clock,
  Group,
} from "three";
import * as THREEx from "@ar-js-org/ar.js/three.js/build/ar-threex";
console.log(THREEx);
const renderer = new WebGLRenderer({
  antialias: true,
  alpha: true,
});
renderer.setClearColor(new Color(), 0);
renderer.setSize(640, 480);
renderer.domElement.style.position = "absolute";
renderer.domElement.style.top = "0px";
renderer.domElement.style.left = "0px";
document.body.appendChild(renderer.domElement);

const scene = new Scene();
scene.visible = false;
const camera = new Camera();
scene.add(camera);

const arToolkitSource = new THREEx.ArToolkitSource({
  sourceType: "webcam",
});

arToolkitSource.init(() => {
  setTimeout(() => {
    onResize();
  }, 2000);
});

addEventListener("resize", () => {
  onResize();
});

const arToolkitContext = new THREEx.ArToolkitContext({
  cameraParametersUrl: "data/camera_para.dat",
});

arToolkitContext.init(() => {
  camera.projectionMatrix.copy(arToolkitContext.getProjectionMatrix());
});

function onResize() {
  arToolkitSource.onResizeElement();
  arToolkitSource.copyElementSizeTo(renderer.domElement);
  if (arToolkitContext.arController !== null) {
    arToolkitSource.copyElementSizeTo(arToolkitContext.arController.canvas);
  }
}

const marker = new Group();
scene.add(marker);
const arMarkerControls = new THREEx.ArMarkerControls(arToolkitContext, marker, {
  type: "pattern",
  patternUrl: "data/pattern-23.patt",
  changeMatrixMode: "modelViewMatrix",
});

const mesh = new Mesh(new CylinderBufferGeometry(), new MeshNormalMaterial());
mesh.position.y = 1.0;
marker.add(mesh);

const clock = new Clock();
requestAnimationFrame(function animate() {
  requestAnimationFrame(animate);
  if (arToolkitSource.ready) {
    arToolkitContext.update(arToolkitSource.domElement);
    scene.visible = camera.visible;
  }
  const delta = clock.getDelta();
  mesh.rotation.x += delta * 1.0;
  mesh.rotation.y += delta * 1.5;
  renderer.render(scene, camera);
});
</script>

<style scoped lang="scss"></style>
