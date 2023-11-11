<template>
  <div ref="home" class="home">
    <div v-for="i in houseMessage[data.nowId].interactivePoints" :key="i.key" class="point"
      :style="{ transform: `translateX(${i.transformX}px) translateY(${i.transformY}px) scale(60%,60%)`, display: `${i.visable == true ? '' : 'none'}` }">
      <div class="message">
        <div class="message-icon" :style="{ 'background-image': `url(${i.cover})` }"></div>
        <div class="text">
          <div class="label">{{ i.value }}</div>
          <div class="content">{{ i.content }}</div>
        </div>
      </div>
    </div>
    <Map :rotate="data.PeopleRotate" :position="data.position"></Map>
  </div>
  <!-- 按钮组 -->
  <div class="buttonGrounp">
    <div @click="jump(i)" class="scenc-button" v-for="i in showMessage" :key="i.id">
      <span>{{ i.name }}</span>
      <span class="icon"></span>
    </div>
  </div>
</template>


<script setup>
import * as THREE from 'three';//引入three
import gsap from 'gsap'//引入gsap动画库
import Map from './views/map.vue'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { computed, onMounted, reactive, ref } from 'vue';
const houseMessage = reactive([{
  name: '大厅',
  id: 0,
  image: '../public/hall.jpg',
  left: 40,//区域小地图人物位置
  top: 90,
  interactivePoints: [
    {
      key: 'tv',
      value: '电视',
      content: '智能电视',
      cover: '../public/icon/tv.png',
      position: new THREE.Vector3(-6, 2, -15),
      transformX: 0,//点位移
      transformY: 0,
      visable: true,//是否可见
    },
    {
      key: 'art',
      value: '艺术品',
      content: '精美艺术',
      cover: '../public/icon/art.png',
      position: new THREE.Vector3(10.5, 0, -15),
      transformX: 0,
      transformY: 0,
      visable: true,//是否可见
    },
    {
      key: 'plant',
      value: '绿植',
      content: '自由呼吸',
      cover: '../public/icon/plant.png',
      position: new THREE.Vector3(-17, 1, -15),
      transformX: 0,//位移
      transformY: 0,
      visable: true,//是否可见
    },
    {
      key: 'plant',
      value: '沙发',
      content: '现在家居',
      cover: '../public/icon/sofa.png',
      position: new THREE.Vector3(-3, -2, 2),
      transformX: 0,//位移
      transformY: 0,
      visable: true,//是否可见
    },
  ]
}, {
  name: '卧室',
  id: 1,
  image: '../public/bed.jpg',
  left: 90,//区域小地图人物位置
  top: 45,
  interactivePoints: [
    {
      key: 'bed',
      value: '床',
      content: '温暖的床',
      cover: '../public/icon/bed.png',
      position: new THREE.Vector3(0, -0.5, -2),
      transformX: 0,//位移
      transformY: 0,
      visable: false,//是否可见
    },
  ]
}, {
  name: '厨房',
  id: 2,
  image: '../public/kitchen.jpg',
  left: 85,//区域小地图人物位置
  top: 80,
  interactivePoints: [
    {
      key: 'fruit',
      value: '水果',
      content: '美味食物',
      cover: '../public/icon/fruit.png',
      position: new THREE.Vector3(0.1, -0.5, -2),
      transformX: 0,//位移
      transformY: 0,
      visable: false,//是否可见
    },
    {
      key: 'fridge',
      value: '冰箱',
      content: '智能家电',
      cover: '../public/icon/fridge.png',
      position: new THREE.Vector3(7, 1, 1),
      transformX: 0,//位移
      transformY: 0,
      visable: false,//是否可见
    },
  ]
}, {
  name: '厕所',
  id: 3,
  image: '../public/wc.jpg',
  left: 40,//区域小地图人物位置
  top: 30,
  interactivePoints: []
}, {
  name: '走廊',
  id: 4,
  image: '../public/wash.jpg',
  left: 60,//区域小地图人物位置
  top: 35,
  interactivePoints: []
},])
const data = reactive({
  jumping: false,//是否正在场景跳转
  nowId: 0,//当前显示场景的id
  controls: null,
  sizes: {
    width: window.innerWidth,
    height: window.innerHeight,
  },
  position: {//地图小点的xY位移
    left: 40,
    top: 90,
  },
  PeopleRotate: null,//地图小点的旋转角度
});
const home = ref(null)
const raycaster = new THREE.Raycaster();//光线投射实例
const showMessage = computed(() => {
  const list = houseMessage.filter((e) => {
    return e.id != data.nowId
  })
  return list
})
const scene = new THREE.Scene();//创建大厅场景
// 创建摄像机，参数为角度，显示宽高比，显示对象的距离区间
const camera = new THREE.PerspectiveCamera(75, data.sizes.width / data.sizes.height, 0.1, 1000);
const renderer = new THREE.WebGLRenderer();//创建渲染器
renderer.setSize(data.sizes.width, data.sizes.height);//设置渲染大小
renderer.setPixelRatio(window.devicePixelRatio);//设备像素比适配
const geometry = new THREE.SphereGeometry(16, 128, 128)//设置圆半径，长宽分段数
let hallTexture = new THREE.TextureLoader().load('../public/hall.jpg');
const material = new THREE.MeshBasicMaterial({ map: hallTexture, opacity: 1, transparent: true });//设置对象材质
let sqhere = new THREE.Mesh(geometry, material);//设置网格对象
sqhere.rotation.y = Math.PI / 2;//旋转90度
geometry.scale(1, 1, -1)
scene.add(sqhere);//场景添加物品，默认添加位置（0,0,0）
camera.position.z = 0.1;//避免摄像机摄像不到设置点距离

