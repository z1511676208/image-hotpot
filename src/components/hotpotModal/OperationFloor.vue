<template>
  <el-dialog
    title="编辑热区"
    :visible.sync="dialogVisible"
    top="8vh"
    width="1200px"
    center
    :before-close="closeModal"
    :close-on-click-modal="false"
    @open="openModal"
  >
    <div class="operationFloor">
      <div class="imgBox" @mouseup.left.stop="changeStop()">
        <div ref="container" id="img-box-container" class="container">
          <img
            ref="backgroundImg"
            :src="imgs"
            ondragstart="return false;"
            oncontextmenu="return false;"
            onselect="document.selection.empty();"
            alt="img"
            @mousedown.left.stop="mouseDown($event)"
          />
          <!--draw hotpot-->
          <div
            v-show="caseShow"
            :style="{
              width: areaWidth + 'px',
              height: areaHeight + 'px',
              left: starX + 'px',
              top: starY + 'px'
            }"
            class="area"
          />
          <!--be hotpot-->
          <AreaBox
            v-for="(item, index) in areaData"
            :area-data-index="index"
            :key="'area' + index"
            :link="item.link"
            :title="item.title"
            :type="parseInt(item.type)"
            :area-init.sync="item"
            :parent-width="parentWidth"
            :parent-height="parentHeight"
            @delAreaBox="delAreaBox"
            @addURL="addURL"
          />
        </div>
      </div>
      <!-- 热区链接配置 -->
      <div class="form">
        <div v-for="(item, index) in areaData" :key="index" class="form-row">
          <div class="form-item">
            <el-tag>热区{{ item.number }}</el-tag>
          </div>
          <div class="form-item">
            <label class="label">名称</label
            ><el-input v-model="item.title" :style="linkInputStyle" />
          </div>
          <div class="form-item">
            <label class="label">链接</label
            ><el-input
              v-model.trim="item.link"
              :style="linkInputStyle"
              @blur="onBlur($event, index)"
            />
          </div>
          <i class="el-icon-delete" @click="delAreaBox(index)" />
        </div>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="closeModal"> 取 消 </el-button>
        <el-button type="primary" @click="saveAreaData"> 保 存 </el-button>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import AreaBox from "./AreaBox";
import { checkURL } from "@/utils/validate";

