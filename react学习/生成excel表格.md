1. **安装相关库**

```
npm install xlsx

npm install js-export-excel
```

2. **由于 ts 不能直接找到 js 模块，所以在根目录下添加一个名为 export.d.ts 的文件**

```tsx
declare module "js-export-excel";
```

3. **封装导出 excel 文件组件**

```tsx
import ExportJsonExcel from "js-export-excel";

const ExportFile = (dataSource) => {
  const option = {
    fileName: "demo表",
    datas: [
      {
        sheetData: dataSource,
        sheetName: "demo",
        sheetFilter: ["test1", "test2", "test3"],
        sheetHeader: ["测试1", "测试2", "测试3"],
      },
    ],
  };
  const toExcel = new ExportJsonExcel(option);
  toExcel.saveExcel();
};

export default ExportFile;
```
