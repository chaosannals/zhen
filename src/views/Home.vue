<template>
  <div class="home">
    <div>
      <img v-if="all.length <= 0" alt="Vue logo" src="@/assets/logo.png" />
    </div>
    <div>
      <el-input v-model="plain" class="search-input" />
    </div>
    <div class="content">
      <el-table :data="group" height="100%">
        <el-table-column prop="word" label="单词" />
        <el-table-column prop="definition" label="释义" />
      </el-table>
    </div>
    <el-pagination
      layout="prev, pager, next, jumper, ->, sizes, total"
      :pager-count="5"
      :current-page.sync="page"
      :page-size.sync="size"
      :page-sizes="[50, 100, 200, 300]"
      :total="all.length"
    />
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import Component from "vue-class-component";
import { ipcRenderer as ipc } from "electron";

@Component
export default class Home extends Vue {
  page: number = 1;
  size: number = 50;
  plain: string = "";
  words: Array<any> = [];

  get group(): Array<any> {
    let count = Math.floor(this.all.length / this.size);
    let tail = this.all.length % this.size;
    let start = (this.page - 1) * this.size;
    let end = start + (this.page >= count ? tail : this.size);
    return this.all.slice(start, end);
  }

  get all(): Array<any> {
    if (this.plain.length <= 0) return [];
    return this.words.filter(
      i =>
        i.word.indexOf(this.plain) >= 0 || i.definition.indexOf(this.plain) >= 0
    );
  }

  created() {
    ipc.send("load");
    ipc.on("load", (event, data: any) => {
      let keys = Object.keys(data);
      this.words = keys.map(i => {
        return {
          word: i,
          definition: data[i]
        };
      });
    });
  }
}
</script>

<style lang="scss" scoped>
.home {
  display: flex;
  flex-flow: column;
  position: relative;
  width: 100%;
  height: 100%;

  .search-input {
    max-width: 20em;
  }

  .content {
    overflow: hidden;
    position: relative;
    flex-grow: 1;
  }
}
</style>