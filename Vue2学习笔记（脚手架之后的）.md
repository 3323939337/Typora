#笔记
## ref属性
    1.被用来给元素或者子组件注册引用信息（id得替代）
    2.应用在Html标签上获取到得是真实DOM元素，应用在组件标签上获取到的是组件实例对象
    3使用方式：
        打标识：<h1 ref="xxx">.....</h1>或者<VC ref="xxx"></Vc>
        获取：this.$refs.xxx

## props配置项：
    功能：接收外部的数据
    （1）.传递数据
        <demo name="xxx">
    （2）.接收数据
        第一种方式：只接收
        props:["xxx"]
        第二种方式：接收数据并对数据类型进行限制
        props:{
            name:String
        }
        第三种方式：接收数据并限制类型，限制必要性，限制默认值
        props:{
            name:{
                type:String,
                //下面这两个属性不同时存在
                required:ture,
                default:xx
            }
        }
    备注：props是只读的，如果对props中的数据进行更改，那么控制台会报错，但是如果确实需要修改，那么请给一个中转的值对props中的值进行储存，后续对props中的数据的更改就对这个中转值进行操作

## mixins配置项：
    功能：可以把多个组件所共有的配置提取成一个对象
    使用方法：
        第一步首先在目录中创建一个mixin的js文件，然后定义混合例如：
            {
                data(){}
                ......
            }
        第二部使用混合，例如：
            （1）全局混入：Vue.mixin(xxx),
            （2）局部混入：mixins['xxx']


## 插件：
    功能：用于增强Vue
    本质包含install方法的一个对象，install的第一个参数是Vue，第二个以后的参数是插件使用者传递的数据
    定义插件：
        对象.install = function(Vue,option){
            //注册全局过滤器
        Vue.filter('mySlice',function(value){
            return value.slice(0,4)
        })
    
        //定义全局指令
        Vue.directive('fbind',{
            bind(element,bindings){
                element.value = bindings.value
            },
            inserted(element,bindings){
                element.focus()
            }
        })
        ......
    }
    使用插件：
        Vue.use()


​    
## scoped:
    作用：让样式局部生效，防止冲突
    写法：<style scoped>


## TodoList案例总结：
    1.组件化编码流程：
        （1）拆分静态组件：组件按照功能点拆分，命名不要和html中的名字冲突
        （2）实现动态组件：考虑数据的存放位置，数据是一个组件再用还是一些组件在用：
            一个组件在用：就把这个数据放在自身；
            一些组件在用：把这个数据放在他们的父组件身上（状态提升）
        （3）实现交互：从绑定事件开始
    2.props适用于：
        （1）父==>子 通信
        （2）子==>父 通信，这个需要父组件提前给子组件传递一个方法
    3.使用v-model的时候要切记：v-model不能绑定props传递过来的值，因为这违背原则，porps的值只能被读不能被改
    4.如果props传递过来的值是多层次类型，那么修改内部层次的值不会被vue检测到，但不推荐这么做

## 组件的自定义事件：
    1.作用：是一种组件间的通信方式，适用于：子组件==>父组件
    2.使用场景：假设A是父组件，B是子组件，B想传递数据给A，那么A就要给B绑定自定义事件，但是回调在A身上
    3.绑定自定义事件：
        1.第一种方法，在父组件中：<Demo @atguigu="test">或<Demo v-on:atguigu="test">
        2.第二种方法：在父组件：
            <Demo ref="demo">
            ......
            mounted(){
                this.$ref.demo.$on('atguigu',this.test)
            }
        3.若是只想自定义事件触发一次，可以使用$once方法,或者on修饰符
    4.触发自定义事件this.$emit('atguigu',数据)
    5.解绑自定义事件this.$off('atguigu')
    6.组件上也可以绑定原生事件，通过.native修饰符
    7.注意：通过this.$ref.demo.$on('atguigu',回调)，此处回调中的this是子组件的实例对象，若想解决问题，要么把回调方法
    配置在methods中，要么用箭头函数

