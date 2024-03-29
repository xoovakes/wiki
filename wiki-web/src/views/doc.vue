<template>
  <a-layout class="box">
    <a-layout-content class="border">
      <div class="noEbook" v-if="parentCate.length === 0">
        该电子书为空！将在
        <span class="counter">{{ counter }}</span>
        秒后自动回到主页~
      </div>

      <a-layout v-if="parentCate.length !== 0">
        <a-layout-sider style="background: #fff" width="250px">
          <el-row>
            <el-col :span="8">
              <a-tree
                v-if="parentCate.length > 0"
                :tree-data="parentCate"
                @select="onSelect"
                :replaceFields="{ title: 'name', key: 'id', value: 'id' }"
                :defaultExpandAll="true"
                :defaultSelectedKeys="defaultSelectedKeys"
              >
              </a-tree>
            </el-col>
          </el-row>
        </a-layout-sider>

        <a-layout-content style="background: #fff">
          <a-row>
            <a-col :span="1">
              <a-divider
                type="vertical"
                style="height: 100%; width: 2px; background-color: #f0f0f0"
              />
            </a-col>
            <a-col :span="23">
              <div>
                <h2>{{ doc.name }}</h2>
                <div>
                  <span>阅读数：{{ doc.viewCount }}</span> &nbsp; &nbsp;
                  <span>点赞数：{{ doc.voteCount }}</span>
                </div>
                <a-divider style="height: 2px; background-color: #f0f0f0" />
              </div>
              <div class="wangeditor" :innerHTML="html"></div>
              <div class="vote-div">
                <a-button
                  type="primary"
                  shape="round"
                  :size="'large'"
                  @click="vote"
                >
                  <template #icon
                    ><LikeOutlined /> &nbsp;点赞：{{ doc.voteCount }}
                  </template>
                </a-button>
              </div>
            </a-col>
          </a-row>
        </a-layout-content>
      </a-layout>
    </a-layout-content>
  </a-layout>
</template>

<script lang="ts" setup>
import { onMounted, ref } from "vue";
import axios from "axios";
import { message } from "ant-design-vue";
import { Tool } from "@/util/tool";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const docs = ref();
const html = ref();

const defaultSelectedKeys = ref<any[]>([]);

// 当前选中的文档
const doc = ref();

const parentCate = ref<any[]>([]); // 一级文档树

/**
 * 内容查询
 **/
const handleQueryContent = async (id: number) => {
  const res = await axios.get("/doc/find-content/" + id);
  const data = res.data;
  if (data.success) {
    html.value = data.content;
  } else {
    message.error(data.message);
  }
};

/**
 * 数据查询
 **/

const router = useRouter();
const counter = ref(5);
let timer: any;
const handleQuery = async () => {
  const res = await axios.get("/doc/all/" + route.query.ebookId);
  const data = res.data;
  if (data.success) {
    docs.value = data.content;

    parentCate.value = [];
    parentCate.value = Tool.array2Tree(docs.value, 0);

    if (Tool.isEmpty(parentCate.value)) {
      timer = setInterval(() => {
        counter.value = counter.value - 1;
        if (counter.value <= 0) {
          clearInterval(timer);
          timer = null;
          router.push({ path: "/" });
        }
      }, 1000);
    }

    if (Tool.isNotEmpty(parentCate.value)) {
      // defaultSelectedKeys.value = [parentCate.value[0].id];
      handleQueryContent(parentCate.value[0].id);
      // 初始显示文档信息
      doc.value = parentCate.value[0];
    }
  } else {
    message.error(data.message);
  }
};

const onSelect = (selectedKeys: any, info: any) => {
  console.log("selected", selectedKeys, info);
  if (Tool.isNotEmpty(selectedKeys)) {
    // 选中某一节点时，加载该节点的文档信息
    doc.value = info.selectedNodes[0].props;
    // 加载内容
    handleQueryContent(selectedKeys[0]);
  }
};

// 点赞
const vote = async () => {
  const res = await axios.get("/doc/vote/" + doc.value.id);
  const data = res.data;
  if (data.success) {
    doc.value.voteCount++;
  } else {
    message.error(data.message);
  }
};

onMounted(() => {
  handleQuery();
});
</script>

<style>
/* wangeditor默认样式, 参照: http://www.wangeditor.com/doc/pages/02-%E5%86%85%E5%AE%B9%E5%A4%84%E7%90%86/03-%E8%8E%B7%E5%8F%96html.html */
/* table 样式 */
.wangeditor table {
  border-top: 1px solid #ccc;
  border-left: 1px solid #ccc;
}
.wangeditor table td,
.wangeditor table th {
  border-bottom: 1px solid #ccc;
  border-right: 1px solid #ccc;
  padding: 3px 5px;
}
.wangeditor table th {
  border-bottom: 2px solid #ccc;
  text-align: center;
}

/* blockquote 样式 */
.wangeditor blockquote {
  display: block;
  border-left: 8px solid #d0e5f2;
  padding: 5px 10px;
  margin: 10px 0;
  line-height: 1.4;
  font-size: 100%;
  background-color: #f1f1f1;
}

/* code 样式 */
.wangeditor code {
  display: inline-block;
  *display: inline;
  *zoom: 1;
  background-color: #f1f1f1;
  border-radius: 3px;
  padding: 3px 5px;
  margin: 0 3px;
}
.wangeditor pre code {
  display: block;
}

/* ul ol 样式 */
.wangeditor ul,
ol {
  margin: 10px 0 10px 20px;
}

/* 和antdv p冲突，覆盖掉 */
.wangeditor blockquote p {
  font-family: "YouYuan";
  margin: 20px 10px !important;
  font-size: 16px !important;
  font-weight: 600;
}

/* 点赞 */
.vote-div {
  padding: 15px;
  text-align: center;
}

/* 图片自适应 */
.wangeditor img {
  max-width: 100%;
  height: auto;
}

/* 视频自适应 */
.wangeditor iframe {
  width: 100%;
  height: 400px;
}
.noEbook {
  height: 90%;
  width: 100%;
  /* opacity: 0.7; */
  text-align: center;
  line-height: 130px;
  font-size: 24px;
  background-image: url("../assets/noEbook.png");
  background-size: 50%;
  background-position: 50% 30%;
  background-repeat: no-repeat;
  max-height: "530px";
}
.counter {
  font-weight: 600;
  color: #ff0000;
}
</style>
