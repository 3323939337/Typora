## 1.拉开序幕的setup
        1.理解：Vue3.0中的一个全新的配置项，值为一个函数
        2.setup是所有Composition Api（组合式api）的‘表演舞台’
        3.组件中所用到的数据、方法等均要写在setup中
        4.setup的两种返回值：
        1.返回一个对象（常用）
            返回的对象中的数据、方法在模板中均可以直接使用
        2.返回一个渲染函数 （了解）
            return ()=>{return h('h1','重工')}
        5；注意点：ww
            1.尽量不要与Vue2混用
                ·Vue2.x配置的data、methods中均可以访问到setup中返回的数据、方法
                ·Vue3中的setup无法读取到Vue2配置的data与methods等配置
                ·如果Vue2中的数据或者等等配置的名字和Vue3的中的名字起了冲突，以Vue3中的为准
            2.setup不能是一个async函数，因为async函数返回的是一个promise而不是对象，模板看不到promise中的数据
                补充：
                    当后期使用suspense组件时：使用suspense以及异步引入的时候，setup可以是async函数，返回值可以是promise类型的数据
            3.setup中的方法或者配置如果想读setup中的数据是不用this的
## 2.Ref函数
        1.作用：定义一个响应式的数据
        2.语法：const key = ref('value')
            ·创建一个包含响应式数据的引用对象（reference对象，简称ref对象）
            ·JS中操作数据：xxx.value
            ·模板中读取数据不需要添加.value，直接类似于在标签中用插值语法读取即可
        备注：
            ·接收的数据类型可以是基本数据类型，也可以是对象
            ·基本类型的数据：响应式依然是靠Object.defineProperty()方法通过getter和setter实现数据代理
            ·对象类型的数据：借助reactive()函数将数据打包成Proxy数据类型
## 3.Reactive函数
        1.作用：定义一个对象类型的响应式数据，（基本数据类型必须用Ref函数）
        2.语法：const 代理对象 = reactive(源对象)接收一个对象或者一个数组，返回一个代理对象Proxy的实例对象
        3.reactive函数定义的响应式数据是深层次的
        4.内部基于ES6的Proxy实现，通过代理对象操作对象的内部数据进行操作
## 4.Vue3.0中的响应式原理
        Vue2.x的响应式原理：
        ·实现原理：
            ·对象形式：通过Object.defineProperty()实现对属性的熟读、修改进行拦截（数据劫持）
            ·数组类型：通过重写更新数组的一系列方法来实现拦截（对数组的变更方法进行了包裹）
            ·存在的问题：
                1.无法监测到对数据的增加和删除的操作，也就是说增、删数据页面不会更新
                    这个方法可以通过this.$set()来解决，或者引入Vue，通过Vue.set解决
                2.无法通过下标的形式对数组进行修改
                    除了上面的解决办法可以使用以外，只能通过数组本身自带的方法（push、shift、pop、splice）对数组的数据进行改变
        Vue3的响应式原理：
        ·实现原理：
            ·通过Proxy（代理），拦截对象中任意属性的变化，包括。属性值的读写、属性的添加、属性的删除等
            ·通过reflect（反射），对源对象的属性进行操作
            const p = new Proxy(person,{
                //拦截对属性进行读取的操作
                get(target,propname){
                    return Reflect.get(target,propname)
                }
                //拦截对属性值的修改或者添加新属性的操作
                set(target,propname,value){
                    return Reflect.set(target,propname,value)
                }
                //拦截删除属性的操作
                deleteProperty(target,propname){
                    return Reflect.deleteProperty(target,propname)
                }
            })
