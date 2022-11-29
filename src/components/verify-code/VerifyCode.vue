<script lang="ts" setup>
import { onMounted, ref, type Ref } from 'vue'
interface StyleList{
    slideBg?:string,
    checkOK?:string,
    checkFail?:string   
}
interface PropType{
    width?:number,
    styleObj?:StyleList,
    tip?:string,
    verifyImgList?:Array<string>,
}
const emits = defineEmits(['ok','fail'])
const { 
    width,
    styleObj:{slideBg,checkOK,checkFail},
    tip,
    verifyImgList
} = withDefaults(defineProps<PropType>(),{
    width:360,
    styleObj:()=>({
        slideBg:'rgb(75, 137, 237)',
        checkOK:'#6ECCAF',
        checkFail:'#DC3535'
    }),
    tip:'拖动滑块验证',
    verifyImgList:()=>['./hah.png']
})
const slide = ref<HTMLBodyElement | null>(null)
const isMouseDown = ref<boolean>(false)
const isRegisterDrag = ref<boolean>(false)
const slidingWidth = ref<string>('0px')
const imgUrl = ref<string>('')
const targetBlockX = ref<number>(0)
const originalBlockX = ref<number>(0)
const status = ref<number>(0)
onMounted(() => {
    initGlobalProperty()
    initCanvas()
})
const dragSlide = (target:Ref)=> {
    if(target){
        const targetVal = target.value
        const maxMove = width - targetVal.offsetWidth - 2
        const canvasBlock = document.querySelector('.canvas-block') as HTMLCanvasElement
        const blockWdith = canvasBlock.offsetWidth
        let originalX = 0
        targetVal.addEventListener('mousedown',(e:any)=>{
            isMouseDown.value = true
            originalX = e.pageX
            document.documentElement.addEventListener('mousemove',handleMove)
        })
        document.documentElement.addEventListener('mouseup',()=>{
            checkVerifyCode()
            document.documentElement.removeEventListener('mousemove',handleMove)
            setTimeout(() => {
                if(status.value === 1){
                    emits('ok')
                }else if(status.value === 2){
                    emits('fail')
                }
                reFresh()
            }, 1000)
        })
        
        function handleMove(e:any){
            const moveX = e.pageX - originalX
            if(moveX <= maxMove && moveX >= 0){
                slidingWidth.value = moveX + 'px'
                targetVal.style.left = moveX + 'px'
                canvasBlock.style.left = originalBlockX.value + moveX + 'px'
                if(moveX >= width - originalBlockX.value - blockWdith){
                    canvasBlock.style.left = width - blockWdith + 'px'
                }
            }else if(moveX > maxMove){
                slidingWidth.value = maxMove + 'px'
                targetVal.style.left = maxMove + 'px'
                canvasBlock.style.left = width - blockWdith + 'px'
            }else if(moveX < 0){
                slidingWidth.value = '0px'
                targetVal.style.left = '0px'
                canvasBlock.style.left = originalBlockX.value + 'px'
            }
        }
        function checkVerifyCode() {
            const differenceValue = Math.abs(canvasBlock.offsetLeft - targetBlockX.value)
            if(differenceValue <= 3){
                document.documentElement.style.setProperty('--slide-color',checkOK as string)
                status.value = 1
            }else{
                document.documentElement.style.setProperty('--slide-color',checkFail as string)
                status.value = 2
            }
        }
    }
}
const reFresh = ()=> {
    (slide.value as HTMLBodyElement).style.left = '0px';
    (document.querySelector('.canvas-block') as HTMLCanvasElement).style.left = originalBlockX.value + 'px'
    document.documentElement.style.setProperty('--slide-color',slideBg as string)
    isMouseDown.value = false
    slidingWidth.value = '0px'
    status.value = 0
    initCanvas()
}
const initGlobalProperty = ()=> {
    document.documentElement.style.setProperty('--slide-width',width+'px')
    document.documentElement.style.setProperty('--slide-color',slideBg as string)
 }
