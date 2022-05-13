---
title: Vite+Vue3+TSX 实践  
date: 2022-05-09 19:30:00  
author: Marshall  
img: https://unsplash.it/1920/1080?random8  
top: true  
cover: true  
toc: true  
mathjax: false  
summary: 在项目中将TSX搭配Vue进行开发  
categories: Vue  
reprintPolicy: cc_by  
tags:
- TypeScript
- JSX/TSX
- Vue
- Vite
---

在开发时，碰到一个需求：利用Element Plus的MessageBox组件实现一个弹框选择器，即MessageBox+Select组件搭配使用，要将Select放在MessageBox的message配置项里，但message接受的类型为`string | VNode`，所以要搭配Vue的render函数`h()`来实现。  
思路就是将Select组件封装好后，用`h()`转换为`VNode`，一开始本打算直接在message配置项中写好`h()`，但是实际实现的时候比较麻烦，最后决定将Select用TSX封装后用`h()`转换为VNode最方便。  

## tsx支持  
首先需要安装官方维护的vite插件`@vitejs/plugin-vue-jsx`  
```bash
yarn add @vitejs/plugin-vue-jsx -D
```
安装好后在`vite.config.ts`配置，代码如下  
```typescript
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import vueJsx from "@vitejs/plugin-vue-jsx";

export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
  ],
})

```

## 封装Select组件  
我想封装一个可刷新Option的组件，结构为Select + button > Option，并且需要父子组件间通信，即需要有`props`和`emit`。代码如下:  
```html
<!--child.vue-->
<script lang="tsx">
import { Refresh } from '@element-plus/icons-vue'
import axios, { AxiosRequestConfig } from 'axios';
import { defineComponent, ref, reactive, onMounted } from "vue"
export default defineComponent({
    props: ['id'], //显式声明 prop
    setup(props, { emit }) {
        let id = props.id
        interface optionsProps {
            label: string
            value: string
        }
        const options = reactive<optionsProps[]>([])
        //刷新数据的函数
        const getOptionsList = async () => {
            options.splice(0, options.length)
            const config: AxiosRequestConfig = {
                method: 'get',
                url: '/example',
                headers: {}
            };
            let data = await axios(config)
            let res = data.data
            res.map((item: string) => {
                if (item !== id) {
                    options.push({
                        label: item,
                        value: item
                    })
                }
            })
        }
        const selectValue = ref('')
        //组件发出的自定义事件，将Select选中的数据传递给父组件
        const selectChange = (val: string) => {
            emit("getSelect", val)
        }
        onMounted(() => {
            getOptionsList()
        })
        return () => (
            <div class="container">
                //Vue中的tsx规范
                <el-select v-model={selectValue.value} onChange={selectChange}>
                    // v-for
                    {
                        options.map((item) => {
                            return <el-option
                                key={item.value}
                                label={item.label}
                                value={item.value}
                            />
                        })
                    }
                </el-select>
                <el-button icon={Refresh} size="large" onClick={getOptionsList} circle class="btn" />
            </div>
        )
    }
})
</script>

<style scoped>
.container {
    display: flex;
    align-items: center;
    justify-content: center;
}

.btn {
    margin-left: 10px;
}
</style>
```

## 在MessageBox使用封装好的组件
代码如下
```html
<!--parent.vue-->
<script setup lang="ts">
import { h, ref } from 'vue'
import { ElMessageBox } from 'element-plus'
import child from '@/components/child.vue'

const selectValue = ref('')

ElMessageBox({
  message: h(child, {
    id,//传递给child的prop
    onGetSelect: (val: string) => {
      selectValue.value = val
    }//emit
  }),
  confirmButtonText: '发送请求',
  beforeClose: (_action, _instance, _done) => {
    //code
  }
})
</script>

```