## 全局事件总线：
    1.是一种组件间通信的方法，适用于任意组件间的通信
    2.安装全局总线的方法：
        new Vue({
            .....
            beforeCreate(){
                Vue.protoType.$bus = this 这里的$bus就是vm
            }
        })
    3.使用事件总线：
        1.接收数据：A组件想要收到数据，首先要在$bus上绑定一个自定义事件，事件的回调留在A组件：
        methods:{
            demo(){
                ....
            }
        }
        mounted(){
            this.$bus.$on('xxxx',this.demo)
        }
        2.提供数据：this.$emit('xxxx',数据)
    4.最好在beforeDestroy()中解绑当前组件所用到的自定义事件

## 消息订阅与发布：
    1.是一种任意组件间通信的方式
    2.使用步骤：
        （1）安装pubsub：npm i pubsub-js
        （2）引入pubsub： import pubsub from 'pubsub.js'
        （3）接收数据：A组件想要接收消息，那么A组件就要订阅消息，订阅消息的回调函数在A组件身上：
            const pId = pubsub.subscribe('xxx',demo(MsgName,data)=>{
    
            })
            注意：这里的回调函数要写成箭头函数的形式，或者提前在methods中配置好方法让this指向是Vc
            发送数据：B组件发送数据：
            pubsub.publish('hello',MsgName,data)
        （4）最好在beforeDestroy中通过pId取消订阅的消息：
         pubsub.unsubscribe(pId)
    注意：消息订阅与发布和全局事件总线一样都是任意组件间通信的方法，但是由于消息订阅与发布涉及到了第三方库，
    而全局事件总线用的全都是Vue自己的技术，所以两者相比之下更推荐使用全局事件总线






## $nextTick()函数：
    1.语法：$nextTick(回调函数)
    2.作用：在下一次DOM更新结束后执行其回调
    3.什么时候用：在改变数据后，要基于更新后的DOM进行某些操作时，要在nextTick所指定的回调函数中执行

## Vue封装的过渡与动画：
    1.作用：在DOM元素插入、更新、移除时，在合适的时候给元素添加样式类名
        注意：每个过渡有三个部分：起点、终点以及整个过程
    2.写法：
        1.准备好样式：
            1.v-enter：进入的起点2.v-enter-to：进入的终点3.v-enter-active：进入的整个过程（离开几把enter变成leave）
        2.使用<transition>包裹要过渡的元素，并配置name
            name是为了防止又多个元素使用过渡效果，不能都用v-....
        3.若有多个元素需要过渡，要使用<transition-group>




## Vue脚手架配置代理：
    方法一：
    在vue.config.js中添加如下配置：
    devServer:{
        proxy:'http://localhost:xxxx（目标地址端口）'
    }
    说明：
    优点：配置简单，请求资源时直接发送给8080即可
    缺点：不能配置多个代理，不能选择虚拟服务器是否发送请求
    工作方式：按照如上配置，如果请求了本机端口不存在的资源时，虚拟服务器则会发送请求给接收请求的服务器（本地资源优先）
    方法二：
    在vue.config.js添加如下配置：
    devServer:{
        proxy:{
            '/xxx':{
                pathRewrite:{'^/xxx':''},
                target:'http://localhost:xxxx'//目标地址的基础地址
                changeOrigin:true,
                ws:true,//开启websocket支持    
            }，
            '/xxxx':{
                pathRewrite:{'^/xxxx':''},
                target:'http://localhost:xxxxxx'
                changeOrigin:true,
                ws:true,//开启websocket支持    
            }
        }
    }
    说明：
    缺点：配置略微繁琐，发送请求时必须加前缀
    优点：可以配置多个代理，可以选择发送请求是否走代理

## 插槽：
    1.定义：是父组件向子组件指定的位置放置一个html结构的方法，也是一种组件间通信的方法，适用于：父组件==>子组件
    2.分类：默认插槽，具名插槽，作用域插槽
    3.使用方式：
        1.默认插槽：
            父组件中：
            <Categroy>
                <div></div>
            </Categroy>
            子组件中：
            <template>
                <div></div>
                <slot></slot>
            </template>
        2.具名插槽：
            父组件中：
            <Categroy>
                <div slot:xxx></div>
                <template v-slot:xxxx><div></div></template>
            </Categroy>
            子组件中：
            <template>
                <div></div>
                <slot name="xxx"></slot>
                <slot name="xxxx"></slot>
            </template>
        3.作用域插槽：(适用于使用插槽的组件想要定义插槽的组件的数据，而这个数据不能直接拿到使用插槽的组件身上的情况)
        也就是数据在组件的自身，但是根据数据生成的结构需要组件的使用者来决定
        注：作用域插槽可以和具名插槽结合使用
            父组件中：
            <Categroy>
                <div scope="getdata" v-for="g in games">{{getdata.games}}</div>
                <template slot-scope="getdata"><div v-for="g in games">{{getdata.games}}</div></template>
            </Categroy>
            子组件中：
            <template>
                <div></div>
                <slot :games="games"></slot>
            </template>

