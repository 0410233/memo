<template>
  <div class="memo">
    <img class="memo__bg" :src="bg" alt="充满幻想事物的场景"
      :style="{filter: `blur(${step === 1 ? 0 : 21}px)`}"
    >
    <div class="memo__mask"></div>
    <div class="memo__welcome" :class="{'is-hide': step > 0}">欢迎来到「造梦空间」</div>
    <div class="memo__start" :class="{'is-hide': step > 0}" @click="start">开启梦境之旅</div>

    <div v-if="step > 0" class="memo__story story">
      <div class="scroll-view">
        <div v-if="step === 1" class="story__content">
          <template v-if="reply.length">
            <template v-for="text, index in reply">
              <div v-if="/^\d\./.test(text)"
                class="story__choise"
                :style="{animationDelay: index*1000 + 'ms'}"
                @click="onChoose(text)"
              >{{ text }}</div>
              <p v-else class="story__section" :style="{animationDelay: index*1000 + 'ms'}">{{ text }}</p>
            </template>
            <div v-if="!hasChoises" class="story__bottom" :style="{animationDelay: reply.length*1000 + 'ms'}">
              <div class="story__btn" @click="onLookback">回看</div>
              <div class="story__btn" @click="onContinue">继续</div>
            </div>
          </template>
          <svg v-else width="24" height="24" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><ellipse class="spinner_rXNP" cx="12" cy="5" rx="4" ry="4" fill="#ffffff"/></svg>
        </div>
        <div v-else-if="step === 2" class="story__content">
          <p class="story__section" v-for="text in history">{{ text }}</p>
          <div class="story__bottom">
            <div class="story__btn" @click="onContinue">继续</div>
          </div>
        </div>
        <div v-else-if="step === 3" class="story__content">
          <p class="story__section">梦境精灵信号不太好</p>
          <div class="story__bottom">
            <div class="story__btn" @click="onContinue">重试</div>
          </div>
        </div>
      </div>
    </div>

    <div class="change-btn" @click="changeBg">切换背景</div>
  </div>
</template>

<script setup lang="ts">
import type OpenAI from "openai";
// import axios from 'axios';
import { computed, ref } from 'vue';
import img01 from './assets/memo_bg_01.jpg';
import img02 from './assets/memo_bg_02.jpg';
import img03 from './assets/memo_bg_03.jpg';
import img04 from './assets/memo_bg_04.jpg';
import img05 from './assets/memo_bg_05.jpg';
import img06 from './assets/memo_bg_06.jpg';
import img07 from './assets/memo_bg_07.jpg';
import img08 from './assets/memo_bg_08.jpg';
import img09 from './assets/memo_bg_09.jpg';

const imgList = [img01,img02,img03,img04,img05,img06,img07,img08,img09]

function randomRange(n1: number, n2: number) {
  return Math.floor(Math.random() * (n2-n1)) + n1
}

function getRandomBg() {
  return imgList[randomRange(0, 8)]
}

const bg = ref(getRandomBg())
const blur = ref(21)
const step = ref(0)

function changeBg() {
  let img;
  do {
    img = getRandomBg()
  } while (img === bg.value);
  bg.value = img
}

// const openai = new OpenAI({
//   apiKey: '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',
//   dangerouslyAllowBrowser: true,
// });

type AssistantMessageParam = OpenAI.Chat.Completions.ChatCompletionAssistantMessageParam
type SystemMessageParam = OpenAI.Chat.Completions.ChatCompletionSystemMessageParam
type UserMessageParam = OpenAI.Chat.Completions.ChatCompletionUserMessageParam
type ChatMessageParam = SystemMessageParam | UserMessageParam | AssistantMessageParam

const startMessages = [{
  role: "system",
  content: "这里是「造梦空间」，你是造梦空间里一只美丽神秘、偶尔有些调皮的梦境精灵。你的职责是通过文字引导人类完成一次或温馨或惊险或奇趣的梦境之旅。当人类说出“开始吧”，请随机给出三个关于梦境主题的选项。主题选项请尽量简短。人类选择后，梦境之旅开始。请尽量在 7 次以内引导完成。期间有你对梦境的描述，也要有两到三次的互动，即给出如何继续的选项，让人类选择。选项要尽量简短，且每个选项必须以数字+英文点号+空格开头。"
}, {
  role: "user",
  content: "开始吧"
}] as ChatMessageParam[]

const context = ref([] as ChatMessageParam[])