## 5.ref对比Reactive
        ·从定义数据的角度
            ·ref定义数据：基本数据类型
            ·Reactive定义数据：对象（或数组）类型的数据
            ·备注：ref也可以定义对象或者数组类型的数据，但内部还是交给了reactive去处理
        ·从原理的角度
            ·ref通过Object.defineProperty()的get和set来实现响应式（数据劫持）
            ·Reactive通过Proxy来实现响应式（数据劫持），通过Reflect来操作源数据的内部的数据
        ·从使用对比的角度
            ·ref定义的数据在js中使用数据要用.value的形式，在模板中不用.value的形式
            ·reactive定义的数据不论是在js中还是在模板中都不用使用.value
## 6.setup中的两个注意点
    ·setup的执行时机：
        在beforeCreate前执行setup，所以setup中不能使用this来进行数据指向，此处的this是未定义
    ·setup的参数
        ·props：值为对象，其中包含的是外部传过来的切props中声明接收了的属性
        ·context：上下文对象
            ·attrs：外部传入的，props未声明接收的属性就会放到attrs里面
            ·emit：触发自定义事件的函数
            ·slots：收到插槽的内容
## 7.计算属性与监视
    1.计算函数
    ·作用：与Vue2中的计算属性功能一致
    ·写法
        import { computed } from 'vue'
        setup(){
            let person = reactive({
                firstName = '张',
                lastName = '三'
            })
            //简写
            person.fullName = computed(()=>{
                return person.firstName +'-'+ person.lastName
            })
            //完全
            person.fullName = computed({
                get(){
                    return person.firstName +'-'+ person.lastName
                },
                set(value){
                    const nameArr = value.split('-')
                    person.firstName = nameArr[0]
                    person.lastName = nameArr[1]
                }
            })
        }
    2.监视函数
        1.作用：与Vue2.x一致
        2.注意点：
            //情况一：监视ref函数定义的的一个响应式数据
            /* 监视行为是一种行为，所以不用return ,监视属性能收到三个参数，一个是监视的属性，第二个是监视的行为，第三个是监视的配置*/
            /* watch(sum,(nvalue,ovalue)=>{
            console.log('sum变了',nvalue,ovalue)
            },{immediate:true}) */
            
            //情况二：监视ref函数定义的多个响应式数据
            //通过数组规定监视多个对象，这里的newValue和oldValue是监视属性的newValue的集合以及s监视属性的oldValue的集合
            /* watch([sum,msg],(nvalue,ovalue)=>{
            console.log('sum或者msg变了',nvalue,ovalue)
            },{immediate:true})*/
            //情况三：监视reactive函数定义的一个响应式数据的全部属性
            /* 
                如果直接监视用reactive定义的一个响应式数据会导致：
                1.nvalue,ovalue都显示的是newvalue的值，也就是说无法正常获取到旧值
                2.deep配置项强制开启（也就是说deep配置无效）
            */
            /* watch(person,(nvalue,ovalue)=>{
            console.log('person改变了',nvalue,ovalue)
            }) */
            //情况四：监视reactive函数定义的一个响应式数据中的某个属性
            /* 
            在这种情况下newValue和oldValue能正常拿到，因为单独拿某个属性其实就相当于拿ref函数配置的属性，也就正常了
            */
            /* watch(()=>person.age,(nvalue,ovalue)=>{
            console.log('person的age改变了',nvalue,ovalue)
            }) */
            //情况五：监视reactive函数定义的一个响应式数据中的某些属性
            /* watch([()=>person.age,()=>person.name],(nvalue,ovalue)=>{
            console.log('person的age或name改变了',nvalue,ovalue)
            }) */
            //特殊情况
            /* 
            如果监视的对象不再是直接由reactive所定义的数据，而是reactive定义的一个数据中的某个具有
            深层次结构的对象，那么deep就不再强制开启，而是需要手动开启，否则就监测不到
            */
            watch(person.job,(nvalue,ovalue)=>{
            console.log('person的job改变了',nvalue,ovalue)
            },{deep:true})
    3.watchEffect监视
        ·watch监视的套路是：既要指明监视的对象，又要指明监视的回调
        ·watchEffect监视的套路时：不用指明监视的对象，只用指明监视的回调，回调中用到什么数据就监视什么数据
        ·watchEffect其实和计算属性很像：
            当computed依赖的数据变了，computed的回调就会执行；当watchEffect监视的数据变了，watchEffect监视的回调就会执行
            ·但是computed更加注重计算的结果，必须返回计算的值
            ·watchEffect监视的数据更加注重过程，也就是说监视的数据变了就行，不返回结果