// 右侧房间跳转
const jump = async (e) => {
  if (data.jumping == false) {
    const timeline = gsap.timeline();
    data.jumping = true
    data.nowId = e.id
    data.position.left=houseMessage[data.nowId].left
    data.position.top=houseMessage[data.nowId].top
    let interactivePoints = houseMessage[data.nowId].interactivePoints
    houseMessage[data.nowId].interactivePoints = []
    hallTexture = new THREE.TextureLoader().load(`${e.image}`);
    let newMaterial = new THREE.MeshBasicMaterial({ map: hallTexture, opacity: 0, transparent: true });//设置对象材质
    timeline.to(sqhere.material, { //淡出
      opacity: 0.05,
      duration: 0.7,
      transparent: true,
      onComplete: () => {
        sqhere.material = newMaterial;//设置网格对象
      }
    })
    timeline.to(newMaterial, {
      opacity: 1, duration: 1, transparent: true,
      onComplete: () => {
        data.jumping = false
        houseMessage[data.nowId].interactivePoints = interactivePoints
      }
    }, 0.7)//淡入
    camera.updateProjectionMatrix();//更新相机摄像情况
  }
}
// 页面缩放事件监听
window.addEventListener('resize', () => {
  data.sizes.width = window.innerWidth;
  data.sizes.height = window.innerHeight;
  // 更新渲染
  renderer.setSize(data.sizes.width, data.sizes.height);
  renderer.setPixelRatio(window.devicePixelRatio);
  // 更新相机
  camera.aspect = data.sizes.width / data.sizes.height;
  camera.updateProjectionMatrix();
});
function animate() {
  for (const point of houseMessage[data.nowId].interactivePoints) {
    // 获取2D屏幕位置
    const screenPosition = point.position.clone();
    const pos = screenPosition.project(camera);
    // 通过摄像机和鼠标位置更新射线
    raycaster.setFromCamera(screenPosition, camera);
    // 计算物体和射线的焦点
    const intersects = raycaster.intersectObjects(scene.children, true);
    if (intersects.length === 0) {
      // 未找到相交点，显示
      point.visable = true;
    } else {
      // 获取相交点的距离和点的距离
      const intersectionDistance = intersects[0].distance;
      const pointDistance = point.position.distanceTo(camera.position);
      // 相交点距离比点距离近，隐藏；相交点距离比点距离远，显示
      intersectionDistance < pointDistance
        ? point.visable = false
        : point.visable = true;
    }
    // 物体转动到背面时隐藏，否则显示
    pos.z > 1
      ? point.visable = false
      : point.visable = true;
    point.transformX = screenPosition.x * data.sizes.width * 0.5;
    point.transformY = -screenPosition.y * data.sizes.height * 0.5;
  }
  // 获取小地图人物旋转角度
  if (camera && data.controls) {
    const dirx = camera.position.x - data.controls.target.x
    const dirz = camera.position.z - data.controls.target.z
    data.PeopleRotate = -Math.atan2(dirx, dirz) * 180 / Math.PI;
  }
  renderer.render(scene, camera);//渲染
  requestAnimationFrame(animate);//掉用动画函数
}
onMounted(() => {
  const controls = new OrbitControls(camera, home.value);//设置控制器，鼠标拖拽可使相机围绕目标进行运动
  controls.enableDamping = true//开启旋转阻力
  data.controls = controls;
  home.value.appendChild(renderer.domElement)//添加渲染节点
  animate();
})
</script>



<style scoped>
body {
  margin: 0;
  padding: 0;
}

.home {
  width: 100vw;
  height: 100vh;
}

.buttonGrounp {
  position: fixed;
  top: 50%;
  right: 15px;
  display: flex;
  flex-direction: column;
  transform: translate(0, -50%);
}

.scenc-button {
  margin: 8px 0;
  background-color: rgba(84, 84, 84, 0.5);
  border: 0px solid;
  border-radius: 5px;
  height: 30px;
  width: 80px;
  display: flex;
  justify-content: center;
  align-items: center;
  cursor: pointer;
}

.scenc-button:hover {
  background-color: rgb(84, 84, 84);
}

.icon {
  background-image: url('../public/arrow.png');
  background-size: 100% 100%;
  width: 25px;
  height: 20px;
  transform: rotate(180deg)
}

.point {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(20px, 20px);
  background-image: url('../public/point.png');
  background-repeat: no-repeat;
  background-position: 0, 0;
  background-size: 100%;
  width: 64px;
  height: 64px;
  transform: scale(60%, 60%);
  animation: greenPoint 1s steps(24) both infinite;
}

@keyframes greenPoint {
  0% {
    background-position: 0 0
  }

  100% {
    background-position: 0 -1536PX;
  }
}

.message {
  position: absolute;
  left: 100%;
  top: 0;
  background-color: rgba(255, 255, 255, 0.6);
  width: 180px;
  height: 80px;
  border-radius: 20px;
  display: flex;
  justify-content: flex-start;
  align-items: center;
}

.message-icon {
  width: 80px;
  height: 80px;
  margin-left: 5px;
  background-repeat: no-repeat;
  background-position: 0 0;
  background-size: 100% 100%;
}

.text {
  display: flex;
  flex-direction: column;
  margin-left: 10px;
  justify-content: flex-start;
  align-items: flex-start;
}

.label {
  color: #1d1f24;
  font-weight: 800;
  font-size: 20px;
  text-align: left;
}

.content {
  color: #00aa47;
}
</style>
