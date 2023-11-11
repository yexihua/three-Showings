<template>
    <div class="page">
        <!-- 地图 -->
        <div class="map">
            <!-- 人物点 -->
            <div class="people" :style="{
                'transform': `rotate(${props.rotate}deg) translate(0,-11px)` , 
                'left': `${props.position.left}px`,
                'top': `${props.position.top}px`
            }">
            </div>
        </div>
    </div>
</template>

<script setup>
const props = defineProps({
    rotate: {
        type: Number,
        default: 0,
    },
    position: {
        type: Object,
        default: () => ({ left: 0, top: 0 }),
    },
});
</script>

<style scoped>
.page {
    position: fixed;
    background-color: rgba(84, 84, 84, 0.5);
    top: 15px;
    right: 15px;
    width: 140px;
    height: 180px;
    border-radius: 5px;
}

.map {
    width: 100%;
    height: 100%;
    background: url('../../public/map.png') no-repeat;
    background-size: 100% 100%;
}

.people {
    position: absolute;
    width: 30px;
    height: 30px;
    background: url('../../public/people.png') no-repeat;
    background-size: 100% 100%;
    transition: top 1.5s,left 1.5s;
}
</style>