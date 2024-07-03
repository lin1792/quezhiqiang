<template>
  <div class="home card">
    <div class="footer">
      <el-button type="primary" @click="exportAction">生成图</el-button>
      <el-upload
        class="upload-demo"
        ref="upload"
        accept=".xls,.xlsx"
        action=""
        :on-change="uploadExcel"
        :show-file-list="false"
        :auto-upload="false"
      >
        <el-button size="large">导入excel</el-button>
      </el-upload>
    </div>
    <div id="exportAll" class="preview">
      <ForestPlot ref="ForestPlotRef" />
    </div>
  </div>
</template>

<script lang="ts" setup>
import html2canvas from "html2canvas";
import XLSX from "xlsx";
import ForestPlot from "./components/ForestPlot.vue";
import { ref, onMounted } from "vue";
const tableData = ref([
  ["femsa", "1.002(1.000, 1.004)", "<0.114"],
  ["ead", "1.002(1.000, 1.004)", "0.111"],
  ["", "1.002(1.000, 1.004)", ""]
] as any);
const ForestPlotRef = ref(null as any);
const create = () => {
  console.log(tableData.value);
  ForestPlotRef.value.data = [...tableData.value];
  ForestPlotRef.value.drawChart();
};
//导入excel
// const tableData: any = ref([]);
const uploadExcel = (file: any, fileList: any) => {
  console.log(fileList);

  let files = { 0: file.raw };
  const fileReader = new FileReader();
  fileReader.onload = (e: any) => {
    try {
      const workbook = XLSX.read(e.target.result, {
        type: "binary"
      });
      const wsname = workbook.SheetNames[0]; // 取第一张表
      const ws: any = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); // 生成json表格内容
      const data: any = [];
      ws.map((item: any) => {
        console.log(item);

        // let obj: any = {};
        // Object.values(item).map((child: any, index: number) => {
        //   obj[`name${index}`] = child;
        // });

        let dots: any = [];
        if (item["95% CI"]) {
          // 第一个部分是括号外的数值，第二个部分是括号内的数值
          let parts = item["95% CI"].match(/([0-9.]+)\(([^)]+)\)/);
          let mainNumber = parseFloat(parts[1]);
          let innerNumbers = parts[2].split(",").map((num: any) => parseFloat(num.trim()));
          // 将所有数值组合成一个数组
          dots = [mainNumber, ...innerNumbers];
        }

        data.push({ label: item.character, pointEstimate: dots[0], ci: [dots[1], dots[2]], p: item.p });
      });
      tableData.value = data;
      console.log("导入excel", data);
      create();
    } catch (err) {
      console.log(err);
    }
  };
  fileReader.readAsBinaryString(files[0]);
};

//导出图片
// 导出 -> 这个是按钮的导出按钮的触发事件
const exportAction = () => {
  download();
};
// 下载
const download = () => {
  let targetDom: any = document.getElementById("exportAll"); //原本需要截图的div
  // console.log("🚀 ~ file: index.vue:33 ~ download ~ targetDom:", targetDom.clientWidth);
  let clonedNode = targetDom.cloneNode(true); //复制一个
  clonedNode.setAttribute("style", `width: ${targetDom.clientHeight};height: ${targetDom.clientWidth};`);
  document.body.appendChild(clonedNode); //放到body后面
  // 转换成canvas
  html2canvas(targetDom, {
    allowTaint: true,
    taintTest: false
  } as any).then(function (canvas) {
    let pageData = canvas.toDataURL("image/png", 1.0);
    saveFile(pageData.replace("image/png", "image/octet-stream"), new Date().getTime() + ".png");
    document.body.removeChild(clonedNode);
  });
};
// 保存路径下载
const saveFile = (data: any, filename: any) => {
  let save_link: any = document.createElementNS("http://www.w3.org/1999/xhtml", "a");
  save_link.href = data;
  save_link.download = filename;
  document.body.appendChild(save_link);
  save_link.click();
  save_link.remove();
};

onMounted(() => {});
</script>

<style scoped lang="scss">
@import "./index.scss";
.home {
  padding: 20px;
  margin-top: 20px;
}
</style>
./components/ForestPlot.vue
