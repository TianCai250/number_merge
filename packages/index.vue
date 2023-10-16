<template>
    <div class="container">
        <div class="header">
            <div class="list" v-for="item in list" :key="item.key">
                <div v-if="isMobile" :class="['list_box', { limit: item.cards.length === 10 }]" ref="listBoxRefs">
                    <div class="list_header"></div>
                    <div class="common_card card" v-for="(card, index) in item.cards" :key="index">{{ card }}</div>
                </div>
                <div v-else :class="['list_box', { limit: item.cards.length === 10 }]" ref="listBoxRefs" @drop="drop">
                    <div class="list_header"></div>
                    <div class="common_card card" v-for="(card, index) in item.cards" :key="index">{{ card }}</div>
                </div>
            </div>
        </div>
        <div class="footer">
            <div class="footer_left">
                <div class="will_box">
                    <div class="common_card will">{{ queue[0] || '' }}</div>
                    <div class="common_card will">{{ queue[1] || '' }}</div>
                </div>
                <div v-if="isMobile" class="common_card current" @touchstart.prevent="touchstart" @touchmove="touchmove" @touchend="touchend">
                    {{ queue[2] || '' }}
                </div>
                <div v-else class="common_card current" draggable="true" @dragstart="touchstart" @dragend="touchend">
                    {{ queue[2] || '' }}
                </div>
            </div>
            <div class="footer_right">SCORE: {{ SCORE }}</div>
        </div>
    </div>
</template>

<script setup>
import { ref, onBeforeMount } from 'vue';

const nums = [2, 4, 8, 16, 32, 64];
const MAX_NUM = 2048;
const MAX_COUNT = 10;

let SCORE = ref(0);
const isMobile = ref(false);
const list = ref([
    {
        key: 1,
        cards: []
    },
    {
        key: 2,
        cards: []
    },
    {
        key: 3,
        cards: []
    },
    {
        key: 4,
        cards: []
    }
]);
const queue = ref([]);
const listBoxRefs = ref(null);

onBeforeMount(() => {
    init();
});

const init = () => {
    isMobile.value = getIsMobile();
    queue.value = getRandoms(3);
    window.addEventListener('resize', () => {
        isMobile.value = getIsMobile();
    });
};

function getIsMobile() {
    let userAgentInfo = navigator.userAgent;
    let Agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod'];
    let getArr = Agents.filter((i) => userAgentInfo.includes(i));
    return getArr.length ? true : false;
}

// 生成count个随机卡片
const getRandoms = (count) => {
    let res = [];
    for (let i = 0; i < count; i++) {
        const random = Math.round(Math.random() * 5);
        res.push(nums[random]);
    }
    return res;
};

let elLeft = 0;
let elTop = 0;
// 开始拖拽
const touchstart = (e) => {
    const target = (e.changedTouches ? e.changedTouches[0] : e).target;
    if (!target) {
        return;
    }
    elLeft = (e.changedTouches ? e.changedTouches[0] : e).clientX - target.offsetLeft;
    elTop = (e.changedTouches ? e.changedTouches[0] : e).clientY - target.offsetTop;
};

// 拖拽中
const touchmove = (e) => {
    const target = (e.changedTouches ? e.changedTouches[0] : e).target;
    if (!target) {
        return;
    }
    target.style.position = 'absolute';
    target.style.left = (e.changedTouches ? e.changedTouches[0] : e).clientX - elLeft + 'px';
    target.style.top = (e.changedTouches ? e.changedTouches[0] : e).clientY - elTop + 'px';
};