## 8.自定义hook函数
    ·什么是hook：本质是一个函数，把setup中使用的组合式Api进行了一个封装
    ·类似于Vue2中的mixin（好几个组件都要使用的东西，就封装成一个hook）
    ·自定义hook的优势：复用代码，让setup中的逻辑更加清晰
## 9.toRef
    ·作用：创建一个ref对象，其value指向另一个对象的属性
    ·语法：const name = toRef(peron,'name')
    ·应用：要将响应式对象中的某个属性单独提交给外部使用时
    ·扩展:toRefs和toRef一致，但是toRefs能一次批量创建多个ref对象，语法：toRefs(person)
## 10.shallowRef以及shallowReactive
    ·shallowReactive只是定义最外层的属性为响应式（浅层响应）
    ·shallowRef只处理基本数据类型，不会把对象类型的数据转换为响应式（不会去借助reactive）
    ·什么时候使用？
        ·如果只有一个对象数据，结构比较深，但是变化时只是最外层的变化===>shallowReactive
        ·如果只有一个对象数据，后续功能不会修改该对象中的属性，而是使用新生的属性来替换====>shallowRef
## 11.readonly以及shallowReadonly
    ·readonly让一个数据变成只读（深只读）
    ·shallowReadonly让一个数据的第一层变成只读（浅只读）
    ·应用场景：不希望数据被修改
## 12.toRaw以及markRaw
    ·toRaw
        ·作用：将一个由reactive定义的响应式对象转换为普通对象
        ·使用场景：用于读取响应式对象对应的普通对象（类似于拿到源数据），对这个普通对象的所有操作都不会引起页面更新
    ·markRaw
        ·作用：标记一个对象，使其永远不会变成响应式对象
        ·应用场景：
            1.当有些值不应该被设置成响应式的，例如复杂的第三方库
            2.当渲染数据是一种拥有不可变数据的大列表时，跳过响应式可以提高性能
## 13.自定义Ref（customRef）
    ·作用：创建一个自定义的ref，并对“具依赖项追踪”和“更新触发”进行显示控制
    ·实现防抖效果：
        <template>
            <input type="text" v-model="keyword">
            <h4>{{keyword}}</h4>
        </template>
        <scrpit>
        import {customRef} from 'vue'
            export default{
                name:'App',
                setup(){
                    let timer
                    function myRef(value,delay){
                        return customRef((track,trigger)=>{
                            return{
                                get(){
                                    track()
                                    return value
                                },
                                ser(newValue){
                                    clearTimeout(timer)
                                    timer = setTimeout(()=>{
                                        value = newValue
                                        trigger()
                                    },delay)
                                }
                            }
                        })
                    }
                    let keyword = myRef('hello',500)
                    return{
                        keyword
                    }
                }
            }
        </script>
## 14.provide以及inject
    ·作用：实现祖与后代之间通信
    ·套路：祖先有一个provide选项来提供数据，后代有个inject来获取祖先提供的数据
    ·具体写法：
        祖先组件中：
            let car = reactive({name:xxx,price:xxx})
            provide('car',car)
        后代组件中：
            let car = inject('car')
            return{
                ...toRefs(car)
            }
## 15.isRef,isReactive,isProxy,isReadonly
    ·isRef：判断一个数据是否是通过ref生成的
    ·isReactive：判断一个数据是否是通过reactive生成的
    ·isReadonly：判断一个数据是否是只读类型的数据
    ·isProxy：判断一个数据是否是通过reactive或者readonly类型生成的