const reply = ref([] as string[])
// const reply = ref([
//   '欢迎来到梦境之旅！现在，请选择你想要的梦境主题：',
//   '',
//   '1. 迷失森林：探索一个古老神秘的森林，寻找失落的宝藏。',
//   '2. 幻想仙境：进入一个充满魔法和奇幻生物的仙境，解开它的谜题。',
//   '3. 时间旅行：穿越时空，回到过去或者未来，亲眼见证历史的变迁。',
//   '',
//   '请选择一个主题开始你的梦境之旅吧！'
// ] as string[])
// const reply = ref([
//   '你选择了返回现实，结束这场梦境之旅。你对时光机器投下最后一瞥，感慨万分地意识到这次梦境之旅给你带来了无尽的想象和探索的乐趣。',
//   '当你缓缓闭上眼睛，再次展开眼睛时，你发现自己已经回到了现实世界。这个梦境之旅可能只是短暂的，但它在你心中留下了深刻的印记。',
//   '感谢你选择「造梦空间」，希望你在这次梦境之旅中有一个愉快、奇妙的体验。如果你还想再次进入梦境，随时来找我吧！',
//   '祝你美梦连连，再见！',
// ] as string[])

const hasChoises = computed(() => {
  return reply.value.findIndex(x => /^\d\./.test(x)) >= 0
})

const history = computed(() => {
  const content = [] as string[]
  context.value.filter(x => x.role === 'assistant').forEach(msg => {
    if (typeof msg.content === 'string') {
      msg.content.split('\n').forEach(text => {
        if (text.trim().length) {
          content.push(text)
        }
      })
    }
  })
  return content
  // return [
  //   '从前',
  //   '从前有座',
  //   '从前有座山，',
  //   '从前有座山，山上',
  //   '从前有座山，山上有座',
  //   '从前有座山，山上有座庙，',
  //   '从前有座山，山上有座庙，庙里',
  //   '从前有座山，山上有座庙，庙里有个',
  //   '从前有座山，山上有座庙，庙里有个老和',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  //   '从前有座山，山上有座庙，庙里有个老和尚',
  // ]
})

// async function chat() {
//   const completion = await openai.chat.completions.create({
//     model: "gpt-3.5-turbo",
//     // messages: [{ role: "user", content: "Say this is a test" }],
//     messages: context.value,
//   });
//   console.log('completion:', completion);
//   return completion.choices[0].message
// }

async function chat() {
  try {
    const url = 'https://service-08uknn1t-1313644629.ca.tencentapigw.com/release/send'
    const response = await fetch(url, {
      method: 'POST',
      cache: "no-cache",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({messages: context.value}),
    })

    const completion: OpenAI.Chat.Completions.ChatCompletion = await response.json()
    console.log('completion:', completion);
    return completion.choices[0].message
  } catch (error) {
    console.warn(error)
    return null
  }
}

function handleReceive(receive: OpenAI.Chat.Completions.ChatCompletionMessage) {
  context.value.push(receive)
  reply.value = receive.content?.split('\n').filter(x => x.length) || []
}

function start() {
  blur.value = 0
  step.value = 1

  context.value = [...startMessages]
  reply.value = []
  chat().then(receive => {
    console.log({receive})
    if (receive) {
      handleReceive(receive)
    } else {
      step.value = 3
    }
  })
}

function onChoose(choise: string) {
  reply.value = []
  const message = {role: 'user', content: choise.split('.')[0]} as UserMessageParam
  context.value.push(message)
  chat().then(receive => {
    if (receive) {
      handleReceive(receive)
    } else {
      step.value = 3
    }
  })
}

function onLookback() {
  step.value = 2
}

function onContinue() {
  step.value = 0
}
</script>

<style scoped>

.memo {
  width: 100%;
  height: 100%;
  padding: 0;
  margin: 0;
  overflow: hidden;
}

.memo {
  position: relative;
}

.memo__bg {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  /* filter: blur(25px); */
  transition: all 2s;
  transform: scale(1.05);

  position: absolute;
  top: 0;
  left: 0;
  z-index: 0;
}

.memo__mask {
  width: 100%;
  height: 100%;
  background-color: rgba(0,0,0,0.38);
  position: absolute;
  top: 0;
  left: 0;
  z-index: 10;
}

.memo__welcome {
  width: 100%;
  font-size: 18px;
  color: rgba(255, 255, 255, 0.8);
  text-align: center;
  white-space: nowrap;
  overflow: hidden;

  animation-name: fadeInCompact;
  animation-duration: 2.4s;
  animation-timing-function: cubic-bezier(0.165, 0.84, 0.44, 1);
  animation-fill-mode: both;

  position: absolute;
  top: 38.2%;
  left: 0;
  transform: translateY(-50%);
  z-index: 20;
}

.memo__start {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 128px;
  height: 128px;
  border-radius: 50%;
  /* border: 3px solid rgba(255,255,255,0.618); */
  font-size: 14px;
  letter-spacing: 0.1em;
  line-height: 1;
  background: rgba(0,0,0,0.382);
  color: #ffffff;
  box-shadow: 0 0 64px rgba(255,255,255,0.618);
  cursor: pointer;

  animation-name: fadeInScaleDown;
  animation-duration: 1.6s;
  animation-delay: 0.4s;
  /* animation-timing-function: cubic-bezier(0.47, 0, 0.745, 0.715); */
  animation-timing-function: cubic-bezier(0.445, 0.05, 0.55, 0.95);
  animation-fill-mode: both;

  position: absolute;
  left: 50%;
  bottom: 38.2%;
  margin: -64px;
  z-index: 20;
}

