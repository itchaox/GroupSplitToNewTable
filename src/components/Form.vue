<!--
 * @Version    : v1.00
 * @Author     : itchaox
 * @Date       : 2023-12-23 09:34
 * @LastAuthor : itchaox
 * @LastTime   : 2024-01-01 01:28
 * @desc       : 
-->

<script setup lang="ts">
  import { bitable } from '@lark-base-open/js-sdk';
  import { useTable } from '@/hooks/useTable';

  import { Close } from '@element-plus/icons-vue';

  const base = bitable.base;

  let table;
  async function getTable() {
    table = await bitable.base.getActiveTable();
  }

  const list = [
    {
      num: 1,
      option: 'A',
      title: 'a',
    },

    {
      num: 5,
      option: 'A',

      title: 'c',
    },
    {
      num: 2,

      option: 'B',

      title: 'e',
    },
    {
      num: 3,

      option: 'B',

      title: '韩子',
    },
    {
      num: 4,
      option: 'C',

      title: '安家',
    },
    {
      num: 4,
      option: 'E',

      title: '安家',
    },
    {
      num: 4,
      option: 'F',

      title: '安家',
    },
  ];

  let view;
  async function init() {
    table = await base.getActiveTable();
    view = await table.getActiveView();
    groupFieldList.value = await view.getFieldMetaList();
  }

  onMounted(async () => {
    // await getTable();
    // const { records } = await table.getRecords({
    //   pageSize: 5000,
    // });
    // // FIXME 从这个打印位置直接复制
    // console.log('🚀  records:', records);
    await init();
  });

  const collapse = ref('1');

  // 分组的字段列表
  const groupFieldList = ref([]);

  // FIXME 分组
  const groupList = ref([]);

  const addGroup = () => {
    const idsInGroup = groupList.value.map((item) => item.id);
    const _list = groupFieldList.value.filter((item) => !idsInGroup.includes(item.id));

    if (_list?.[0]) {
      groupList.value.push({
        id: _list?.[0]?.id,
        name: _list?.[0]?.name,
        type: _list?.[0]?.type,
        desc: false,
      });
    }

    console.log('🚀  groupList:', groupList.value);
  };

  const groupFiledChange = async (item, index) => {
    let _activeItem = groupFieldList.value.find((i) => i.id === item.id);

    groupList.value[index] = {
      name: _activeItem.name,
      type: _activeItem.type,
      id: _activeItem.id,
      desc: false,
    };
  };

  /**
   * @desc  : 顺序 / 倒序
   * @param  {any} type：字段类型
   * @return {any} 文本
   */
  const getGroupText = (type, isOrder) => {
    let textList = [1, 11, 13, 15, 17, 18, 19, 20, 21, 22, 23, 403, 1003, 1004, 99001];
    let numberList = [2, 5, 1001, 1002, 1005, 99002, 99003, 99004];
    let optionList = [3, 4, 7];

    let _text;
    if (textList.includes(type)) {
      _text = isOrder ? 'A → Z' : 'Z → A';
    } else if (numberList.includes(type)) {
      _text = isOrder ? '0 → 9' : '9 → 0';
    } else if (optionList.includes(type)) {
      _text = isOrder ? '顺序' : '倒序';
    } else {
      _text = isOrder ? 'A → Z' : 'Z → A';
    }

    return _text;
  };

  function start() {}
</script>

<template>
  <div class="home">
    <!-- 折叠面板 -->
    <el-collapse
      v-model="collapse"
      class="collapse"
    >
      <!-- FIXME 分组 -->
      <el-collapse-item name="1">
        <template #title>
          <el-icon><SetUp /></el-icon>
          <span class="collapse-title">设置分组条件</span>
          <span
            v-if="groupList.length > 0"
            style="color: #dd742f"
            >（{{ groupList.length }}）</span
          >
        </template>
        <div class="collapse-line-list">
          <div
            class="collapse-line"
            v-for="(item, index) in groupList"
            :key="item.id"
          >
            <!-- 字段名 -->
            <div class="collapse-line-filed">
              <el-select
                size="small"
                v-model="item.id"
                :title="item.name"
                @change="groupFiledChange(item, index)"
              >
                <el-option
                  v-for="field in groupFieldList"
                  :key="field.id"
                  :label="field.name"
                  :title="field.name"
                  :value="field.id"
                  :disabled="groupList.map((j) => j.id).includes(field.id)"
                >
                  <field-icon :fieldType="field.type" />
                  <span>
                    {{ field.name }}
                  </span>
                </el-option>
              </el-select>
            </div>
            <div class="collapse-line-other">
              <!-- 值 -->
              <div class="collapse-line-value">
                <el-button-group size="small">
                  <el-button
                    type="primary"
                    :plain="item.desc"
                    @click="() => (item.desc = !item.desc)"
                    >{{ getGroupText(item.type, true) }}</el-button
                  >
                  <el-button
                    type="primary"
                    :plain="!item.desc"
                    @click="() => (item.desc = !item.desc)"
                  >
                    {{ getGroupText(item.type, false) }}
                  </el-button>
                </el-button-group>
              </div>
              <el-button
                :icon="Close"
                class="collapse-delete"
                @click="() => groupList.splice(index, 1)"
                text
              />
            </div>
          </div>
        </div>
        <el-button
          v-if="groupList.length < 3"
          text
          @click="addGroup"
        >
          <el-icon><Plus /></el-icon>添加条件
        </el-button>
      </el-collapse-item>
    </el-collapse>

    <!-- 分组后的名字 -->

    <!-- 操作按钮 -->
    <el-button
      type="primary"
      @click="start"
      >开始拆分</el-button
    >
  </div>
</template>

<style scoped>
  .home {
    font-size: 14px;
  }

  .collapse {
    margin: 20px 0;
  }

  .collapse-title {
    margin-left: 5px;
  }

  .collapse-line-list {
    margin: 10px 0;
  }

  .collapse-line {
    display: flex;
    flex-wrap: nowrap;
    justify-content: space-between;
    margin-bottom: 10px;
    height: 24px;
  }

  .collapse-line-filed {
    margin-right: 10px;
  }

  .collapse-line-other {
    display: flex;
    justify-content: flex-end;
  }

  .collapse-line-value {
    margin: 0 5px;
  }

  .collapse-delete {
    padding: 5px;
    height: 24px;
  }
</style>