// 结束拖拽
const touchend = (e) => {
    const target = (e.changedTouches ? e.changedTouches[0] : e).target;
    if (!target) {
        return;
    }
    // 放置区位置
    const area = [];
    listBoxRefs.value.forEach((item, index) => {
        area.push({
            left_top: {
                x: item.offsetLeft,
                y: item.offsetTop
            },
            right_bottom: {
                x: item.offsetLeft + 50,
                y: item.offsetTop + 60 + 20 * (list.value[index].cards.length - 1 > 0 ? list.value[index].cards.length - 1 : 0)
            }
        });
    });
    // 检测是否位于放置区
    // 左上角
    const x1 = (e.changedTouches ? e.changedTouches[0] : e).clientX - elLeft;
    const y1 = (e.changedTouches ? e.changedTouches[0] : e).clientY - elTop;
    // 右上角
    const x2 = (e.changedTouches ? e.changedTouches[0] : e).clientX - elLeft + 50;
    const y2 = (e.changedTouches ? e.changedTouches[0] : e).clientY - elTop;
    // 左下角
    const x3 = (e.changedTouches ? e.changedTouches[0] : e).clientX - elLeft;
    const y3 = (e.changedTouches ? e.changedTouches[0] : e).clientY - elTop + 60;
    // 右下角
    const x4 = (e.changedTouches ? e.changedTouches[0] : e).clientX - elLeft + 50;
    const y4 = (e.changedTouches ? e.changedTouches[0] : e).clientY - elTop + 60;
    // 还原
    elLeft = 0;
    elTop = 0;
    target.style.position = 'relative';
    target.style.left = 0;
    target.style.top = 0;
    for (let i = 0; i < area.length; i++) {
        let flag1 = x1 > area[i].left_top.x && x1 < area[i].right_bottom.x && y1 > area[i].left_top.y && y1 < area[i].right_bottom.y;
        let flag2 = x2 > area[i].left_top.x && x2 < area[i].right_bottom.x && y2 > area[i].left_top.y && y2 < area[i].right_bottom.y;
        let flag3 = x3 > area[i].left_top.x && x3 < area[i].right_bottom.x && y3 > area[i].left_top.y && y3 < area[i].right_bottom.y;
        let flag4 = x4 > area[i].left_top.x && x4 < area[i].right_bottom.x && y4 > area[i].left_top.y && y4 < area[i].right_bottom.y;
        if (flag1 || flag2 || flag3 || flag4) {
            if (list.value[i].cards.length === MAX_COUNT && list.value[i].cards[list.value[i].cards.length - 1] !== queue.value[queue.value.length - 1]) {
                // 列超过最大数量
            } else if (list.value[i].cards.length === 0 || list.value[i].cards[list.value[i].cards.length - 1] !== queue.value[queue.value.length - 1]) {
                const v = queue.value.pop();
                queue.value.unshift(...getRandoms(1));
                list.value[i].cards.push(v);
            } else {
                const v = queue.value.pop();
                queue.value.unshift(...getRandoms(1));
                merge(i, list.value[i].cards.length - 1, v, true);
            }
            break;
        }
    }
};

// 处理合并(index: 第几列，idx: 和第几张比较,v: 新来的值, firstFlag: 第一次合并)
const merge = (index, idx, v, firstFlag) => {
    const cards = list.value[index].cards;
    if (idx < 0) {
        return;
    }

    if (cards.length > 0 && cards[idx] === v) {
        cards[idx] = v * 2;
        if (!firstFlag) {
            cards.pop();
        }
        SCORE.value++;
        // 达到最大值，清空
        if (cards[cards.length - 1] >= MAX_NUM) {
            list.value[index].cards = [];
            return;
        }
        merge(index, idx - 1, cards[idx], false);
    }
};
</script>

<style lang="scss" scoped>
* {
    padding: 0;
    margin: 0;
}
.container {
    width: 100vw;
    max-width: 800px;
    min-width: 375px;
    height: 100vh;
    margin: 0 auto;
    min-height: 400px;
    font-size: 16px;
    user-select: none;
    overflow: hidden;
    box-sizing: border-box;
    background-color: rgb(247, 245, 245);
}
.header {
    width: 100%;
    height: calc(100% - 100px);
    display: flex;
    .list {
        width: 25%;
        height: 100%;
        overflow: hidden;
        .list_box {
            width: 50px;
            margin: 0 auto;
            margin-top: 10px;
            &.limit {
                border: 1px solid red;
            }
            .list_header {
                width: 100%;
                height: 60px;
                border: 1px dashed gray;
            }
            .card {
                margin-top: -40px;
            }
            .card:nth-child(2) {
                margin-top: -60px;
            }
        }
    }
}
.common_card {
    width: 50px;
    height: 60px;
    border: 1px solid gray;
    background-color: #fff;
    text-align: center;
    cursor: move;
}
.footer {
    width: 100%;
    height: 100px;
    border-top: 1px solid #000;
    display: flex;
    .footer_left {
        width: 50%;
        display: flex;
        justify-content: space-between;
        align-items: center;
        .will_box {
            display: flex;
            justify-content: space-around;
            width: 110px;
            height: 60px;
            .will {
                cursor: default;
            }
        }
    }
    .footer_right {
        width: 50%;
        display: flex;
        justify-content: center;
        align-items: center;
    }
}
</style>