## Vuex:
    1.概念
        在Vue中实现集中状态（数据）管理的一个Vue插件，对Vue中多个组件的共享状态进行一个集中式的管理（读/写），也是
        一种组件间通信的方式，适用于任意组件间通信
    2.何时使用
        多个组件需要共享数据时
    3.搭建vuex环境
        //引入Vue
        import Vue from 'vue'
        //引入Vuex
        impore Vuex from 'vuex'
        //应用Vuex插件
        Vue.use(Vuex)
        //定义actions--用于响应组件中用户的动作
        const actions = {}
        //定义mutations--修改state中的数据
        const mutation = {}
        //定义state--用于存储数据
        const state = {}
        //创建并暴露store
        export dafault new Vuex.Store({
            actions,
            mutation,
            state,
        })
    4.在main.js下导入创建好的store
        import Vue from "vue";
        import App from "./App.vue";
    
        //引入store
        import store from "./store";
    
        Vue.config.productionTip = false



        new Vue({
            el:`#app`,
            render:h=>h(App),
            store,
            beforeCreate(){
                Vue.prototype.$bus = this
            }
        })
    5.在组件中读取vuex中的数据：
        this.$state.store.num
    6.在组件中向vuex传递数据：
        （this.）$state.dispatch('方法名',传递的数据)，用于给actions传递数据，这里的this如果实在模板里就不用加
        或者：this.$state.commit('方法名'，传递的数据)，用于给mutations传递数据
    7.getters的使用：
        1.概念：当state中的数据需要加工后再使用时(当逻辑复杂并且需要复用的时候)，就可以使用getters进行加工
        2.再index.js中配置getters配置项：
            const getters(state){
                xxxx(){
                    return state.xxxxxx (进行加工的各种操作)
                }
            }
        3.在组件中使用数据：
            $store.getters.xxxx
    8.四个map的使用：
        1.mapState：
            作用：生成计算属性，借助state中的数据
            对象写法：
            ...mapState({he:'sum'})
            数组写法：
            ...mapGetters(['sum'])(计算属性和state中的数据一致时才能这么使用)
        2.mapGetters：
            作用：生成计算属性，借助state中的数据
            对象写法：
            ...mapGetters({he:'sum'})
            数组写法：
            ...mapGetters(['sum'])(计算属性和state中的数据一致时才能这么使用)
        3.mapactions:
            作用：帮助程序员生成与actions对话的方法，方法中会调用this.$store.dispatch()方法
            对象写法：
            ...mapactions{he:'sum'})
            数组写法：
            ...mapactions(['sum'])(计算属性和state中的数据一致时才能这么使用)
        3.mapMutations:
            作用：帮助程序员生成与mutations对话的方法，方法中会调用this.$store.commit()方法
            对象写法：
            ...mapmutations{he:'sum'})
            数组写法：
            ...mapmutations(['sum'])(计算属性和state中的数据一致时才能这么使用)
    9.模块化+命名空间
        1.目的让代码更好维护，让多种数据分类明确
        2.修改index.js的代码（开启命名空间）
            const countAbout={
                namespaced:true,
                acions:{
                    xxxxx
                },
                mutations:{
                    xxxxx
                },
                state:{
                    xxxxx
                },
                getters:{
                    xxxxx
                },
            }
            const personAbout={
                namespaced:true,
                acions:{
                    xxxxx
                },
                mutations:{
                    xxxxx
                },
                state:{
                    xxxxx
                },
                getters:{
                    xxxxx
                },
            }
            export default new Vuex.store({
                modules:{
                    personAbout,
                    countAbout
                }
            })
        3.开启命名空间后，组件读取数据的方式要发生改变，因为原本分散的数据都归于各自的模块。state中存储的就是模块了

