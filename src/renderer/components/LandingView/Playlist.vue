<template>
    <div class="controller">
        <div class="playlist"
             v-if="hasRecentPlaylist">
            <div class="item shadow"
                 v-for="(item, index) in lastPlayedFile"
                 :id="'item'+index"
                 :key="item.path"
                 :style="{
              backgroundImage: itemShortcut(item.shortCut),
              width: item.chosen ? '140px' : '114px',
              height: item.chosen ? '80px' : '65px',
            }"
                 @click.stop="onRecentItemClick(item)"
                 @mouseover="onRecentItemMouseover(item, index)"
                 @mouseout="onRecentItemMouseout(index)"
                 @mousedown.stop.capture="onRecentItemMousedown($event, index)">
            </div>
        </div>
    </div>
</template>

<script>
import path from 'path';
import asyncStorage from '@/helpers/asyncStorage';
export default {
  name: 'playlist',
  data() {
    return {
      imageTurn: '',
      isTurnToOdd: false,
      backgroundUrlOdd: '',
      backgroundUrlEven: '',
      showShortcutImage: false,
      langdingLogoAppear: true,
      displayInfo: [],
      mouseDown: false,
      isDragging: false,
      disX: '',
      disY: '',
      recentFileDel: false,
    };
  },
  props: {
    lastPlayedFile: {
      type: Object.Array,
      require: true,
      default: [],
    },
  },
  mounted() {
  },
  computed: {
    hasRecentPlaylist() {
      return this.lastPlayedFile.length > 0;
    },
  },
  methods: {
    backgroundUrl() {
      switch (this.imageTurn) {
        case 'odd': return this.backgroundUrlOdd;
        case 'even': return this.backgroundUrlEven;
        default: return '';
      }
    },
    itemShortcut(shortCut) {
      return `url("${shortCut}")`;
    },
    itemInfo() {
      return {
        baseName: path.basename(this.item.path, path.extname(this.item.path)),
        lastTime: this.item.lastPlayedTime,
        duration: this.item.duration,
        percentage: (this.item.lastPlayedTime / this.item.duration) * 100,
      };
    },
    onRecentItemMouseover(item, index) {
      this.item = item;
      this.$set(this.lastPlayedFile[index], 'chosen', true);
      if (item.shortCut !== '') {
        this.isChanging = true;
        this.isTurnToOdd = !this.isTurnToOdd;
        if (this.isTurnToOdd) {
          this.imageTurn = 'odd';
          this.backgroundUrlOdd = item.shortCut;
        } else {
          this.imageTurn = 'even';
          this.backgroundUrlEven = item.shortCut;
        }
        this.langdingLogoAppear = false;
        this.showShortcutImage = true;
      } else {
        this.langdingLogoAppear = true;
        this.showShortcutImage = false;
      }
      this.displayInfo.langdingLogoAppear = this.langdingLogoAppear;
      this.displayInfo.showShortcutImage = this.showShortcutImage;
      this.displayInfo.imageTurn = this.imageTurn;
      this.displayInfo.backgroundUrl = this.backgroundUrl();
      this.displayInfo.baseName = this.itemInfo().baseName;
      this.displayInfo.lastTime = this.itemInfo().lastTime;
      this.displayInfo.duration = this.itemInfo().duration;
      this.displayInfo.percentage = this.itemInfo().percentage;
      this.$bus.$emit('displayInfo', this.displayInfo);
    },
    onRecentItemMouseout(index) {
      this.$set(this.lastPlayedFile[index], 'chosen', false);
    },
    onRecentItemMousedown(ev, index) {
      this.mouseDown = true;
      this.isDragging = false;
      this.disX = ev.clientX;
      this.disY = ev.clientY;
      const vm = this;
      const item = document.querySelector(`#item${index}`);
      item.style.zIndex = 5;
      function mousemove(ev) {
        if (vm.mouseDown) {
          vm.isDragging = true;
          const l = ev.clientX - vm.disX;
          const t = ev.clientY - vm.disY;
          item.style.left = `${l}px`;
          item.style.top = `${t}px`;
          if (l <= -140 || l >= 140 || t >= 80 || t <= -80) {
            vm.recentFileDel = true;
          } else {
            vm.recentFileDel = false;
          }
        }
      }
      function mouseup() {
        vm.mouseDown = false;
        if (vm.recentFileDel) {
          vm.displayInfo.langdingLogoAppear = true;
          vm.displayInfo.showShortcutImage = false;
          vm.$bus.$emit('displayInfo', vm.displayInfo);
          vm.lastPlayedFile.splice(index, 1);
          asyncStorage.set('recent-played', vm.lastPlayedFile);
          vm.recentFileDel = false;
        } else {
          item.style.zIndex = 4;
          item.style.left = '';
          item.style.top = '';
        }
      }
      document.onmousemove = mousemove;
      document.querySelector('main').onmouseup = mouseup;
      document.onmouseup = mouseup;
    },
    onRecentItemClick(item) {
      if (!this.isDragging) {
        this.openFile(item.path);
      }
    },
  },
};
</script>

<style lang="scss">
    .controller {
        position: absolute;
        left: 0;
        bottom: 40px;
        width: 100%;
        z-index: 4;

        .playlist {
            -webkit-app-region: no-drag;
            display: flex;
            flex-direction: row;
            justify-content: flex-start;
            align-items: flex-end;
            margin-left: 45px;

            .item {
                color: #e4e4c4;
                border-radius: 2px;
                width: 114px;
                height: 65px;
                color: gray;
                cursor: pointer;
                margin-right: 15px;
                background-size: cover;
                background-color: black;
                background-repeat: no-repeat;
                background-position: center center;
                transition: width 150ms ease-out, height 150ms ease-out;
            }

            .shadow {
                position: relative;
                box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3), 0 0 20px rgba(0, 0, 0, 0.1) inset;
            }
            .shadow:before, .shadow:after {
                content: "";
                position: absolute;
                z-index: -1;
                box-shadow: 0 8px 30px rgba(0,0,0,0.3);
                top: 50%;
                bottom: 0;
                left: 10px;
                right: 10px;
                border-radius: 50px;
            }
            .shadow:after {
                right: 10px;
                left: auto;
                transform: skew(8deg) rotate(3deg);
            }
        }
    }

</style>