const getImg = ()=> {
    return new Promise((resolve,reject)=>{
        const originalImg = verifyImgList[Math.floor(Math.random()*verifyImgList.length)]
        imgUrl.value = new URL(originalImg, import.meta.url).href
        if(imgUrl.value){
            resolve('ok')
        }else{
            reject('获取图片失败')
        }
    })
}
const initCanvas = ()=> {
    const canvasBg = document.querySelector('.canvas-bg') as HTMLCanvasElement
    const ctx = canvasBg.getContext('2d')
    const canvasBlock = document.querySelector('.canvas-block') as HTMLCanvasElement
    const ctxBlock = canvasBlock.getContext('2d')
    if(ctx && ctxBlock){
        getImg()
        .then(()=>{
            const img = document.createElement('img')
            img.src = imgUrl.value
            img.crossOrigin = "anonymous"
            img.onload = ()=> {
                resetCanvas(canvasBg)
                resetCanvas(canvasBlock)
                const picX = Math.floor(Math.random() * (width / 2 - 40) + width / 2)
                const picY = Math.floor(Math.random() * (width / 2 - 30))
                targetBlockX.value = picX
                ctx.drawImage(img,0,0,width,width/2)
                draw(ctx,picX,picY)
                ctx.fill()
                draw(ctxBlock,picX,picY)
                ctxBlock.drawImage(img,0,0,width,width/2)
                const imgData = ctxBlock.getImageData(picX-1,picY-1,42,32)
                canvasBlock.width = 42
                canvasBlock.height = 32
                ctxBlock.putImageData(imgData, 0, 0)
                canvasBlock.style.top = picY + 'px'
                const blockLeft = Math.floor(Math.random() * 50)
                canvasBlock.style.left = blockLeft + 'px'
                originalBlockX.value = blockLeft
                if(!isRegisterDrag.value){
                    isRegisterDrag.value = true
                    dragSlide(slide)
                }
            }
        })
        .catch((err:string)=>{
            ctx.font = 'bold 30px serif'
            ctx.fillText(err,0,100)
        })
        function draw(ctx:any,picX:number,picY:number) {
            ctx.beginPath()
            ctx.moveTo(picX,picY)
            ctx.lineTo(picX+30,picY)
            ctx.lineTo(picX+30,picY+12)
            ctx.arc(picX+34, picY+15, 5, 1.25 * Math.PI, 2.8 * Math.PI)
            ctx.lineTo(picX+30,picY+30)
            ctx.lineTo(picX,picY+30)
            ctx.lineTo(picX,picY+18)
            ctx.arc(picX+4, picY+15, 5, 2.8 * Math.PI, 1.25 * Math.PI, true)
            ctx.lineTo(picX,picY)
            ctx.closePath()
            ctx.fillStyle = 'rgba(225, 225, 225, 0.8)'
            ctx.strokeStyle = 'rgba(225, 225, 225, 0.8)'
            ctx.lineWidth = 1;
            ctx.stroke()
            ctx.clip()
        }
        function resetCanvas(canvas:HTMLCanvasElement) {
            canvas.width = width
            canvas.height = width / 2
        }
    }
}
</script>
<template>
    <div :style="{width:width+'px',position:'relative'}">
        <canvas class="canvas-bg" :width="width" :height="width/2"></canvas>
        <canvas class="canvas-block" :width="width" :height="width/2"></canvas>
        <div class="sliding-block">
            <div class="slide" ref="slide">
                <svg v-show="status === 0" version="1.1" width="30" height="15" >
                    <line id="svg_4" y2="7.40525" x2="29.94681" y1="7.29887" x1="-0.05319" stroke-width="1.5" stroke="#fff" fill="none"/>
                    <line id="svg_5" y2="7.51164" x2="29.94681" y1="0.06483" x1="18.03191" stroke-width="1.5" stroke="#fff" fill="none"/>
                    <line id="svg_6" y2="14.6393" x2="18.35106" y1="7.7244" x1="29.62766" stroke-width="1.5" stroke="#fff" fill="none"/>
                </svg>
                <svg v-show="status === 1" version="1.1" width="30" height="30" >
                    <path stroke="#fff" d="m2.68367,13.92409l2.78579,-2.96246l7.63163,8.1117l12.72169,-13.5195l2.78678,2.96037l-15.50846,16.48405" stroke-width="1" fill="none"/>
                </svg>
                <svg v-show="status === 2" version="1.1" width="30" height="30" >
                    <path d="m-57.38475,-32.74536l3.18376,-2.93425l8.72187,8.03444l14.53907,-13.39074l3.18489,2.93217l-17.72396,16.32706" fill-opacity="null" stroke-opacity="null" stroke-width="1" stroke="#fff" fill="none"/>
                    <path d="m25.85306,21.29581l-6.97114,-6.5427l6.96987,-6.5427l-3.58313,-3.36529l-6.97114,6.5427l-6.97114,-6.5427l-3.58313,3.36529l6.96987,6.5427l-6.97114,6.5427l3.58567,3.3641l6.96987,-6.5427l6.96987,6.5427" fill-opacity="null" stroke-opacity="null" stroke-width="1" stroke="#fff" fill="none"/>
                </svg>
            </div>
            <p class="sliding-tip" :style="{opacity:isMouseDown?.5:1}">{{tip}}</p>
            <div class="sliding-move" :style="{width:slidingWidth}"></div>
        </div>
    </div>
