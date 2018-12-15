<template>
    <div class="goods">
        <div class="menu-wrapper" ref="menuWrapper">
            <ul>
                <li v-for="(item ,index) in goods" :key="index" class="menu-item" :class="{'current':currentIndex === index}" @click="selectMenu(index,$event)" ref="menuList">
                    <span class="text border-1px">
                        <span v-show="item.type>0" class="icon" :class="classMap[item.type]"> </span>
                        {{item.name}}
                    </span>
                </li>
            </ul>
        </div>
        <div class="foods-wrapper" ref="foodsWrapper">
            <ul>
                <li v-for="(item , index) in goods" :key="index" class="food-list" ref="foodList">
                    <h1 class="title">{{item.name}}</h1>
                    <ul>
                        <li v-for="(food,index) in item.foods" :key="index" class="food-item border-1px">
                            <div class="icon">
                                <img width="57" height="57" :src="food.icon" alt="">
                            </div>
                            <div class="content">
                                <h2 class="name">{{food.name}}</h2>
                                <p class="desc">{{food.description}}</p>
                                <div class="extra">
                                    <span>月售{{food.sellCount}}份</span>
                                    <span>好评率{{food.rating}}%</span>
                                </div>
                                <div class="price">
                                    <span class="now">￥{{food.price}}</span>
                                    <span class="old" v-show="food.odlPrice">{￥{food.oldPrice}}</span>
                                </div>
                                <div class="cartcontrol-wrapper">
                                    <cartcontrol :food="food" @add="addFood"></cartcontrol>
                                </div>
                            </div>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
        <shopcart ref="shopcart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    </div>
</template>

<script>
    import BScroll from 'better-scroll'
    import shopcart from '@/components/shopcart/shopcart';
    import cartcontrol from '@/components/cartcontrol/cartcontrol'
    const ERR_OK=0
    export default {
        props: {
            seller:{
                type: Object
            }
        },
        data() {
            return {
                goods:[],
                listHeight: [],
                scrollY: 0,
                selectedFood: {}
            }
        },
        computed:{
            currentIndex(){//当前选项的class样式
                for(let i=0;i<this.listHeight.length;i++){
                    let height1=this.listHeight[i];
                    let height2=this.listHeight[i+1];
                    if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)){//如果最后一个height不存在，直接返回i
                        this._followScroll(i);
                        return i;

                    }
                }
                return 0;
            },
            selectFoods() {
                let foods =[];
                this.goods.forEach((good)=>{
                    good.foods.forEach((food)=>{
                        if(food.count){
                            foods.push(food);
                        }
                    })
                })
                return foods;
            }
            
            
        },
        created() {
            this.$http.get('/api/goods').then((response)=>{
                response = response.body;
                if(response.errno===ERR_OK){
                    this.goods = response.data
                    this.$nextTick(()=>{//为了确保DOM已经渲染
                        this._initScroll();
                        this._caculateHeight();
                    })
                }
            })
            this.classMap=['decrease','discount','special','invoice','guarantee']
        },
        methods: {
            _initScroll() {
                //BScroll 依赖于原生DOM
                this.menuScroll = new BScroll(this.$refs.menuWrapper,{
                    click : true //开启点击事件（BS默认会阻止掉）
                });
                this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{
                    click:true,
                    probeType:3     //设定BS实时监测滚动位置
                })
                this.foodsScroll.on("scroll",(position)=>{
                    //判断滑动方向，避免下拉时分类高亮错误（如第一分类商品数量为1时，下拉使得十二分类高亮）
                    if(position.y<=0){
                        this.scrollY = Math.abs(Math.round(position.y))
                    }
                });
            },
            _caculateHeight(){   //计算高度
                let foodList = this.$refs.foodList;//每个foodlist的高度就是每个区块的高度
                let height = 0;
                this.listHeight.push(height);
                for(let i=0;i<foodList.length;i++){
                    height +=foodList[i].clientHeight; //clientHeight 每个区块的可是高度
                    this.listHeight.push(height)
                }
            },
            selectMenu(index,event) {
                if(!event._constructed){
                    return;
                }
                let foodList = this.$refs.foodList;
                let el = foodList[index];
                this.foodsScroll.scrollToElement(el,300); 

            },
            addFood(target) {//监听子元素的add事件，执行addFood函数
                this._drop(target)

            },
            _drop(tartget) {
                //体验优化，异步执行下落动画 
                this.$nextTick((target)=>{
                    this.$refs.shopcart.drop(target);
                    
                })
            },
            _followScroll(index) {
                let menuList = this.$refs.menuList;
                let el=menuList[index];
                this.menuScroll.scrollToElement(el,300)
            }

        },
        components:{
            shopcart,
            cartcontrol
        },
        events:{
            'cart.add'(target) {
                this._drop(targets)
            }
        }
    }
</script>
<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin";
.goods
    display flex
    position absolute 
    top 174px
    width 100%
    bottom 46px
    overflow hidden
    .menu-wrapper
        flex 0 0 80px
        width 80px
        background #f3f5f7
        .menu-item
            display table
            height 54px
            width 56px
            line-height 14px
            padding:0 12px
            &.current
                position relative
                z-index 10
                margin-top -1px
                background #ffffff
                .text
                    @include border-none()
                    color #000
                    font-weight 500 
            .icon 
                display inline-block
                width 12px
                height 12px
                margin-right 2px
                vertical-align middle
                background-size 12px 12px
                &.decrease
                    bg-image('decrease_3')
                &.discount
                    bg-image('discount_3')
                &.guarantee
                    bg-image('guarantee_3')
                &.invoice
                    bg-image('invoice_3')
                &.special
                    bg-image('special_3')
            .text
                display table-cell
                width 56px
                vertical-align middle
                border-1px(rgba(7,17,27,.1))
                font-size 12px


    .foods-wrapper
        flex 1
        .title
            padding-left 14px
            height 26px
            line-height 26px
            border-left 2px solid #d9dde1
            font-size 12px
            color rgb(147,153,159)
            background #f3f5f7
        .food-item
            display flex
            margin 18px
            padding-bottom 18px
            border-1px(rgba(7,17,27,.1))
            &:last-child
                border-none()
                margin-bottom 0
            .icon
                flex 0 0 57px
                margin-right 10px
            .content
                flex 1
                .name
                    margin 2px 0 8px 0
                    height 14px
                    line-height 14px
                    font-size 14px
                    color rgb(7,17,27)
                .desc ,.extra
                    line-height 10px
                    font-size 10px
                    color rgb(147,153,159)
                .desc
                    margin-bottom 8px
                .extra
                    &.count
                        margin-right 12px
                .price
                    font-weight 700
                    line-height 24px
                    .now
                        margin-right 8px
                        font-size 14px
                        color rgb(240,20,20)
                    .old
                        text-decoration line-through
                        font-size 10px
                        color rgb(147,153,159)
                .cartcontrol
                    position absolute 
                    right 0
                    bottom 5px




</style>