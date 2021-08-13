<template>
  <div class="layout">
    <div class="top" />
    <div class="left" />
    <div class="center">
      <OperationFloor ref="operationFloor" />
    </div>
    <div class="right">
      <div class="save" @click="saveHotSpotInfo">保存</div>
      <div class="preview" @click="$router.push('/preview')">预览</div>
      <div class="preview" @click="clearHotSpotInfo">清空所有</div>
      <div class="preview" @click="openFloorModal">打开编辑弹窗</div>
    </div>
    <OperationFloorModal
      ref="hotpot"
      :imgs="imgUrl"
      :img-area-data="imgAreaData"
      @delAreaData="handleAreaData"
      @saveAreaData="handleAreaData"
    ></OperationFloorModal>
  </div>
</template>

<script>
import OperationFloor from "@/components/HotspotImage/Main/OperationFloor";
import OperationFloorModal from "@/components/hotpotModal/OperationFloor";
import imgUrl from "@/assets/hotAreaImage.png";
export default {
  name: "Layout",
  components: {
    OperationFloor,
    OperationFloorModal
  },
  data() {
    return {
      imgUrl,
      imgAreaData: []
    };
  },
  methods: {
    saveHotSpotInfo() {
      this.$refs.operationFloor.saveAreaData();
    },
    clearHotSpotInfo() {
      this.$refs.operationFloor.clearHotSpotInfo();
    },
    openFloorModal() {
      this.$refs.hotpot.dialogVisible = true;
    },
    handleAreaData(areaData) {
      console.log("保存的数据", areaData);
    }
  }
};
</script>

<style scoped lang="scss" ref="stylesheet/scss">
.layout {
  width: 100%;
  background: pink;
  display: grid;
  grid-template-columns: 1fr 1000px 1fr;
  background: linear-gradient(90deg, #83a4d4, #b6fbff);
  .top {
    grid-column-start: 1;
    grid-column-end: 4;
    height: 70px;
  }
  .right {
    place-self: start center;
    > div {
      width: 100px;
      height: 40px;
      color: #eeeeee;
      border-radius: 3px;
      text-align: center;
      line-height: 40px;
      background: rgba(#3a7bd5, 0.6);
      margin: 10px;
      box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
    }
    > div:hover {
      opacity: 0.9;
    }
  }
}
</style>
