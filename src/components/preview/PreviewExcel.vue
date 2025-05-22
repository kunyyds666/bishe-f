<template>
    <div v-html="excelContent" class="table-info"></div>
</template>

<script setup>
import * as XLSX from "xlsx";
import { ref, reactive, getCurrentInstance, nextTick, onMounted } from "vue";
const { proxy } = getCurrentInstance();

const props = defineProps({
    url: {
        type: String,
    },
});

const excelContent = ref();
const initExcel = async () => {
    let result = await proxy.Request({
        url: props.url,
        responseType: "arraybuffer",
    });
    if (!result) {
        return;
    }

    let workbook = XLSX.read(new Uint8Array(result), { type: "array" });
    let worksheet = workbook.Sheets[workbook.SheetNames[0]];
    let rawHtml = XLSX.utils.sheet_to_html(worksheet);

    // 创建临时 DOM 解析
    let tempDom = document.createElement("div");
    tempDom.innerHTML = rawHtml;
    let tds = tempDom.querySelectorAll("td");

    // 为每个 td 添加 title 属性
    tds.forEach(td => {
        if (td.textContent) {
            td.setAttribute("title", td.textContent.trim());
        }
    });

    excelContent.value = tempDom.innerHTML;
};

onMounted(() => {
    initExcel();
});
</script>

<style lang="scss" scoped>
.table-info {
    width: 100%;
    padding: 10px;
    overflow: auto;

    :deep table {
        width: 100%;
        border-collapse: collapse;
        table-layout: fixed; // 重点：表格列宽固定，便于省略效果生效
        td {
            border: 1px solid #ddd;
            padding: 5px;
            height: 30px;
            min-width: 50px;

            // 添加省略样式
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
    }
}
</style>
