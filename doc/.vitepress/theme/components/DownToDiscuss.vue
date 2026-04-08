<script setup>
import { onMounted, onBeforeUnmount, ref } from "vue";

const props = defineProps({
    // 目标容器 ID，通常是 giscus
    targetId: {
        type: String,
        default: "giscus"
    }
});

const showDiscuss = ref(false);

/**
 * 核心逻辑：双段滚动
 * 1. 如果当前不在评论区，滚到评论区顶部（触发加载）
 * 2. 如果已在评论区，滚到页面最底部（直达输入框）
 */
const scrollToComments = () => {
    const el = document.getElementById(props.targetId);
    const totalHeight = document.documentElement.scrollHeight;

    if (!el) {
        window.scrollTo({ top: totalHeight, behavior: "smooth" });
        return;
    }

    const scrollTop = window.scrollY;
    const targetRect = el.getBoundingClientRect();
    const targetTop = targetRect.top + scrollTop - 20;

    // 判断逻辑：当前是否已经接近评论区顶部
    const isAlreadyAtComments = Math.abs(scrollTop - targetTop) < 100;

    // 决定滚动终点
    const finalTarget = isAlreadyAtComments ? totalHeight : targetTop;

    window.scrollTo({
        top: finalTarget,
        behavior: "smooth"
    });

    /**
     * 补丁：针对 Giscus 异步加载高度变化的修正
     */
    const checkScrollEnd = () => {
        const currentPos = window.scrollY + window.innerHeight;
        const latestHeight = document.documentElement.scrollHeight;
        // 如果没到底，再补一次
        if (latestHeight - currentPos > 20) {
            window.scrollTo({ top: latestHeight, behavior: "smooth" });
        }
    };

    setTimeout(checkScrollEnd, 700);
};

const handleScroll = () => {
    const el = document.getElementById(props.targetId);
    if (!el) {
        showDiscuss.value = false;
        return;
    }

    const { scrollY, innerHeight } = window;
    const { scrollHeight } = document.documentElement;

    // 接近底部 350px 时隐藏，避免挡住输入框
    const atBottom = (scrollY + innerHeight) > (scrollHeight - 350);

    showDiscuss.value = scrollY > 100 && !atBottom;
};

onMounted(() => {
    window.addEventListener("scroll", handleScroll);
    handleScroll();
});

onBeforeUnmount(() => {
    window.removeEventListener("scroll", handleScroll);
});
</script>

<template>
    <Transition name="fade">
        <div class="discuss-btn-container" v-show="showDiscuss">
            <div
                class="discuss-main"
                title="直达底部评论"
                @click="scrollToComments"
            >
                <svg class="icon" viewBox="0 0 1024 1024">
                    <path d="M512 64C264.6 64 64 221.3 64 416c0 108.3 61.2 205.1 157.1 270.8L160 896l191.1-95.5c50.3 14.7 104.1 22.8 160.9 22.8 247.4 0 448-157.3 448-352S759.4 64 512 64z" fill="#FFF"/>
                </svg>
            </div>
        </div>
    </Transition>
</template>

<style scoped>
/* 还原到你最初定义的坐标和尺寸 */
.discuss-btn-container {
    position: fixed;
    bottom: 90px;
    right: 20px;
    width: 60px;
    height: 60px;
    z-index: 999;
    user-select: none;
}

.discuss-main {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    cursor: pointer;
    width: 44px;
    height: 44px;
    border-radius: 50%;
    background-color: #10b981;
    padding: 10px;
    box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.discuss-main:hover {
    background-color: #059669;
    /* 悬停微调，保持你原有的 translate 逻辑 */
    transform: translate(-50%, -60%);
    box-shadow: 0 6px 16px rgba(5, 150, 105, 0.4);
}

.discuss-main:active {
    transform: translate(-50%, -50%) scale(0.9);
}

.icon {
    width: 22px;
    height: 22px;
}

.fade-enter-active,
.fade-leave-active {
    transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
}

.fade-enter-from,
.fade-leave-to {
    opacity: 0;
    transform: translateY(20px) scale(0.8);
}
</style>