# 第一个CRUD

接下来介绍如何开发你的第一个CRUD

::: tip

强烈建议您先从运行demo开始，然后在demo里面写一个crud来感受一下

:::

## 在demo里面开发一个crud

进入到demo目录下`packages/fs-admin/fs-admin-antdv` 或 `packages/demo-element`

一个crud主要分为3个部分

* crud.ts： crud配置
* index.vue： 页面组件
* api.ts： 接口
* mock.ts： 接口mock数据，可选

### 1. crud.ts

crud配置，每个crud最大的不同就在于此文件。     
将如下代码保存为`views/test/myFirstCrud/crud.ts`

```ts
import { CreateCrudOptionsProps, CreateCrudOptionsRet, dict } from "@fast-crud/fast-crud";
import { addRequest, delRequest, editRequest, pageRequest } from "./api";

export default function ({ crudExpose, context }: CreateCrudOptionsProps): CreateCrudOptionsRet {
  return {
    crudOptions: {
      // 自定义crudOptions配置
      request: {
        pageRequest,
        addRequest,
        editRequest,
        delRequest
      },
      //两个字段
      columns: {
        name: {
          title: "姓名",
          type: "text",
          search: { show: true },
          column: {
            resizable: true,
            width: 200
          }
        },
        type: {
          title: "类型",
          type: "dict-select",
          dict: dict({
            data: [
              { value: 1, label: "开始" },
              { value: 0, label: "停止" }
            ]
          })
        }
      }
    }
  };
}


``` 

::: tip    
你可以尝试在 `crudOptions.columns`下添加更多字段
:::

### 2. index.vue

页面组件内容都一样，实际开发中，直接复制，修改一下name即可      
将如下代码保存为`views/test/myFirstCrud/index.vue`

```html
<template>
    <fs-page>
        <fs-crud ref="crudRef" v-bind="crudBinding" />
    </fs-page>
</template>

<script lang="ts">
    import { defineComponent, onMounted } from "vue";
    import { useFs ,OnExposeContext } from "@fast-crud/fast-crud";
    import createCrudOptions from "./crud";

    //此处为组件定义
    export default defineComponent({
        name: "FsCrudFirst",
        setup(props:any,ctx:any) {
            // // crud组件的ref
            // const crudRef = ref();
            // // crud 配置的ref
            // const crudBinding = ref();
            // // 暴露的方法
            // const {crudExpose} = useExpose({crudRef, crudBinding});
            // // 你的crud配置
            // const {crudOptions} = createCrudOptions({crudExpose});
            // // 初始化crud配置
            // const {resetCrudOptions , appendCrudBinding} = useCrud({crudExpose, crudOptions});

            //  =======以上为fs的初始化代码=========
            //  =======你可以简写为下面这一行========
            const context: any = {props,ctx}; // 自定义变量, 将会传递给createCrudOptions, 比如直接把props,和ctx直接传过去使用
            function onExpose(e:OnExposeContext){} //将在createOptions之前触发，可以获取到crudExpose,和context
            const { crudRef, crudBinding, crudExpose } = useFs({ createCrudOptions, onExpose, context});
            
            // 页面打开后获取列表数据
            onMounted(() => {
                crudExpose.doRefresh();
            });
            return {
                crudBinding,
                crudRef
            };
        }
    });
</script>
``` 

::: warning
注意，由于`fs-crud`默认高度为`100%`,所以你必须让`<fs-crud></fs-crud>`的父容器具备高度，或者直接给`fs-crud`本身设置高度
:::

### 3. api.ts

实现添删改查请求接口，实际开发中，复制后修改url即可，你也可以根据实际业务需求增加和修改方法     
将如下代码保存为`views/test/myFirstCrud/api.ts`

```javascript
import { AddReq, DelReq, EditReq, UserPageQuery, UserPageRes } from "@fast-crud/fast-crud";
import _ from "lodash-es";

/**
 * 此处本地方式模拟远程接口，实际开发，你需要替换成你的后台请求
 */
const records = [{ id: 1, name: "Hello World", type: 1 }]
export const pageRequest = async (query: UserPageQuery): Promise<UserPageRes> => {
    return {
        records:_.cloneDeep(records), //此处跟fs所需字段一致，无需转换
        offset: 0, //后续会在transformRes里面做转化，转换为currentPage
        limit: 20,//后续会在transformRes里面做转化，转换为pageSize
        total: records.length
    };
};
export const editRequest = async ({ form, row }: EditReq) => {
    const target = _.find(records, (item) => {
        return row.id === item.id;
    });
    _.merge(target, form);
    return target;
};
export const delRequest = async ({ row }: DelReq) => {
    _.remove(records, (item) => {
        return item.id === row.id;
    });
};
export const addRequest = async ({ form }: AddReq) => {
    const maxRecord = _.maxBy(records, (item) => {
        return item.id;
    });
    form.id = (maxRecord?.id || 0) + 1;
    records.push(form);
    return form;
};

```  
::: tip
此处以本地方式模拟远程接口，实际开发，你需要替换成你的后台请求
:::

:::warning
实际开发过程中，你后台接口返回的数据大概率与fast-crud所需要的数据结构是不一致的，所以你需要配置公共的`request`转化方法，将请求结果转化为`fast-crud`所需要的结构           
具体请参考[request配置](/api/crud-options/request.html)    
:::

### 4. 添加路由和菜单

在`src/router/modules/crud.ts`中增加路由菜单配置

```js
// src/router/modules/crud.ts
export const crudResources = [
    {
        title: "CRUD示例",
        name: "crud",
        path: "/crud",
        redirect: "/crud/basis",
        meta: {
            icon: "ion:apps-sharp"
        },
        children: [
            // ↓↓↓↓↓↓↓↓↓在此位置增加路由配置↓↓↓↓↓↓↓↓↓↓
            {
                title: "myFirstCrud",
                name: "myFirstCrud",
                path: "/test/myFirstCrud",
                component: "/test/myFirstCrud/index.vue"
            },
        ]
    }
]

```

::: tip   
你也可以在`src/router/resource/modules`目录中仿照`crud.ts`新增你自己的顶级菜单,将其加入`framework.ts`的`children`中即可。
:::

### 6. 看看效果

启动demo，访问`myFirstCrud`查看效果

恭喜，你已经完成了第一个crud功能的开发

## 实际项目中开发CRUD

在实际项目开发中，通常在`示例`的`views`中找一个合适的crud复制到你的项目的`views`，然后再根据需求修改即可    
::: warning
注意：需要修改api.ts的`requestForMock`为`request`，这样才会去调用你的真实后端接口
:::