</template>
<style scoped>
    *{
        box-sizing: border-box;
        padding: 0;
        margin: 0;
        user-select: none;
    }
    .canvas-block{
        position: absolute;
        z-index: 999;
        top: 0;
        left: 0;
    }
    .code_box{
        background-repeat: no-repeat;
        background-size: 100% 100%;
    }
    .sliding-block{
        position: relative;
        margin-top: 10px;
        height: 40px;
        border-radius: 1px;
        background-color: rgb(249, 246, 246);
        border: 1px solid rgb(219, 218, 218);
        box-shadow: 0 0 2px 0px rgb(230, 230, 230);
    }
    .sliding-move{
        position: absolute;
        height: 100%;
        top: 0;
        left: 0;
        background-color: var(--slide-color);
        opacity: .4;
        animation: sliding-move .5s linear infinite alternate;
    }
    .sliding-tip{
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%,-50%);
        color: rgb(130, 126, 126);
        font-size: 17px;
    }
    .slide{
        position: relative;
        z-index: 998;
        height: 100%;
        width: 50px;
        border-radius: 1px;
        display: flex;
        justify-content: center;
        align-items: center;
        background-color: var(--slide-color);
    }
    .slide:hover{
        cursor: pointer;
        box-shadow: inset 0 0 3px rgb(244, 244, 245);
    }
    .slide::after{
        content: '';
        position: absolute;
        width: 100%;
        height: 100%;
        top: 0;
        left: 0;
        background-color: #fff;
        opacity: .2;
    }
    .slide:hover::after{
        content: none;
    }
    .slide:hover:before{
        content: '';
        position: absolute;
        opacity: .5;
        z-index: 999;
        height: 1px;
        width: 1px;
        left: 70px;
        top: 50%;
        transform: translateY(-50%);
        box-shadow: 0 0px 5px 20px #fff;
        animation: light-move 1.8s linear infinite;
    }
    .slide:active:before{
        content: none;
    }
    @keyframes light-move {
        100%{
            left: calc(var(--slide-width) - 10px);
        }
    }
    @keyframes sliding-move{
        0%{
            box-shadow: inset 0 0 0px rgb(238, 240, 240);
        }
        100%{
            box-shadow: inset 0 0 3px 1px rgb(238, 240, 240);
        }
    }
</style>