## 路由：
    1.理解：一个路由就是一组映射关系（key:value），多个路由需要路由器进行管理
    2.关系：key：路径，value：组件
        1.基本使用
            1.安装vue-router:npm i vue-router
            2.应用VueRouter:Vue.use(VueRouter)
            3.创建一个路由器：
                new VueRouter({
                    routers:[
                        {
                            path:'/xxxx',
                            component:xxx
                        }
                        {
                            path:'/xxxx',
                            component:xxx
                        }
                    ]
                })
            4.在main.js中使用创建的路由器
                new Vue({
                    el:'#app',
                    render:h=>h(app)
                    router:router
                })
            5.通过router-link标签实现单页面跳转：
                <router-link to='/xxxx'></router-link>
            6.指定位置放置组件
                <router-view></router-view>
        2.几个注意点
            1.路由组件通常存放在pages文件夹中
            2.通过切换而隐藏了的路由组件其实是被销毁了
            3.每个路由组件都有自己的$route，里面存放着自己的路由信息
            4.每个应用只有一个router,可以通过组件的$router访问到
        3.多级路由
            1.配置多级路由规则：children配置项
                routes:[
                    {
                        path:'/xxx'
                        compontent:xxx
                    },
                    {
                        path:'/xxx'
                        compontent:xxx
                        children:[//通过children配置项配置子路由
                            {
                                path:'xxx'//这里的路径不能添加/,当vue检测到是children下的路由时会自动添加/
                                component:xxx
                            },
                            {
                                path:'xxx'
                                component:xxx
                            },
                        ]
                    },
                ]
            2.query参数：这种传参方式完全不会打扰路由配置
                1.传递参数：
                    字符串写法：
                    <router-link :to="`xxx/xxx/xxx/?id=${xx.id}&title=${xx.title}`">{{}}</router-link>
                    对象写法：
                    <router-link :to="{
                        path:'xxx/xxx/xxx',
                        query:{
                            id:xx.id,
                            title:xx.title
                        }
                    }">
                        {{}}
                    </router-link>
                2.接收参数
                    <li>{{$route.query.id}}</li>
                    <li>{{$route.query.title}}</li>
            3.命名路由
                1.作用：可以简化路径的跳转（一般用于过长的路径，一级路由通常不加）
                2.如何使用
                    1.给路由命名：
                        export default new VueRouter({
                            router:[
                                {
                                    path:'/xxx'
                                    component:xxx
                                }
                                {
                                    path:'/xxx'
                                    component:xxx
                                    children:[
                                        {
                                            name:'xxx',
                                            path:'/xxx'
                                            component:xxx
                                        }
                                    ]
                                }
                            ]
                        })
                    2.简化跳转：
                        简化前：<router-link to='/xxx/xxx/xxx'></router-link>
                        简化后：<router-link :to='{name='xxx'}'></router-link>//这里必须写成对象形式的配置项
            4.params参数
                1.配置路由，使用占位符声明接收params参数
                    export default new VueRouter({
                        router:[
                            {
                                path:'/xxx'
                                component:xxx
                            },
                            {
                                path:'/xxx'
                                component:xxx
                                children:[
                                    {
                                        name:'xxx',
                                        path:'/xxx/xxx/xxx/:xxx/:xxx'
                                        component:xxx
                                    }
                                ]
                            }
                        ]
                    })
                2.传递参数
                    to的字符串形式：
                    <router-link :to="`xxx/xxx/xxx${m.id}/${m.title}`"></router-link>
                    to的对象形式：
                    <router-link :to="{
                        name:'xxx',//params参数的对象形式传递参数必须用name的路径进行跳转
                        params:{
                            id:m.id
                            title:m.title
                        }
                    }">
                    </router-link>
                3.接收参数
                    <li>{{$route.params.id}}</li>
                    <li>{{$route.params.title}}</li>
            5.路由的props配置
                {
                    name:'detail',
                    path:'detail',//接收params参数这里要用占位符占住，并且这里的命名才是控制台收到的参数名
                    component:Detail,
                    //props的第一种写法，值为对象(用的很少，因为只能传递死数据)
                    /* props:{a:'001',b:'你好啊'} */
                    //props的第二种写法，值为布尔值,会将传递的所有params参数以props的形式传递过去(但是只能是params参数)
                    /* props:true */
                    //props的第三种写法，值为函数
                    props($route){
                        return {id:$route.query.id,title:$route.query.title}
                    }
            6.router-link的replace属性
                1.作用：改变router-link操作浏览器记录的模式（默认是push）
                2.浏览器的历史记录有两种操作模式，一种是push，一种是replace，push不会破坏掉之前的浏览器历史，是追加的方式，而replace是替换掉当前的浏览器历史
                3.开启方式
                    再router-link标签中添加一个replace即可
            7.编程式路由导航
                1.作用：不借助<router-link>进行路径跳转
                2.具体编码：
                    $router的两个api
                    pushShow(m){//通过push的方式
                        this.$router.push({
                            name:'detail',//params传递参数，这里必须用name进行路径跳转
                            query:{
                            id:m.id,
                            title:m.title
                            }
                            })
                        },
                    replaceShow(m){
                        this.$router.replace({
                            name:'detail',
                            query:{
                            id:m.id,
                            title:m.title
                        }
                        })
                    }
                    back(){
                        this.$router.back()
                    },
                    forward(){
                        this.$router.forward()
                    },
                    go(){
                        this.$router.go(3)//可以通过参数中的正负数来前进或者后退对应的几步，比如3就是前进3步
                    }
            8.缓存路由组件
                1.作用：让没有被展示的组件不被销毁
                2.具体编码：
                <keep-alive include="News">//注意这里是用的组件名！！！！！
                如果这里的include想包含多个路由组件那么就要用数组的形式：
                <keep-alive :include="["News","xxx"]">
                    <router-view></router-view>
                </keep-alive>
            9.路由的两个生命周期：
                1.作用：两个专属于路由组件的生命钩子，用于捕获路由组件的激活状态
                2.具体名字：
                    activated()//在组件被显示在页面之后，也就是激活后调用内部的函数
                    deactivated()//在组件不在页面上展示后，失活后会调用内部的函数
            10.路由守卫
                1.作用：对路由进行权限判断
                2.分类：全局守卫，独享守卫，组件内守卫
                3.全局守卫
                    //全局前置守卫，初始化执行，每次路由切换前执行
                    router.beforeEach((to,from,next)=>{
                        if(to.meta.isAuth=='true'){//判断是否需要鉴权
                            if(localStorge.getItem('school')=='cqie'){//权限的条件
                                next()//放行
                            }else{
                                alert('暂无权限')
                            }
                        }else{
                            next()
                        }
                    })
                    //全局后置守卫，在初始化以及路由切换后进行执行
                    router.afterEach((to,from)=>{
                        document.title == to.meta.title || 'xxxx'
                    })
                3.独享守卫
                    //某一个路由单独使用的守卫，只对这个路由生效
                    //可以和全局后置路由守卫搭配使用
                    beforeEnter(to,from,next){
                        if(to.meta.isAuth=='true'){//判断是否需要鉴权
                            if(localStorge.getItem('school')=='cqie'){//权限的条件
                                next()//放行
                            }else{
                                alert('暂无权限')
                            }
                        }else{
                            next()
                        }
                    }
                4.组件内路由守卫
                    //在组件内使用的路由守卫
                    //必须是遵循路由规则执行跳转的（也就是说，比如直接挂载在页面上的路由组件不奏效）
                    beforeRouteEnter(to,from,next){
    
                    }
                    beforeRouteLeave(to,from,next){
            11.路由器得两种工作模式
                1.对于一个url来说，什么是hash值：就是#后面得所有得值就是hash值
                2.hash值不会包含在http请求中，也就是说不许被当作资源请求
                3.hash模式：
                    1.地址中永远都会带着#
                    2.若以后将地址通过第三方手机app分享，若app校验严格，该地址会被认为不合法
                    3.兼容性较好
                4.history模式：
                    1.地址干净，美观
                    2.兼容性相较于hash较差
                    3.应用部署上线后需要后端人员支持，解决刷新服务器后404得问题