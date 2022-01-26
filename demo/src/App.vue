<!--
 * @Author: lough
 * @Date: 2021-12-12 09:44:54
 * @LastEditors: Please set LastEditors
 * @LastEditTime: 2022-01-04 14:16:21
 * @Description:
-->

<template>
  <div style="position: relative">
    <div
      id="container"
      style="width: 80%; height: 80%; position: relative; overflow: hidden"
    />

    <button class="btn" @click="btn">init</button>
    <button class="btn1" @click="btn1">init1</button>
  </div>
</template>
<script  id="vertexShader2" type="x-shader/x-vertex">
`	varying vec2 vUv;
			   attribute float percent;
			   uniform float u_time;
			   uniform float number;
			   uniform float speed;
			   uniform float length;
			   varying float opacity;
			   uniform float size;
			   void main()
			   {
			       vUv = uv;
			       vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
			       float l = clamp(1.0-length,0.0,1.0);
			       gl_PointSize = clamp(fract(percent*number + l - u_time*number*speed)-l ,0.0,1.) * size * (1./length);
			       opacity = gl_PointSize/size;
			       gl_Position = projectionMatrix * mvPosition;
			   }`;
</script>
<script>
import * as THREE from "three";
import { Line2 } from "three/examples/jsm/lines/Line2";
import { LineMaterial } from "three/examples/jsm/lines/LineMaterial.js";
import { LineGeometry } from "three/examples/jsm/lines/LineGeometry.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { TWEEN } from "three/examples/jsm/libs/tween.module.min.js";

