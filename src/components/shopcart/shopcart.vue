<template>
    <div class="shopcart">
        <div class="content">
            <div class="content-left">
                <div class="logo-wrapper">
                    <div class="logo" :class="{'highlight':totalCount>0}">
                        <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
                    </div>
                    <div class="num" v-show="totalCount>0">{{totalCount}}</div>
                </div>
                <div class="price" :class="{'highlight':totalCount>0}">￥{{totalPrice}}元</div>
                <div class="desc">
                    另需配送费￥{{deliveryPrice}}元
                </div>
            </div>
            <div class="content-right">
                <div class="pay" :class="payClass">
                    {{payDesc}}
                </div>
            </div>
        </div>
        <transition name="drop">
        <div class="ball-container">
            <div v-for="(ball , index) in balls" :key="index" v-show="ball.show" class="ball">
                <div class="inner inner-hook"></div>
            </div>
        </div>
        </transition>
    </div>
</template>

<script>
    export default {
        props:{
            selectFoods:{
                type: Array,
                default(){
                    return [
                        {
                            price:10,
                            count:2
                        }
                    ]
                }
            },
            deliveryPrice:{
                type:Number,
                default: 0
            },
            minPrice:{
                type:Number,
                default:0
            }
        },
        data() {
            return {
                balls: [
                    {
                        show:false
                    },
                    {
                        show:false
                    },
                    {
                        show:false
                    },
                    {
                        show:false
                    },
                    {
                        show:false
                    }
                ],
                dropBalls:[]
            }
        },
        computed: {
            totalPrice() {
                let total = 0;
                this.selectFoods.forEach((food)=>{
                    total +=food.price*food.count
                })
                return total;
            },
            totalCount() {
                let count = 0;
                this.selectFoods.forEach((food)=>{
                    count +=food.count;
                });
                return count;
            },
            payDesc() {
                if(this.totalPrice === 0){
                    return `￥${this.minPrice}元起送`;
                }else if(this.totalPrice<this.minPrice){
                    let diff = this.minPrice-this.totalPrice
                    return `还差￥${diff}元起送`;
                }else{
                    return  '去结算';
                }
            },
            payClass() {
                if(this.totalPrice<this.minPrice){
                    return 'not-enough'
                }else{
                    return 'enough'
                }
            }
        },
        methods:{
            drop(el){
                for(let i=0;i<this.balls.length;i++){
                    let ball=this.balls[i]
                    if(!ball.show){
                        ball.show=true;
                        ball.el=el;
                        this.dropBalls.push(ball);
                        return;
                    }
                }
            },
            // 小球执行步骤:
            // 1. 点击cartcontrol组件中的”加号按钮“，$emit添加触发事件，将event.target作为参数，由goods父组件监听
            // 2. goods组件接收事件和target，执行_drop(target)，调用shopcart子组件的drop(el)方法，传入了target
            // 3. shopcart 组件执行 drop(el) 方法时，获取el在视口中的位置，编程式调用ball元素的过渡hook接口，执行css过渡

            // 过渡 钩子
            // 动画的原理是先把它偏移到一个位置，然后按照一定的轨迹和时间完成动画，动画完成后要把位置重置
            beforeDrop(el) {    // el 是Vue 钩子选择作用动画的 DOM 对象
                let count = this.balls.length;
                while (count--) {
                let ball = this.balls[count];
                if (ball.show) {
                    let rect = ball.el.getBoundingClientRect(); // 得到元素距视口各边的偏移量
                    // 是相对于小球初始位置的，所以 x 是正的，y 是负的
                    let x = rect.left - 32;
                    let y = -(window.innerHeight - rect.top - 36);
                    el.style.display = '';        // 手动设置为空，小球会显示出来
                    el.style.webkitTransform = `translate3d(0,${y}px,0)`;
                    el.style.transform = `translate3d(0,${y}px,0)`;
                    let inner = el.getElementsByClassName('inner-hook')[0];   // 获取当前el的innerDom节点
                    inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
                    inner.style.transform = `translate3d(${x}px,0,0)`;
                }
                }
            },
            dropping(el, done) {  // Vue enter 的第二个参数必须给，否则会设置成同步
                /* eslint-disable no-unused-vars */
                let rf = el.offsetHeight;
                // 触发浏览器重绘，可以保证 dom 位置渲染正确后再执行之后的动画
                this.$nextTick(() => {
                el.style.webkitTransform = 'translate3d(0,-10px,0)';
                el.style.transform = 'translate3d(0,-10px,0)';
                let inner = el.getElementsByClassName('inner-hook')[0];
                inner.style.webkitTransform = 'translate3d(0,0,0)';
                inner.style.transform = 'translate3d(0,0,0)';
                // 告知Vue，一个小球动画完成
                el.addEventListener('transitionend', done);   // 当动画结束，会有  CSS3 transitionend 事件派发
                });
            },
            afterDrop(el) {
                // 首先触发afterDrop是最先落的小球，所以把第一项也就是先落的小球重置
                let ball = this.dropBalls.shift();    // 删除dropBalls第一项，并返回此项
                if (ball) {
                ball.show = false;
                el.style.display = 'none';
                }
            },
        },
    }
</script>

<style lang="stylus" rel="stylesheet/stylus">
.shopcart
    position fixed
    left 0
    bottom 0
    z-index 50
    width 100%
    height 48px
    .content
        display flex
        background #141d27
        font-size 0
        color rgba(255,255,255,.4)
        .content-left
            flex 1
            .logo-wrapper
                display inline-block
                position relative
                vertical-align top
                top:-10px
                margin 0 12px
                padding 6px
                width 56px
                height 56px
                box-sizing border-box              
                border-radius 50%
                background #141d27
                .logo
                    width 100%
                    height 100%
                    border-radius 50%
                    background #2b333c
                    text-align center
                    &.highlight
                        background rgb(0,160,220)
                    .icon-shopping_cart
                        line-height 44px
                        font-size 24px
                        color #80858a
                        &.highlight
                            color #fff
                .num
                    position absolute 
                    top 0
                    right 0
                    width 24px
                    height 16px
                    line-height 16px
                    text-align center
                    border-radius 16px
                    font-size 9px
                    font-weight 400
                    color #fff
                    background rgb(240,20,20)
                    box-shadow 0 4px 8px 0 rgba(0,0,0,.4)
            .price
                display inline-block
                vertical-align top
                line-height 24px
                margin-top 12px
                padding-right 12px
                box-sizing border-box
                border-right 1px solid rgba(255,255,255,.1)
                font-size 16px
                font-weight 700
                &.highlight
                    color #fff
            .desc
                display inline-block
                vertical-align top
                line-height 24px
                margin 12px 0 0 12px
                font-size 10px
        .content-right
            flex 0 0 150px
            width 105px
            .pay
                height 48px
                line-height 48px
                text-align center
                font-size 12px
                &.not-enough
                    background #2b333b
                &.enough
                    background #00b43c
                    color #fff
    .ball-container
        .ball
            position fixed
            left 32px
            bottom 22px
            z-index 200
            transition all .4s
            .inner
                width 16px
                height 16px
                border-radius 50%
                background rgb(0,160,220)
                transition all .4s


            


</style>