## 16.组合式Api的优点
    ·传统配置类型的api：所有的数据、方法、计算属性、监视属性等配置杂糅在一起很难找到，如果修改或者添加了一个数据，就需要在对应的方法、计算属性等配置里进行更改，如果此时面对的是一个大型的项目，这时在配置项中找东西就很麻烦
    ·组合式api的优势：前提是要借助hook函数，在借助hook函数的情况下，对数据的修改就只需要找到对应的hook的js文件，在单个的文件中去修改即可，能更加优雅的将代码中的数据、方法有序的组织在一起
## 17.新的组件
    1.fargment
        ·在Vue2中，所有的组件或者标签需要放在一个根组件里面
        ·在Vue3中，所有的组件都会包含在一个虚拟的fargment标签里面
        ·优点：减少标签级，减少内存占用
    2.teleport
        ·什么是teleport组件：teleport组件是一种可以将组件的html组件移动到指定位置的技术
        <teleport to="body">
            <div class="mask" v-show="isShow">
                <div class="dialog">
                    <h1>xxx<h1>
                    <h1>xxx<h1>
                    <h1>xxx<h1>
                    <button @click="isShow=false">点我关闭弹窗</button>
                </div>
            </div>
        </teleport>
    3.suspense
        ·等待异步组件时渲染一些额外的内容，让用户有更好的体验
        ·使用步骤：
            ·异步引入组件
            import {defineAsyncComponent} from 'vue'
            const child = defineAsyncComponent(()=>import('./component/child'))
            ·使用异步组件
            <suspense>
                //真正放置的组件
                <template v-slot:default>
                    <child/>
                </template>
                //真正放置的组件还未渲染时放置的内容
                <template v-slot:fallback>
                    <h4>稍等，加载中。。。。</h4>
                </template>
            </suspense>
            补充知识：
            当时用suspense时，setup可以是async类型的：
                async setup(){
                    let sum = ref(0)
                    let p = promise((resolve,reject)=>{
                        setTimeout(()=>{
                            resolve({sum})
                        },3000)
                    })
                    return await sum
                }
## 18.其他
    1.全局Api的转移
        ·Vue2中有许多全局Api和配置
        例如注册全局组件、注册全局指令等
        //注册全局组件
        Vue.Component('Mybutton',{
            data:()=>({
                count:0
            }),
            template:'<button @click="count++">Click{{count}} times.</button>'
        })
        //注册全局指令
        Vue.directive('focus',{
            inserted:el=>el.focus()
        })
        ·Vue3中对这些做出了调整
        将全局的Api，从Vue.xxx调整到了应用实例app身上
        Vue.config.xxx ====> app.config.xxx
        Vue.config.productionTip ====>移除
        Vue.component ====> app.component
        Vue.directive ====> app.directive
        Vue.mixin ====> app.mixin
        Vue.use ====> app.use
        Vue.prototype ====> app.config.globalProperties
    2.其他改变
        ·data应该始终被声明为一个函数
        ·过度类名的更改：
            Vue2.x的写法：
            v-enter
            v-leave-to{
                opacity:0
            }
            v-leave
            v-enter-to{
                opacity:0
            }
            Vue3的写法：
            v-enter-from
            v-leave-to{
                opacity:0
            }
            v-leave-from
            v-enter-to{
                opacity:0
            }
        ·移除了keyCode作为v-on（@）的修饰符，同时也不再支持config.keyCodes(这个方法在Vue2中用于通过一个按键的code定义一个按键)
        ·移除了v-on.native
            在Vue2中如果要给子组件绑定一个click事件需要添加一个.native例如：@click.native="xxxx"，不然Vue会认为这个事件是一个自定义事件，不通过点击触发
            而在Vue3中：
            父组件中：
            <Mycomponent @close="xxxx" @click="xxxx">
            </Mycomponent>
            子组件中：
            export default{
                name:'Mycomponent'
                emits:['close']//这里不通过emits接收click，那么click就不会作为一个自定义事件
            }
        ·移除过滤器（filter）