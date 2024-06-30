<template>
  <div class="home card">
    <div class="importTable">
      <el-table :data="tableData.slice(0, tableData.length - 1)" border style="width: 100%">
        <el-table-column v-for="(item, index) in tableHeader" :key="index" label="表头" width="180">
          <template #default="scope">
            <div style="display: flex; align-items: center">
              <el-input v-model="scope.row[index]" style="width: 240px" placeholder="" />
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="footer">
      <el-button type="primary" @click="addRow">添加行</el-button>
      <el-button type="primary" @click="addColum">添加列</el-button>
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
      <el-table :data="tableData" style="width: 100%">
        <el-table-column label="表头" width="180">
          <template #default="scope">
            <div style="display: flex; align-items: center">
              <span>{{ scope.row[0] }}</span>
            </div>
          </template>
        </el-table-column>
        <el-table-column v-for="(item, index) in tableHeader.slice(1, tableHeader.length)" :key="index" label="表头" width="300">
          <template #default="scope">
            <div v-if="scope.$index != tableData.length - 1" style="display: flex; align-items: center" class="previewItem">
              <div v-if="strMatch(scope.row[index + 1])" class="line">
                <div class="center" :style="{ left: 10 + 'px' }"></div>
                <div class="dot1" :style="{ left: parseAndSort(scope.row[index + 1])[0] - 2 + 'px' }"></div>
                <div
                  class="line1"
                  :style="{
                    left: parseAndSort(scope.row[index + 1])[0] - 1 + 'px',
                    width: parseAndSort(scope.row[index + 1])[2] - 1 - (parseAndSort(scope.row[index + 1])[0] - 2) + 'px'
                  }"
                ></div>
                <div class="dot2" :style="{ left: parseAndSort(scope.row[index + 1])[1] - 1 + 'px' }"></div>
                <div class="line2"></div>
                <div class="dot3" :style="{ left: parseAndSort(scope.row[index + 1])[2] - 1 + 'px' }"></div>
              </div>
              <span @click="ceshi(scope)">{{ scope.row[index + 1] }}</span>
            </div>
            <div v-else style="display: flex; align-items: center" class="previewItem end">
              <div v-if="strMatch(scope.row[index + 1])" class="line">
                <div class="center" :style="{ left: 10 + 'px' }"></div>
                <div class="dot2" :style="{ left: parseAndSort(scope.row[index + 1])[1] - 2 + 'px' }"></div>
                <div
                  class="line1"
                  :style="{
                    left: parseAndSort(scope.row[index + 1])[0] - 1 + 'px',
                    width: parseAndSort(scope.row[index + 1])[2] - 1 - (parseAndSort(scope.row[index + 1])[0] - 2) + 'px'
                  }"
                ></div>
                <div class="dot1" :style="{ left: parseAndSort(scope.row[index + 1])[0] - 1 + 'px' }"></div>
                <div class="line2"></div>
                <div class="dot3" :style="{ left: parseAndSort(scope.row[index + 1])[2] - 1 + 'px' }"></div>
              </div>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script lang="ts" setup>
import html2canvas from "html2canvas";
import XLSX from "xlsx";
import { ref, computed } from "vue";
const tableHeader = ref(["", "", ""] as any);
const tableData = ref([
  ["femsa", "1.002(1.000, 1.004)", "<0.114"],
  ["ead", "1.002(1.000, 1.004)", "0.111"],
  ["", "1.002(1.000, 1.004)", ""]
] as any);
// const rowNum = computed(() => tableHeader.value.length);
const columnNum: any = computed(() => tableData.value[0].length);
const addRow = () => {
  let row = [];
  for (let index = 0; index < Number(columnNum.value); index++) {
    row.push("");
  }
  tableData.value.splice(tableData.value.length - 1, 0, row);
  console.log(tableHeader.value);
};
const addColum = () => {
  tableHeader.value.push("");
  tableData.value.forEach((item: any) => {
    item.push("");
  });
  console.log(tableHeader.value, tableData.value);
};
const strMatch = (str: any) => {
  if (!!str) {
    return str.match(/([0-9.]+)\(([^)]+)\)/);
  } else {
    return false;
  }
};
const parseAndSort = (str: any) => {
  // 首先找到括号内的内容并分割
  let parts = str.match(/([0-9.]+)\(([^)]+)\)/);
  if (!parts) {
    throw new Error("输入格式不正确");
  }

  // 第一个部分是括号外的数值，第二个部分是括号内的数值
  let mainNumber = parseFloat(parts[1]);
  let innerNumbers = parts[2].split(",").map((num: any) => parseFloat(num.trim()));

  // 将所有数值组合成一个数组
  let allNumbers = [mainNumber, ...innerNumbers];

  // 乘以 1000 再减去 1000
  let adjustedNumbers = allNumbers.map(num => num * 1000 - 1000);

  // 检查是否存在 0，如果有，将每个值加 1
  if (adjustedNumbers.includes(0)) {
    adjustedNumbers = adjustedNumbers.map(num => num + 1);
  }
  adjustedNumbers = adjustedNumbers.map(num => num * 10);
  adjustedNumbers = adjustedNumbers.sort((a, b) => a - b);
  console.log(adjustedNumbers);
  return adjustedNumbers;
};
const ceshi = (scope: any) => {
  console.log(scope);
};

//导入excel
const menuData: any = ref([]);
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
        let obj: any = {
          arr: []
        };
        Object.values(item).map((child: any, index: number) => {
          obj.arr.push("name" + index);
          obj[`name${index}`] = child;
        });
        data.push(obj);
      });
      // tableData.value = data;
      tableData.value = [];
      data.forEach((item: any) => {
        let line: any = [];
        Object.keys(item).forEach(key => {
          console.log(key);
          key != "arr" ? line.push(item[key]) : "";
        });
        tableData.value.push(line);
      });
      console.log("导入excel", data);

      menuData.value = Object.keys(ws[0]);
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
</script>

<style scoped lang="scss">
@import "./index.scss";
.home {
  .importTable {
    :deep(.el-table__header-wrapper) {
      display: none;
    }
    :deep(.el-input) {
      .el-input__wrapper {
        box-shadow: none;
      }
    }
  }
  .preview {
    padding: 20px;
    margin-top: 20px;
    border: 1px solid #666666;
    :deep(.el-table__header-wrapper) {
      display: none;
    }
    :deep(.el-table__inner-wrapper)::before {
      display: none;
    }
    :deep(.el-table__body-wrapper) {
      td {
        padding: 0;
        border: 0;
        .previewItem {
          height: 40px;
          .line {
            position: relative;
            display: flex;
            justify-content: center;
            width: 100px;
            height: 100%;
            .center {
              position: absolute;
              top: 0;
              width: 1px;
              height: 100%;
              background-color: black;
            }
            .dot1,
            .dot2,
            .dot3 {
              position: absolute;
              top: 20px;
              width: 4px;
              height: 4px;
              background-color: black;
              border-radius: 999px;
            }
            .dot2 {
              top: 18px;
              width: 8px;
              height: 8px;
              background-color: #4287fc;
            }
            .line1 {
              position: absolute;
              top: 21.5px;
              height: 1px;
              background-color: #000000;
            }
          }
        }
        .end {
          .line {
            .center {
              position: absolute;
              top: 0;
              width: 1px;
              height: 50%;
              background-color: black;
            }
            .dot2 {
              position: absolute;
              top: 20px;
              width: 4px;
              height: 4px;
              background-color: black;
              border-radius: 999px;
            }
          }
        }
      }
    }
    :deep(.el-input) {
      .el-input__wrapper {
        box-shadow: none;
      }
    }
  }
}
</style>