export default {
  name: "OperationFloor",
  components: {
    AreaBox
  },
  props: {
    imgs: {
      type: String,
      default: () => ""
    },
    isHotPot: {
      type: Boolean,
      default: () => false
    },
    imgAreaData: {
      type: Array,
      default: () => []
    },
    linkInputStyle: {
      type: Object,
      default: () => ({
        width: "250px"
      })
    }
  },
  data() {
    return {
      dialogVisible: false,
      starX: 0,
      starY: 0,
      areaWidth: 0,
      areaHeight: 0,
      caseShow: false,
      nowImgWidth: null,
      areaData: [],
      imgNum: 1,
      parentWidth: 0,
      parentHeight: 0
    };
  },
  computed: {},
  watch: {
    imgAreaData(val) {
      this.areaData = [...val];
    }
  },
  mounted() {
    this.areaData = [...this.imgAreaData];
  },
  methods: {
    openModal() {
      this.$nextTick(() => {
        const parentDiv = document.querySelector("#img-box-container");
        //获取元素的宽高
        this.parentWidth = parentDiv.clientWidth;
        this.parentHeight = parentDiv.clientHeight;
        // console.log("this.parentWidth", this.parentWidth, this.parentHeight)
      });
    },
    closeModal() {
      this.$confirm("未保存内容，是否在离开前放弃保存？", "提示信息", {
        distinguishCancelAndClose: true,
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          this.dialogVisible = false;
          this.areaData = [...this.imgAreaData];
        })
        .catch((action) => {
          console.log(action);
        });
    },
    clearList() {
      this.$confirm("确定放弃修改内容嘛？", "提示信息", {
        distinguishCancelAndClose: true,
        confirmButtonText: "确定",
        cancelButtonText: "取消"
      })
        .then(() => {
          this.areaData = this.imgAreaData || [];
          this.dialogVisible = false;
        })
        .catch((action) => {
          console.log(action);
        });

      // this.imgNum = 1;
    },
    onBlur(e, index) {
      console.log("areaData_data", e, this.areaData[index]);
    },
    // 绘画热区开始
    mouseDown(e) {
      e.preventDefault();
      this.caseShow = true;
      // 记录滑动的初始值
      this.starX = e.layerX;
      this.starY = e.layerY;
      // 鼠标滑动的过程
      if (!document.onmousemove) {
        document.onmousemove = (ev) => {
          this.areaWidth = ev.layerX - this.starX;
          this.areaHeight = ev.layerY - this.starY;
        };
      }
    },
    // 绘画热区结束
    changeStop() {
      document.onmousemove = null;
      this.imgNum = this.areaData.length + 1;
      if (this.caseShow && this.areaWidth > 10 && this.areaHeight > 10) {
        const data = {
          number: this.imgNum,
          starX: this.starX,
          starY: this.starY,
          areaWidth: this.areaWidth,
          areaHeight: this.areaHeight,
          nowImgWidth: this.$refs.backgroundImg.width,
          link: "",
          title: "",
          target: "_self"
        };
        this.areaData.push(data);
      }
      // 初始化绘图
      this.caseShow = false;
      this.starX = 0;
      this.starY = 0;
      this.areaWidth = 0;
      this.areaHeight = 0;
      // this.saveAreaData()
    },
    // 删除指定热区
    delAreaBox(index) {
      /* 删除某个热区 */
      this.areaData.splice(index, 1);
      this.$emit("delAreaData", this.areaData);
      /* 删除后 每个热区按顺序重新编号 */
      if (this.areaData) {
        const arr = this.areaData.filter((i) => i.number > index);
        if (!arr) return;
        arr.forEach((i) => i.number--);
        if (this.areaData[this.areaData.length - 1]) {
          this.imgNum = this.areaData[this.areaData.length - 1].number + 1;
        } else {
          this.imgNum = 1;
        }
      }
    },
    // 添加网址
    addURL(index, url, title) {
      let obj = {
        ...this.areaData[index],
        title,
        link: url
      };
      this.$set(this.areaData, index, obj);
      // this.saveAreaData()
    },
    // 保存热区信息
    saveAreaData() {
      if (
        (this.areaData && !this.areaData.length) ||
        !this.checkData(this.areaData)
      ) {
        this.$message.error("热区是否配置链接、是否至少添加一个热区?");
        return;
      }
      this.$emit("saveAreaData", this.areaData);
      this.dialogVisible = false;
      this.$message.success("保存成功!!");
    },
    checkData(list) {
      let isCheck = true;
      list.some((val) => {
        if (!val.link || !checkURL(val.link)) {
          isCheck = false;
        }
      });
      return isCheck;
    }
  }
};
</script>

<style scoped lang="scss" ref="stylesheet/scss">
.btn {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 16px 0;
}
.dialog-footer {
  text-align: right;
  margin-top: 20px;
  margin-right: 20px;
}
.operationFloor {
  position: relative;
  max-height: 75vh;
  overflow-y: auto;
  .header {
    .titleBox {
      display: flex;
      justify-content: space-between;
      align-items: center;
      height: 100px;
      .name {
        font-size: 13px;
        font-weight: bold;
      }
    }
    .textBox {
      font-size: 12px;
      color: #777;
      margin-bottom: 10px;
    }
  }
  .imgBox {
    display: flex;
    justify-content: center;
    .container {
      position: relative;
      overflow: hidden;
    }
    img {
      cursor: crosshair;
      display: block;
    }
    .area {
      position: absolute;
      width: 200px;
      height: 200px;
      left: 200px;
      top: 300px;
      background: rgba(#2980b9, 0.3);
      border: 1px dashed #34495e;
    }
  }
}
.form {
  font-size: 12px;
  width: 100%;
  .form-row {
    display: flex;
    margin: 8px 0;
    align-items: center;
    .form-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      white-space: nowrap;
      margin: 0 5px;
      .label {
        margin-right: 10px;
      }
    }
  }
  .el-icon-delete {
    font-size: 16px;
  }
}
</style>