export default {
  data() {
    return {
      posArr: [
        {
          x: -2.2050067398610094,
          y: 2.609370233463886,
          z: -3.650908415980897,
        },
        {
          x: -2.1965610576118175,
          y: 2.1955955192304506,
          z: -3.9184792759587768,
        },
        {
          x: -2.2290975556080355,
          y: 2.6054406912933263,
          z: -3.639066211507457,
        },
        {
          x: 0.5738958419746141,
          y: -0.44114968930852216,
          z: 4.9473255920938985,
        },
        { x: -0.9326350073394328, y: 2.8399222968004114, z: -4.00812091773949 },
        { x: 3.469198597393574, y: 1.2295167303380952, z: -3.3842206934036057 },
        {
          x: -2.4019084876611916,
          y: -2.190220428765315,
          z: 3.7991801866087123,
        },
        { x: -2.49363689878109, y: -4.099696049856375, z: 1.4050862307450966 },
        { x: -2.3729307780326305, y: 2.840227787960863, z: 3.3618901878497454 },
        {
          x: -2.0636200279017873,
          y: 0.7444294629976027,
          z: -4.493027615657812,
        },
        { x: 0.47725894517680106, y: 2.4327372143508037, z: -4.34212085796347 },
        {
          x: -2.4777001955161246,
          y: -1.2092952460724242,
          z: 4.171163716394502,
        },
        {
          x: -0.03915748918627658,
          y: -0.008362945319338826,
          z: 4.999839672648135,
        },
        { x: 1.5223738738260317, y: -1.032865814102439, z: -4.649254348640267 },
        {
          x: -0.26640112020426315,
          y: -4.314854187280748,
          z: 2.5121830716848077,
        },
        {
          x: -4.031470206741836,
          y: -2.606648761952297,
          z: -1.3973654511134501,
        },
        { x: 0.8544382232162094, y: 1.5274953155132989, z: 4.683662390031124 },
        { x: 3.0409624989238546, y: 1.76433738825175, z: -3.555230043268055 },
        {
          x: -4.721251023266457,
          y: 1.2354922989397954,
          z: -1.0878177947459262,
        },
        { x: 2.1518961827021106, y: 3.891904027152385, z: -2.285262755638206 },
        { x: 0.8501960736517479, y: -2.851729208821255, z: -4.018060123480341 },
        { x: 2.5631840141785176, y: 4.263234820997851, z: -0.5048926326370041 },
        {
          x: -0.4580143454812531,
          y: -2.6523265200067385,
          z: 4.213714144386437,
        },
      ],
      radius: 5,
      planGeometry: new THREE.PlaneBufferGeometry(1, 1), //默认在XOY平面上
      group: null,
      groupDots: null,
      groupLines: null,
      groupHalo: null,
      aGroup: null,
      WaveMeshArr: [], // 所有波动光圈集合
      animateDots: [],
      map: null,
      width: null,
      height: null,
      renderer: null,
      camera: null,
      scene: null,
      controls: null,
      uniforms2: {
        u_time: { value: 0.0 },
      },
      globeTextureLoader: null,
      camaPositions: [
        { x: 5, y: -20, z: 200 }, //远处
        { x: 0, y: 0, z: 20 }, //近处
      ],
      camaPositions1: [
        {
          x: 0,
          y: 0,
          z: 20,
        },
        {
          x: 0.5,
          y: 1,
          z: 8,
        },
      ],
      camaPositions2: [
        {
          x: 0.5,
          y: 1,
          z: 8,
        },
        {
          x: 0,
          y: 0,
          z: 20,
        },
      ],
      initFlag: false,
      stars: null,
      vIndex: 0,
      fragmentShader2: ``,
      vertexShader2: ``,
    };
  },
  mounted() {
    this.onload();
  },
  methods: {
    btn() {
      this.cameraCon1(3000);
    },

    btn1() {
      let tweena2 = this.cameraCon2(3000);
      tweena2.start();
    },
    onload() {
      const Dom = document.querySelector("#container");
      this.width = Dom.clientWidth;
      this.height = Dom.clientHeight;
      this.globeTextureLoader = new THREE.TextureLoader();
      this.group = new THREE.Group();
      this.groupDots = new THREE.Group();
      this.groupLines = new THREE.Group();
      this.groupHalo = new THREE.Group();
      this.aGroup = new THREE.Group();
      this.map = new THREE.Object3D();

      this.fragmentShader2 = `#ifdef GL_ES
        precision mediump float;
        #endif
        varying float opacity;
     uniform vec3 color;
     void main(){
        if(opacity <=0.2){
            discard;
        }
        gl_FragColor = vec4(color,1.0);
      }`;
      this.vertexShader2 = `	varying vec2 vUv;
			attribute float percent;
			uniform float u_time;
			uniform float number;
			uniform float speed;
			uniform float length;
			varying float opacity;
			uniform float size;
			void main()
			{
			vUv = uv;
			vec4 mvPosition = modelViewMatrix * vec4( position, 1.0 );
			float l = clamp(1.0-length,0.0,1.0);
			gl_PointSize = clamp(fract(percent*number + l - u_time*number*speed)-l ,0.0,1.) * size * (1./length);
			opacity = gl_PointSize/size;
			gl_Position = projectionMatrix * mvPosition;
			}`;

      this.initRenderer();
      this.initCamera();
      this.initScene();
      this.initLight();
      // //初始化地球
      this.initEarth();
      // //卫星特效
      // this.initSatellite();
      // //地球光晕
      // this.initEarthSprite();
      // //初始化动态星空背景
      this.initPoints();
      // //外圈中国描边高亮
      this.initGeoJson();
      this.initControls();

      // this.initEarthRayLine();
      this.initTween();
      this.initDotAndFly();
      this.initEarthCloud();

      // // initGui();
      this.animate();
      window.addEventListener("resize", this.onWindowResize(), false);
    },

    // 3D球面取点
    getPots(radius, a, b) {
      let x = radius * Math.sin(a) * Math.cos(b);
      let y = radius * Math.sin(a) * Math.sin(b);
      let z = radius * Math.cos(a);
      return { x, y, z };
    },

    // three 经纬度转换
    lglt2xyz(lng, lat) {
      const theta = (90 + lng) * (Math.PI / 180);
      const phi = (90 - lat) * (Math.PI / 180);
      return new THREE.Vector3().setFromSpherical(
        new THREE.Spherical(this.radius, phi, theta)
      );
    },

    // 经纬度转换球面坐标
    /*
     * @param {地球半径} R
     * @param {经度(角度值)} longitude
     * @param {维度(角度值)} latitude
     */
    lon2xyz(R, longitude, latitude) {
      let lon = (longitude * Math.PI) / 180; //转弧度值
      let lat = (latitude * Math.PI) / 180; //转弧度值
      lon = -lon; // three.js坐标系z坐标轴对应经度-90度，而不是90度
      // 经纬度坐标转球面坐标计算公式
      let x = R * Math.cos(lat) * Math.cos(lon);
      let y = R * Math.sin(lat);
      let z = R * Math.cos(lat) * Math.sin(lon);
      // 返回球面坐标
      return {
        x: x,
        y: y,
        z: z,
      };
    },

    // 经纬度转地球坐标
    createPosition(lnglat) {
      const spherical = new THREE.Spherical();
      spherical.radius = this.radius;
      const lng = lnglat[0];
      const lat = lnglat[1];
      const theta = (lng + 90) * (Math.PI / 180);
      const phi = (90 - lat) * (Math.PI / 180);
      spherical.phi = phi; //方位角
      spherical.theta = theta; //倾斜角
      const position = new THREE.Vector3();
      position.setFromSpherical(spherical);
      return position;
    },

    // 计算 v1 v2 的终点
    getVCenter(v1, v2) {
      const v = v1.add(v2);
      return v.divideScalar(2);
    },

    // 计算V1，V2向量固定长度的点
    getLenVcetor(v1, v2, len) {
      const v1v2Len = v1.distanceTo(v2);
      return v1.lerp(v2, len / v1v2Len);
    },

    // 随机设置点
    setRandomDot(group) {
      let icon1 = require("./assets/红1.png");
      let icon2 = require("./assets/红2.png");
      let texture = new THREE.TextureLoader().load(icon1);
      let texture2 = new THREE.TextureLoader().load(icon2);
      this.posArr.map((e) => {
        let dotMesh = this.createPointMesh(e, texture);
        let waveMesh = this.createWaveMesh(e, texture2);
        group.add(dotMesh);
        group.add(waveMesh);
        this.WaveMeshArr.push(waveMesh);
      });
    },

    // 标注
    createPointMesh(pos, texture) {
      let material = new THREE.MeshBasicMaterial({
        map: texture,
        transparent: true, //使用背景透明的png贴图，注意开启透明计算
        depthWrite: false, //禁止写入深度缓冲区数据
      });

      let mesh = new THREE.Mesh(this.planGeometry, material);
      let size = this.radius * 0.04; // 矩形平面Mesh的尺寸
      mesh.scale.set(size, size, size); // 设置Mesh大小
      // 设置mesh大小
      mesh.position.set(pos.x, pos.y, pos.z);
      // mesh在球面的法线方向
      let coordVec3 = new THREE.Vector3(pos.x, pos.y, pos.z).normalize();
      // mesh在莫仍的xoy平面上,法线方向沿着z轴
      let meshNormal = new THREE.Vector3(0, 0, 1);
      // 四元数属性 .quaterion表示mesh的角度状态
      // .setFromUnitVectors() 计算两个向量之间构成的四元数组
      mesh.quaternion.setFromUnitVectors(meshNormal, coordVec3);
      return mesh;
    },

    // 标注的光圈
    createWaveMesh(pos, texture) {
      let material = new THREE.MeshBasicMaterial({
        // color: 0x22ffcc,
        map: texture,
        transparent: true, //使用背景透明的png贴图，注意开启透明计算
        opacity: 1.0,
        // side: THREE.DoubleSide, //双面可见
        depthWrite: false, //禁止写入深度缓冲区数据
      });
      let mesh = new THREE.Mesh(this.planGeometry, material);
      let size = this.radius * 0.055; //矩形平面Mesh的尺寸
      mesh.size = size; //自顶一个属性，表示mesh静态大小
      mesh.scale.set(size, size, size); //设置mesh大小
      mesh._s = Math.random() * 1.0 + 1.0; //自定义属性._s表示mesh在原始大小基础上放大倍数  光圈在原来mesh.size基础上1~2倍之间变化
      mesh.position.set(pos.x, pos.y, pos.z);
      // mesh姿态设置
      // mesh在球面上的法线方向(球心和球面坐标构成的方向向量)
      let coordVec3 = new THREE.Vector3(pos.x, pos.y, pos.z).normalize();
      // mesh默认在XOY平面上，法线方向沿着z轴new THREE.Vector3(0, 0, 1)
      let meshNormal = new THREE.Vector3(0, 0, 1);
      // 四元数属性.quaternion表示mesh的角度状态
      //.setFromUnitVectors();计算两个向量之间构成的四元数值
      mesh.quaternion.setFromUnitVectors(meshNormal, coordVec3);
      return mesh;
    },

    // 添加飞线
    addLines(v0, v3) {
      // 夹角
      let angle = (v0.angleTo(v3) * 1.8) / Math.PI / 0.1; // 0 ~ Math.PI
      let aLen = angle * 0.4,
        hLen = angle * angle * 12;
      let p0 = new THREE.Vector3(0, 0, 0);
      // 法线向量
      let rayLine = new THREE.Ray(p0, this.getVCenter(v0.clone(), v3.clone()));

      // 顶点坐标
      let vtop = rayLine.at(
        hLen / rayLine.at(1, new THREE.Vector3(0, 0, 0)).distanceTo(p0),
        new THREE.Vector3(0, 0, 0)
      );
      // 控制点坐标
      let v1 = this.getLenVcetor(v0.clone(), vtop, aLen);
      let v2 = this.getLenVcetor(v3.clone(), vtop, aLen);
      // 绘制三维三次贝赛尔曲线
      let curve = new THREE.CubicBezierCurve3(v0, v1, v2, v3);
      let geometry = new LineGeometry();
      let points = curve.getSpacedPoints(50);
      let positions = [];
      let colors = [];
      let color = new THREE.Color();

      /**
       * HSL中使用渐变
       * h — hue value between 0.0 and 1.0
       * s — 饱和度 between 0.0 and 1.0
       * l — 亮度 between 0.0 and 1.0
       */
      for (let j = 0; j < points.length; j++) {
        color.setHSL(0 + j * 0.001, 1, 0.655);
        colors.push(color.r, color.g, color.b);
        positions.push(points[j].x, points[j].y, points[j].z);
      }
      geometry.setPositions(positions);
      geometry.setColors(colors);

      let matLine = new LineMaterial({
        linewidth: 0.001,
        vertexColors: true,
        dashed: false,
      });
      return {
        curve: curve,
        lineMesh: new Line2(geometry, matLine),
      };
    },

    // 初始化渲染场景
    initRenderer() {
      this.renderer = new THREE.WebGLRenderer({
        antialias: true,
        alpha: true,
      });
      this.renderer.setPixelRatio(window.devicePixelRatio);
      this.renderer.setSize(this.width, this.height);
      const containerDom = document.querySelector("#container");
      containerDom.appendChild(this.renderer.domElement);
    },

    // 初始化相机
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(
        45,
        this.width / this.height,
        1,
        10000
      );
      this.camera.position.set(5, -20, 200);
      this.camera.lookAt(0, 3, 0);
      window.camera = this.camera;
    },

    // 初始化场景
    initScene() {
      this.scene = new THREE.Scene();
      this.scene.background = new THREE.Color(0x020924);
      this.scene.fog = new THREE.Fog(0x020924, 200, 1000);
      window.scene = this.scene;
    },

    // 初始化用户交互
    initControls() {
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      // 使动画循环使用时阻尼或自转 意思是否有惯性
      this.controls.enableDamping = true;
      //是否自动旋转
      this.controls.autoRotate = false;
      this.controls.autoRotateSpeed = 2;
      //是否开启右键拖拽
      this.controls.enablePan = true;
    },

    // 动画过渡
    initTween() {
      let tweena = this.cameraCon(3000);
      tweena.start();
    },

    cameraCon(time) {
      let tween1 = new TWEEN.Tween(this.camaPositions[0])
        .to(this.camaPositions[1], time)
        .easing(TWEEN.Easing.Quadratic.InOut);

      let update = () => {
        this.camera.position.set(
          this.camaPositions[0].x,
          this.camaPositions[0].y,
          this.camaPositions[0].z
        );
      };
      tween1.onUpdate(update);
      tween1.onComplete(function () {
        this.initFlag = true;
      });
      return tween1;
    },

    cameraCon1(time) {
      let tween2 = new TWEEN.Tween(this.camaPositions1[0])
        .to(this.camaPositions1[1], time)
        .onUpdate(() => {
          this.camera.position.set(
            this.camaPositions1[0].x,
            this.camaPositions1[0].y,
            this.camaPositions1[0].z
          );
        })
        .start();
      return tween2;
    },

    cameraCon2(time) {
      let tween3 = new TWEEN.Tween(this.camaPositions2[0])
        .to(this.camaPositions2[1], time)
        .easing(TWEEN.Easing.Quadratic.InOut);

      let update = () => {
        this.camera.position.set(
          this.camaPositions2[0].x,
          this.camaPositions2[0].y,
          this.camaPositions2[0].z
        );
      };
      tween3.onUpdate(update);
      tween3.onComplete(function () {
        this.initFlag = true;
      });
      return tween3;
    },
    // 初始化光
    initLight() {
      const ambientLight = new THREE.AmbientLight(0xcccccc, 1.1);
      this.scene.add(ambientLight);
      let directionalLight = new THREE.DirectionalLight(0xffffff, 0.2);
      directionalLight.position.set(1, 0.1, 0).normalize();
      let directionalLight2 = new THREE.DirectionalLight(0xff2ffff, 0.2);
      directionalLight2.position.set(1, 0.1, 0.1).normalize();
      this.scene.add(directionalLight);
      this.scene.add(directionalLight2);
      let hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444, 0.2);
      hemiLight.position.set(0, 1, 0);
      this.scene.add(hemiLight);
      let directionalLight3 = new THREE.DirectionalLight(0xffffff);
      directionalLight3.castShadow = true;
      directionalLight3.shadow.camera.top = 18;
      directionalLight3.shadow.camera.bottom = -10;
      directionalLight3.shadow.camera.left = -52;
      directionalLight3.shadow.camera.right = 12;
      this.scene.add(directionalLight3);
    },

    // 初始化星空
    initPoints() {
      let starpng = require("./assets/gradient.png");
      let texture = new THREE.TextureLoader().load(starpng);
      let positions = [];
      let colors = [];
      let geometry = new THREE.BufferGeometry();
      for (let i = 0; i < 10000; i++) {
        let vertex = new THREE.Vector3();
        vertex.x = Math.random() * 2 - 1;
        vertex.y = Math.random() * 2 - 1;
        vertex.z = Math.random() * 2 - 1;
        positions.push(vertex.x, vertex.y, vertex.z);
        let color = new THREE.Color();
        color.setHSL(
          Math.random() * 0.2 + 0.5,
          0.55,
          Math.random() * 0.25 + 0.55
        );
        colors.push(color.r, color.g, color.b);
      }
      geometry.setAttribute(
        "position",
        new THREE.Float32BufferAttribute(positions, 3)
      );
      geometry.setAttribute(
        "color",
        new THREE.Float32BufferAttribute(colors, 3)
      );
      let starsMaterial = new THREE.PointsMaterial({
        map: texture,
        size: 1,
        transparent: true,
        opacity: 1,
        vertexColors: true, //true：且该几何体的colors属性有值，则该粒子会舍弃第一个属性--color，而应用该几何体的colors属性的颜色
        blending: THREE.AdditiveBlending,
        sizeAttenuation: true,
      });
      this.stars = new THREE.Points(geometry, starsMaterial);
      this.stars.scale.set(300, 300, 300);
      this.scene.add(this.stars);
    },

    // 地球初始化
    initEarth() {
      let earth = require("./assets/Earth_Diffuse_6K.jpg");
      let texture = this.globeTextureLoader.load(earth);
      let globeGgeometry = new THREE.SphereGeometry(this.radius, 100, 100);
      let globeMaterial = new THREE.MeshStandardMaterial({
        map: texture,
        side: THREE.DoubleSide,
      });
      let globeMesh = new THREE.Mesh(globeGgeometry, globeMaterial);
      this.group.rotation.set(0.5, 2.9, 0.1);
      this.group.add(globeMesh);

      this.scene.add(this.group);
    },

    // 地球光晕效果
    // initEarthSprite() {
    //   let EarthSprite = require("./assets/95a47f200a82b50c2a3e6bf31e01e9d.png");
    //   let texture = this.globeTextureLoader.load(EarthSprite);
    //   let spriteMaterial = new THREE.SpriteMaterial({
    //     map: texture,
    //     transparent: true,
    //     opacity: 1,
    //     // depthWrite: false,
    //   });
    //   let sprite = new THREE.Sprite(spriteMaterial);
    //   sprite.scale.set(0.5, 2.9, 0.1);
    //   this.group.add(sprite);
    // },

    initEarthCloud() {
      let EarthSprite = require("./assets/earth_cloud.png");
      let texture = this.globeTextureLoader.load(EarthSprite);
      let globeGgeometry = new THREE.SphereGeometry(5.1, 100, 100);
      let spriteMaterial = new THREE.MeshStandardMaterial({
        map: texture,
        transparent: true,
        opacity: 0.5,
        side: THREE.DoubleSide,
        depthWrite: false,
      });
      let globeMesh = new THREE.Mesh(globeGgeometry, spriteMaterial);
      this.group.rotation.set(0.5, 2.9, 0.1);
      this.group.add(globeMesh);
      this.scene.add(this.group);
    },

    // initEarthRayLine() {
    //   let EarthSprite = require("./assets/c0a7813aff8006171dd5c2bcec4bdd3.png");
    //   let texture = this.globeTextureLoader.load(EarthSprite);
    //   let globeGgeometry = new THREE.SphereGeometry(5.3, 100, 100);
    //   let spriteMaterial = new THREE.MeshStandardMaterial({
    //     map: texture,
    //     transparent: true,
    //     opacity: 0.5,
    //     side: THREE.DoubleSide,
    //     depthWrite: false,
    //   });
    //   let globeMesh = new THREE.Mesh(globeGgeometry, spriteMaterial);
    //   this.group.rotation.set(0.5, 2.9, 0.1);
    //   this.group.add(globeMesh);
    //   this.scene.add(this.group);
    // },

    // 光柱特效
    createLightPillar(pos) {
      let height = this.radius * 0.1; // 光柱高度，和地球半径相关，这样调节地球半径，光柱尺寸跟着变化
      let geometry = new THREE.PlaneBufferGeometry(this.radius * 0.05, height); // 默认在XOY平面上
      geometry.rotateX(Math.PI / 2); //光柱高度方向旋转到z轴上
      geometry.translate(0, 0, height / 2); //平移使光柱底部与XOY平面重合
      let textureLoader = new THREE.TextureLoader(); // TextureLoader创建一个纹理加载器对象
      let material = new THREE.MeshBasicMaterial({
        map: textureLoader.load("./assets/光柱.png"),
        color: 0x44ffaa, //光柱颜色，光柱map贴图是白色，可以通过color调节颜色
        transparent: true, //使用背景透明的png贴图，注意开启透明计算
        side: THREE.DoubleSide, //双面可见
        depthWrite: false, //是否对深度缓冲区有任何的影响
      });
      let mesh = new THREE.Mesh(geometry, material);
      let group = new THREE.Group();
      // 两个光柱交叉叠加
      group.add(mesh, mesh.clone().rotateZ(Math.PI / 2)); //几何体绕x轴旋转了，所以mesh旋转轴变为z
      group.position.set(pos.x, pos.y, pos.z); //设置mesh位置
      let coordVec3 = new THREE.Vector3(pos.x, pos.y, pos.z).normalize();
      let meshNormal = new THREE.Vector3(0, 0, 1);
      // 四元数属性.quaternion表示mesh的角度状态
      //.setFromUnitVectors();计算两个向量之间构成的四元数值
      group.quaternion.setFromUnitVectors(meshNormal, coordVec3);
      return group;
    },

    // 光柱底部矩形特效
    createLightWaveMesh(pos, texture) {
      let geometry = new THREE.PlaneBufferGeometry(1, 1); // 默认在XOY平面上
      let material = new THREE.MeshBasicMaterial({
        color: 0x22ffcc,
        map: texture,
        transparent: true, // 使用背景透明的png贴图，注意开启透明计算
        // side: THREE.DoubleSide, // 双面可见
        depthWrite: false, // 禁止写入深度缓冲区数据
      });

      let mesh = new THREE.Mesh(geometry, material);
      let size = this.radius * 0.05; // 矩形平面Mesh的尺寸
      mesh.scale.set(size, size, size);
      return mesh;
    },

    // 光柱效果
    initLightPillar() {
      let texture = new THREE.TextureLoader().load("./assets/标注.png");

      let datas = [
        {
          lng: 86.39895905468748,
          lat: 45.15923349468924, //合肥
        },
        {
          lng: 106.54041,
          lat: 29.40268, //重庆
        },
      ];
      datas.forEach((e) => {
        let pos = this.lglt2xyz(e.lng, e.lat);
        let LightPillar = this.createLightPillar(pos);
        this.groupDots.add(LightPillar);
        let waveMesh = this.createLightWaveMesh(pos, texture);
        LightPillar.add(waveMesh);
      });
    },

    // 初始化点和曲线
    initDotAndFly() {
      // 创建标注点
      this.setRandomDot(this.groupDots);
      // 随机点加载group上面
      this.group.add(this.groupDots);
      // 曲线
      this.groupDots.children.forEach((e) => {
        if (this.groupDots.children[0].position.x == e.position.x) {
          return true;
        }
        let line = this.addLines(
          e.position,
          this.groupDots.children[0].position
        );
        this.groupLines.add(line.lineMesh);
        this.animateDots.push(line.curve.getPoints(100));
      });
      this.group.add(this.groupLines);
      // 添加动画
      for (let i = 0; i < this.animateDots.length; i++) {
        const aGeo = new THREE.SphereGeometry(0.08, 0.08, 0.08);
        const aMater = new THREE.MeshPhongMaterial({ color: "orange" });
        const aMesh = new THREE.Mesh(aGeo, aMater);
        this.aGroup.add(aMesh);
      }
      this.vIndex = 0;
      this.group.add(this.aGroup);
      this.animateLine();
    },
    // 动画
    animateLine() {
      this.aGroup.children.forEach((elem, index) => {
        const v = this.animateDots[index][this.vIndex];
        elem.position.set(v.x, v.y, v.z);
      });
      this.vIndex++;
      if (this.vIndex > 100) {
        this.vIndex = 0;
      }
      setTimeout(this.animateLine, 20);
    },

    // 中国描边高亮
    initGeoJson() {
      let china = require("./models/china.json");
      let chinaOutLine = require("./models/china-outline.json");

      const loader = new THREE.FileLoader();
      loader.load("./models/china.json", () => {
        // const jsonData = JSON.parse(china);
        this.initMap(china);
      });
      loader.load("./models/china-outline.json", () => {
        this.outLineMap(chinaOutLine);
      });
    },

    // 外围光
    outLineMap(json) {
      json.features.forEach((e) => {
        // 新建一个省份容器:用来存放省份对应的模型和轮廓线
        const province = new THREE.Object3D();
        const coordinates = e.geometry;
        coordinates.coordinates.forEach((multiPolygon) => {
          multiPolygon.forEach((polygon) => {
            // 这里的坐标要做2次使用：1次用来构建模型，1次用来构建轮廓线
            if (polygon.length > 200) {
              let v3ps = [];
              for (let i = 0; i < polygon.length; i++) {
                let pos = this.lglt2xyz(polygon[i][0], polygon[i][1]);
                v3ps.push(pos);
              }
              let curve = new THREE.CatmullRomCurve3(v3ps, false /*是否闭合*/);
              let color = new THREE.Vector3(
                0.5999758518718452,
                0.7798940272761521,
                0.6181903838257632
              );
              let flyLine = this.initFlyLine(
                curve,
                {
                  speed: 0.4,
                  // color: randomVec3Color(),
                  color: color,
                  number: 3, //同时跑动的流光数量
                  length: 0.2, //流光线条长度
                  size: 3, //粗细
                },
                5000
              );
              province.add(flyLine);
            }
          });
        });
        this.map.add(province);
      });
      this.group.add(this.map);
    },

    // 地图
    initMap(chinaJson) {
      // 遍历省份构建模型
      chinaJson.features.forEach((e) => {
        // 新建一个省份容器：用来存放省份对应的模型和轮廓线
        const province = new THREE.Object3D();
        const coordinates = e.geometry;
        coordinates.coordinates.forEach((multiPolygon) => {
          multiPolygon.forEach((polygon) => {
            const lineMaterial = new THREE.LineBasicMaterial({
              color: 0xf19553,
            });
            const positions = [];
            const linGeometry = new THREE.BufferGeometry();
            for (let i = 0; i < polygon.length; i++) {
              var pos = this.lglt2xyz(polygon[i][0], polygon[i][1]);
              positions.push(pos.x, pos.y, pos.z);
            }
            linGeometry.setAttribute(
              "position",
              new THREE.Float32BufferAttribute(positions, 3)
            );
            const line = new THREE.Line(linGeometry, lineMaterial);
            province.add(line);
          });
        });
        this.map.add(province);
      });
      this.group.add(this.map);
    },

    /*
     * @param curve {THREE.Curve} 路径,
     * @param matSetting {Object} 材质配置项
     * @param pointsNumber {Number} 点的个数 越多越细致
     */

    initFlyLine(curve, matSetting, pointsNumber) {
      let points = curve.getPoints(pointsNumber);
      let geometry = new THREE.BufferGeometry().setFromPoints(points);
      const length = points.length;
      let percents = new Float32Array(length);
      for (let i = 0; i < points.length; i += 1) {
        percents[i] = i / length;
      }
      geometry.addAttribute("percent", new THREE.BufferAttribute(percents, 1));
      const lineMaterial = this.initLineMaterial(matSetting);
      let flyLine = new THREE.Points(geometry, lineMaterial);
      return flyLine;
    },

    initLineMaterial(setting) {
      const number = setting ? Number(setting.number) || 1.0 : 1.0;
      const speed = setting ? Number(setting.speed) || 1.0 : 1.0;
      const length = setting ? Number(setting.length) || 0.5 : 0.5;
      const size = setting ? Number(setting.size) || 3.0 : 3.0;
      const color = setting
        ? setting.color || new THREE.Vector3(0, 1, 1)
        : new THREE.Vector3(0, 1, 1);
      const singleUniforms = {
        u_time: this.uniforms2.u_time,
        number: { type: "f", value: number },
        speed: { type: "f", value: speed },
        length: { type: "f", value: length },
        size: { type: "f", value: size },
        color: { type: "v3", value: color },
      };
      const lineMaterial = new THREE.ShaderMaterial({
        uniforms: singleUniforms,
        vertexShader: this.vertexShader2.textContent,
        fragmentShader: this.fragmentShader2.textContent,
        transparent: true,
        //blending:THREE.AdditiveBlending,
      });
      return lineMaterial;
    },

    // 窗口变动
    onWindowResize() {
      // this.camera.aspect = innerWidth / innerHeight;
      this.camera.aspect = innerWidth / innerHeight;
      this.camera.updateProjectionMatrix();
      this.renders();
      this.renderer.setSize(innerWidth, innerHeight);
    },

    // 渲染
    renders() {
      this.renderer.clear();
      this.renderer.render(scene, camera);
    },

    // 更新
    animate() {
      window.requestAnimationFrame(() => {
        if (this.controls) this.controls.update();
        if (TWEEN) TWEEN.update();
        // if (this.initFlag) {
        //   //光环
        //   // this.groupHalo.rotation.z = this.groupHalo.rotation.z + 0.01;
        //   this.group.rotation.y = this.group.rotation.y + 0.001;
        //   // 所有波动光圈都有自己的透明度和大小状态
        //   // 一个波动光圈透明度变化过程是：0~1~0反复循环
        //   if (this.WaveMeshArr.length) {
        //     this.WaveMeshArr.forEach(function (mesh) {
        //       mesh._s += 0.007;
        //       mesh.scale.set(
        //         mesh.size * mesh._s,
        //         mesh.size * mesh._s,
        //         mesh.size * mesh._s
        //       );
        //       if (mesh._s <= 1.5) {
        //         //mesh._s=1，透明度=0 mesh._s=1.5，透明度=1
        //         mesh.material.opacity = (mesh._s - 1) * 2;
        //       } else if (mesh._s > 1.5 && mesh._s <= 2) {
        //         //mesh._s=1.5，透明度=1 mesh._s=2，透明度=0
        //         mesh.material.opacity = 1 - (mesh._s - 1.5) * 2;
        //       } else {
        //         mesh._s = 1.0;
        //       }
        //     });
        //   }
        // }
        if (this.stars) {
          this.stars.rotation.y += 0.0001;
        }
        this.uniforms2.u_time.value += 0.007;
        this.renders();
        this.animate();
      });
    },
  },
};
</script>

<style  scoped>
.btn {
  width: 100px;
  height: 100px;
  background-color: red;
  position: absolute;
  right: 1000px;
  top: 0;
  font-size: 20px;
}

.btn1 {
  width: 100px;
  height: 100px;
  background-color: red;
  position: absolute;
  right: 600px;
  top: 0;
  font-size: 20px;
}
</style>