.memo__welcome.is-hide {
  animation-name: fadeOutLoose;
}

.memo__start.is-hide {
  animation-delay: 0s;
  animation-timing-function: cubic-bezier(0.39, 0.575, 0.565, 1);
  animation-name: fadeOutScaleUp;
}

.memo__story.story {
  display: block;
  width: 800px;
  max-width: 80%;
  height: 80vh;
  padding: 20px 24px;
  border-radius: 20px;
  
  background-color: rgba(0,0,0,0.38);
  
  animation-name: fadeIn;
  animation-duration: 1.2s;
  animation-delay: 0.8s;
  animation-timing-function: cubic-bezier(0.445, 0.05, 0.55, 0.95);
  animation-fill-mode: both;

  position: absolute;
  top: 49%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 200;
}

.scroll-view {
  display: block;
  width: 100%;
  height: 100%;
  overflow-y: auto;
  
  scrollbar-width: thin;
  scrollbar-color: #c7c9cc
}

.scroll-view::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}

.scroll-view::-webkit-scrollbar-track {
  background: transparent;
  border-radius: 4px;
}

.scroll-view::-webkit-scrollbar-thumb {
  background-color: #c7c9cc;
  border-radius: 4px;
  border: 2px solid #c7c9cc;
}

.story__content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  min-height: 100%;
}

.story__section {
  font-size: 14px;
  line-height: 22px;
  margin: 6px 0;
  letter-spacing: 1.5px;
  color: rgba(255, 255, 255, 0.95);
  text-align: center;
}

.story__choise {
  display: flex;
  min-width: 144px;
  max-width: 100%;
  padding: 8px 20px;
  border: 1px solid rgba(255, 255, 255, 0.95);
  border-radius: 99px;
  color: rgba(255, 255, 255, 0.95);
  cursor: pointer;
  transition: all 300ms;
  /* white-space: nowrap; */
  /* overflow: hidden; */
  /* text-overflow: ellipsis; */
  margin: 12px 0;
}

.story__choise + .story__choise {
  margin-top: 6px;
}

.story__choise:hover {
  background-color: rgba(255,255,255,0.1);
}

.story__section, .story__choise, .story__bottom {
  animation-name: fadeIn;
  animation-duration: 1000ms;
  animation-timing-function: linear;
  animation-fill-mode: both;
}

.story__bottom {
  flex: none;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 20px;
}

.story__btn {
  width: 160px;
  padding: 8px 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex: none;
  cursor: pointer;
  margin: 0 12px;
  border: 1px solid rgba(255, 255, 255, 0.95);
  color: rgba(255, 255, 255, 0.95);
  border-radius: 99px;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes fadeInCompact {
  from {
    opacity: 0;
    letter-spacing: 5em;
  }
  to {
    opacity: 1;
    letter-spacing: 0.2em;
  }
}

@keyframes fadeOutLoose {
  from {
    opacity: 1;
    letter-spacing: 0.15em;
  }
  to {
    opacity: 0;
    letter-spacing: 5em;
  }
}

@keyframes fadeInScaleDown {
  from {
    opacity: 0;
    transform: scale(1.618);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes fadeOutScaleUp {
  from {
    opacity: 1;
    transform: scale(1);
  }
  to {
    opacity: 0;
    transform: scale(1.618);
  }
}

.change-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  /* padding:  6px 12px; */
  font-size: 14px;
  line-height: 1.15;
  width: 90px;
  height: 30px;
  border-radius: 4px;
  background-color: rgb(12, 168, 116);
  color: #ffffff;
  cursor: pointer;

  position: absolute;
  top: 20px;
  right: 20px;
  z-index: 1001;
}

/* .loading {
  position: absolute;
  top: 50%;
  left: 50%;
  margin: -12px;
} */

.spinner_rXNP {
  animation: spinner_YeBj .8s infinite;
}

@keyframes spinner_YeBj {
  0% {
    animation-timing-function: cubic-bezier(0.33,0,.66,.33);
    cy: 5px;
  }
  46.875% {
    cy: 20px;
    rx: 4px;
    ry: 4px;
  }
  50% {
    animation-timing-function: cubic-bezier(0.33,.66,.66,1);
    cy: 20.5px;
    rx: 4.8px; 
    ry: 3px;
  }
  53.125% {
    rx: 4px;
    ry: 4px;
  }
  100% {
    cy: 5px;
  }
}
</style>
