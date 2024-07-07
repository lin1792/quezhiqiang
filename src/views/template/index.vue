<template>
  <div class="home card">
    <div class="bts">
      <!-- <img src="../../assets/icons/xianxinglvhangriji.svg" alt="" /> -->
      <div>
        <el-button type="primary" @click="exportAction">生成图片</el-button>
      </div>
      <div>
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
          <span>excel格式：第一行为表头</span>
        </el-upload>
      </div>
      <div>
        <span>自定义点:</span>
        <input type="file" id="fileInput" accept="image/*" />
      </div>
      <div>
        <span>Y轴位置:</span>
        <el-input v-model="otherParams.yAxis" type="number" style="width: 100px" placeholder="Please input" />
      </div>
      <div>
        <span>x轴宽度:</span>
        <el-input v-model="otherParams.xAxisWidth" type="number" style="width: 100px" placeholder="Please input" />
      </div>
    </div>
    <div id="exportAll" class="preview">
      <div v-for="(item, index1) in tableData" :key="index1">
        <div v-if="item.type != 'line'" class="data">
          <div class="dataItem">{{ item.header }}</div>
          <div class="dataItems">
            <div v-for="(data, index) in item.data" :key="index" class="dataItem">
              <span>{{ data }}</span>
            </div>
          </div>
        </div>
        <div class="data" v-else>
          <ForestPlot ref="ForestPlotRef" :picData="item" :otherParams="otherParams" />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import html2canvas from "html2canvas";
import XLSX from "xlsx";
import ForestPlot from "./components/ForestPlot.vue";
import { ref, onMounted } from "vue";
import { generateUUID } from "@/utils";
const tableData = ref([
  // { header: "23", data: [], type: "data" },
  // { header: "223", data: ["1.002(1.000, 1.004)", "1.002(1.000, 1.004)"], type: "line" },
  // { header: "233", data: [], type: "data" }
] as any);
const ForestPlotRef = ref(null as any);
let otherParams = ref({
  dotImg: "",
  yAxis: 1,
  xAxisWidth: 200
} as any);
const create = () => {
  // console.log(tableData.value);
  // ForestPlotRef.value.data = [...tableData.value];
  // ForestPlotRef.value.drawChart();
};
function handleFileSelect(event: any) {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = function (e: any) {
      console.log(e.target.result);
      otherParams.value.dotImg = e.target.result;
    };
    reader.readAsDataURL(file);
  }
}

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
      let data: any = [];
      let header: any = [];
      let lineKeys: any = [];
      Object.keys(ws[0]).forEach((key: any) => {
        header.push(key);
        const isLine = ws.find((item: any) => {
          return item[key].match(/([0-9.]+)\(([^)]+)\)/);
        });
        if (!!isLine) {
          console.log(key);
          lineKeys.push(key);
          data.push({ header: key, data: [], type: "line" });
          data.push({ header: key, data: [], type: "data" });
        } else {
          data.push({ header: key, data: [], type: "data" });
        }
      });
      console.log(data);
      ws.forEach((item: any) => {
        data.forEach((dataItem: any) => {
          if (dataItem.type != "line") {
            dataItem.data.push(item[dataItem.header]);
          } else {
            let parts = item[dataItem.header].match(/([0-9.]+)\(([^)]+)\)/);
            let dots: any = [];
            if (!!parts) {
              // 第一个部分是括号外的数值，第二个部分是括号内的数值
              let mainNumber = parseFloat(parts[1]);
              let innerNumbers = parts[2].split(",").map((num: any) => parseFloat(num.trim()));
              // 将所有数值组合成一个数组
              dots = [mainNumber, ...innerNumbers];
              // series[0].pointEstimate = dots[0]; //中点
              // series[0].ci = [dots[1], dots[2]]; //区间
            } else {
            }
            dataItem.data.push({ id: generateUUID(), ci: [dots[1], dots[2]], pointEstimate: dots[0] });
          }
          // return dataItem;
        });
        // data.push({header:})
      });

      // ws.forEach((item: any) => {
      //   console.log(item);
      // const id = generateUUID();
      // const label = item.character;
      // let series = [{}] as any;
      // series[0].pointEstimate = 0;
      // series[0].ci = [0, 0];
      // Object.keys(item).forEach((key: any) => {
      //   let parts = item[key].match(/([0-9.]+)\(([^)]+)\)/);
      //   if (!parts) {
      //     // throw new Error("输入格式不正确");
      //     series[0][key] = item[key];
      //   } else {
      //     let dots: any = [];
      //     // 第一个部分是括号外的数值，第二个部分是括号内的数值
      //     let parts = item[key].match(/([0-9.]+)\(([^)]+)\)/);
      //     let mainNumber = parseFloat(parts[1]);
      //     let innerNumbers = parts[2].split(",").map((num: any) => parseFloat(num.trim()));
      //     // 将所有数值组合成一个数组
      //     dots = [mainNumber, ...innerNumbers];
      //     series[0].pointEstimate = dots[0];
      //     series[0].ci = [dots[1], dots[2]];
      //   }
      // });
      // console.log(series[0]);

      // data.push({
      //   id: id,
      //   label: label,
      //   series: series
      // });
      // });
      tableData.value = data;
      console.log("导入excel", tableData.value);
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

onMounted(() => {
  document.getElementById("fileInput")!.addEventListener("change", handleFileSelect, false);
});
</script>

<style scoped lang="scss">
@import "./index.scss";
.home {
  .bts {
    display: flex;
    align-items: center;
    & > div {
      margin-right: 20px;
    }
    .el-button {
      height: 100%;
      margin-right: 20px;
    }
  }
  .preview {
    display: flex;
    .data {
      display: flex;
      flex-direction: column;
      height: 100%;
      margin: 0 10px;
      .dataItems {
        display: flex;
        flex-direction: column;
        align-items: center;
        height: calc(100% - 40px);
        & > .dataItem {
          display: flex;
          align-items: center;
          height: 20px;
          font-size: 12px;
        }
      }
      & > .dataItem {
        display: flex;
        align-items: center;
        height: 40px;
        line-height: 1;
        span {
          display: inline-block;
        }
      }
    }
  }
}
</style>
