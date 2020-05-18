<br>

# BootStrapVue  Tags 标签搜索、筛选 排除选择标签 实例 - 详解

<br>

 
 ## 文章涉及：
 
**Vuex、Vue、BoostrapVue**
<br>

## 文章解析 ：
**Vue 中 使用 tags 标签 实现 输入名称查询标签列表 排除 已选择标签**
**默认展示全部 可搜索 tags 标签列表  模糊搜索 可用 tags 标签 列表**

<br>


## 文章概览：

**computed  计算 函数 criteria 监听 input 输入内容**

**availableOptions 函数 根据条件获取标签列表 计算可选 tags 标签**

**optional_tag_inputDesc 函数 搜索结束提示**

**add_option_to_tags_Click 函数 实现 添加 选中 tags**


<br>

## 实现效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200518161359303.gif)
<br>

## 实例 教程
<br>


**store.js**

```
import {getp} from '../config/http'

export const state = () => ({
  optional_tags: []
});

export const mutations = {
  set_optional_tags(state, payload) {
    state.optional_tags = payload
  }
};

export const actions = {
  async get_optional_tags({commit, state}, payload) {
    let optional_tags_result = await getp("demo/label_list_by_name.php", new URLSearchParams({"qname": payload.qname}));
    commit("set_optional_tags", optional_tags_result)
  }
}

```

****
**代码解析：**

**actions =>get_optional_tags 异步获取标签列表=>commit=> set_optional_tags=>更新状态**
****


**Template**

```
<template>
  <b-form-group label="标签">
    <b-form-tags v-model="tags" no-outer-focus class="mb-2">
      <template v-slot="{ tags,disabled, addTag, removeTag }">
        <ul v-if="tags.length > 0" class="list-inline d-inline-block mb-2">
          <li v-for="tag in tags" :key="tag" class="list-inline-item">
            <b-form-tag
              @remove="removeTag(tag)"
              :title="tag"
              :tag_disabled="tag_disabled"
              variant="primary"
            >{{ tag }}
            </b-form-tag>
          </li>
        </ul>

        <b-dropdown size="sm" variant="outline-secondary" block menu-class="w-100">
          <template v-slot:button-content>
            <b-icon icon="tag-fill"></b-icon>
            选择标签
          </template>
          <b-dropdown-form @submit.stop.prevent="() => {}">
            <b-form-group
              label-for="tag-search-input"
              label="Search tags"
              label-cols-md="auto"
              class="mb-0"
              label-size="sm"
              :description="optional_tag_inputDesc"
              :disabled="disabled"
            >
              <b-form-input
                v-model="optional_tag_input"
                id="tag-search-inpu"
                type="search"
                size="sm"
                autocomplete="off"
              ></b-form-input>
            </b-form-group>
          </b-dropdown-form>
          <b-dropdown-divider></b-dropdown-divider>
          <b-dropdown-item-button
            v-for="option in availableOptions"
            :key="option"
            @click="add_option_to_tags_Click({ option, addTag })"
          >
            {{ option }}
          </b-dropdown-item-button>
          <b-dropdown-text v-if="availableOptions.length === 0">
            没有可供选择的标记
          </b-dropdown-text>
        </b-dropdown>
      </template>
    </b-form-tags>
  </b-form-group>
</template>
```



<br>
<hr>

**深入理解：**

**b-form-group=> :description  结果提示**

**b-form-input => data 动态获取=>标签=>名称=>结果**

**b-dropdown-item-button =>availableOptions 展示可选标签->标签 =>点击=>添加标签**

****


**Script**

```
<script>
  import {mapState} from "vuex";

  export default { 
    name: "demo",
    computed: {
      ...mapState({optional_tags: (state) => state.writer_article.optional_tags}),
      criteria() { 
        return this.optional_tag_input.trim().toLowerCase()
      },
      availableOptions() {
        const criteria = this.criteria 
        let result = this.$store.dispatch("get_optional_tags", {qname: this.criteria})
        if (result) {
          let temp_tags = []
          for (var optional_tag in  this.optional_tags) {
            temp_tags.push(this.optional_tags[optional_tag]['name'])
          }
          const options = temp_tags.filter(opt => this.tags.indexOf(opt) === -1)
          if (criteria) {
            return options.filter(opt => opt.toLowerCase().indexOf(criteria) > -1);
          }
          // Show all options available
          return options
        }
      },
      optional_tag_inputDesc() {
        if (this.criteria && this.availableOptions.length === 0) {
          return '没有与optional_tag_input条件匹配的标记'
        }
        return ''
      }
    },
    data() {
      return {
        optional_tag_input: '',//搜索标签文字
        tags: [],//选中标签数组
      }
    },
    methods: {
      add_option_to_tags_Click({option, addTag}) {
        addTag(option)
        this.optional_tag_input = ''
      }
    }
  }
</script>


```
<hr>

**深入理解：**

**computed：计算函数=>...mapState 获取 store 状态 映射 optional_tags 为本地 可选标签列表**

**availableOptions => criteria筛选 输入内容 格式为小写**

**this.$store.dipatch("get_optional_tags")=>条件获取 标签列表=>格式数据 =>存放 temp_tags**

**const options = temp_tags.filter(opt => this.tags.indexOf(opt) === -1) 筛选 tags 排除标签**

**add_option_to_tags_Click=>添加 选择 标签=>放入 tags**
****

## 错误解决

**1：
<b-dropdown-item-button
            v-for="option in availableOptions"
            :key="option"
            @click="add_option_to_tags_Click({ option, addTag })"
          >
            {{ option.name }}
          </b-dropdown-item-button>**

**问：配置 name 显示格式 错乱 显示为空？**

**tags 仅支持 一维数组 不支持 指定显示 属性**
****

**2：**
**问：为什么使用 Vuex 不直接 加上 sync await直接异步获取 ？**

**computed：vuex 辅助函数 方法直接使用异步 无效，Vuex 提供了 dispatch 异步 调用函数方法 实现异步调用数据**

****
