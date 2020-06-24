## vue-aliplayer-v2组件的使用

### 安装依赖

```
npm i vue-aliplayer-v2 --save
or
yarn add vue-aliplayer-v2 
```



### 全局注册main.js

```
import VueAliplayerV2 from 'vue-aliplayer-v2';

Vue.use(VueAliplayerV2);
```



### 局部注册 App.vue

```
import VueAliplayerV2 from 'vue-aliplayer-v2';
components:{ VueAliplayerV2: VueAliplayerV2.Player }
```



### 使用

```vue
<div v-if="this.fileInfo!==undefined" style="width: 500px" class="video-div">
   <vue-aliplayer-v2 :source="source" ref="VueAliplayerV2"/>
</div>
```

```vue
// data
// http://ivi.bupt.edu.cn/hls/cctv1hd.m3u8  m3u8测试地址

 data () {
    return {
		source: ''
	}
 }

```



### 参考资料：

https://github.com/langyuxiansheng/vue-aliplayer-v2