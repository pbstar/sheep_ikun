<template>
  <div class="app">
    <div class="aicon2" v-if="!ishome">
      <van-icon name="revoke" color="#fff" size="25px" @click="tohome" />
    </div>
    <div class="aicon">
      <img
        v-show="ismuted"
        @click="ismuted = false"
        src="./assets/image/icon/close_audio.png"
        alt=""
      />
      <img
        v-show="!ismuted"
        @click="ismuted = true"
        src="./assets/image/icon/open_audio.png"
        alt=""
      />
      <video
        style="width: 0; heigth: 0"
        id="tipvideo"
        x5-video-player-type="h5-page"
        x5-video-orientation="landscape|portrait"
        webkit-playsinline="true"
        playsinline="true"
        src=""
        :muted="ismuted"
      ></video>
    </div>
    <div class="home" v-if="ishome">
      <video
        id="ckplayer_a1"
        x5-video-player-type="h5-page"
        x5-video-orientation="landscape|portrait"
        webkit-playsinline="true"
        playsinline="true"
        src="./assets/video/bg.mp4"
        loop
        :muted="ismuted"
        autoplay
      ></video>
      <div class="htop">
        <h3>羊了羊（真ikun版）</h3>
        <p>仅供学习交流</p>
      </div>
      <van-button @click="togame">开始游戏</van-button>
    </div>
    <div class="game" v-if="!ishome">
      <video
        x5-video-player-type="h5-page"
        x5-video-orientation="landscape|portrait"
        webkit-playsinline="true"
        playsinline="true"
        src="./assets/audio/bg.mp3"
        loop
        :muted="ismuted"
        autoplay
      ></video>
      <div class="card-wrap" :style="cardWrapStyle">
        <div
          v-for="item in cardItemList"
          :key="item.key"
          :class="{ 'item-cover': item.cover }"
          class="card-item"
          :style="item.style"
          @click="clickCard(item)"
        >
          <img :src="item.content" alt="" />
        </div>
        <div class="penBox"></div>
        <div
          v-for="item in penddingList"
          :key="item.key"
          class="card-item"
          :style="item.style"
        >
          <img :src="item.content" alt="" />
        </div>

        <div
          v-for="item in clearList"
          :key="item.key"
          class="card-item clear-item"
          :style="item.style"
        >
          <img :src="item.content" alt="" />
        </div>
        <div
          v-for="item in saveList"
          :key="item.key"
          class="card-item"
          :style="item.style"
          @click="clickSaveCard(item)"
        >
          <img :src="item.content" alt="" />
        </div>
      </div>
      <div class="tools">
        <van-button
          icon="upgrade"
          :disabled="!tools.save"
          @click="saveCard"
        ></van-button>
        <van-button
          icon="replay"
          :disabled="!tools.rand"
          @click="randCard"
          style="margin-left: 10px"
        ></van-button>
      </div>
    </div>
  </div>
</template>

<script>
import { Dialog } from "vant";
class CardItem {
  static x = 20;
  static y = 21;
  static content = {
    1: "icon/1.jpg",
    2: "icon/2.jpg",
    3: "icon/3.jpg",
    4: "icon/4.jpg",
    5: "icon/5.jpg",
    6: "icon/6.jpg",
    7: "icon/7.jpg",
    8: "icon/8.jpg",
  };
  constructor({ x, y, z, key }) {
    this.x = x;
    this.y = y;
    this.z = z;
    this.key = key;
    const offset = z * 0;
    this.val = key;
    this.style = {
      top: y * CardItem.y + offset + "px",
      left: x * CardItem.x + offset + "px",
      width: CardItem.x * 2 - 2 + "px",
      height: CardItem.y * 2 - 8 + "px",
    };
  }

  setValue(val) {
    this.val = val;
    this.content = CardItem.content[val];
  }
}
export default {
  name: "App",
  data() {
    return {
      ismuted: true,
      ishome: true,
      option: {
        x: 6,
        y: 4,
        z: 8,
        cardRandom: 0.2,
        maxCardType: 8,
      },
      step: 0,
      win: false,
      cardMap: [],
      cardItemList: [],
      penddingList: [],
      clearList: [],
      saveList: [],
      calcValueList: [],
      maxWidth: 0,
      maxHeight: 0,
      tools: {
        save: true,
        rand: true,
      },
      timer: 0,
    };
  },
  computed: {
    cardWrapStyle() {
      return {
        width: (this.maxWidth + 2) * CardItem.x + "px",
        height: (this.maxHeight + 1) * CardItem.y + "px",
      };
    },
    leftOffset() {
      const wrapWidth = (this.maxWidth + 2) * CardItem.x;
      return (wrapWidth - 7 * CardItem.x * 2) / 2;
    },
  },
  mounted() {
    document.addEventListener(
      "WeixinJSBridgeReady",
      function () {
        var video = document.getElementById("ckplayer_a1");
        video.play();
      },
      false
    );
  },
  methods: {
    togame() {
      this.ishome = false;
      this.initGame();
    },
    tohome() {
      this.ishome = true;
    },
    randCard() {
      if (!this.tools.rand) {
        return;
      }
      this.tools.rand = false;
      const length = this.cardItemList.length;
      this.cardItemList.forEach((item) => {
        const randNum = Math.floor(length * Math.random());
        const newItem = this.cardItemList[randNum];
        let temp;
        temp = item.style.left;
        item.style.left = newItem.style.left;
        newItem.style.left = temp;
        temp = item.style.top;
        item.style.top = newItem.style.top;
        newItem.style.top = temp;
        temp = item.x;
        item.x = newItem.x;
        newItem.x = temp;
        temp = item.y;
        item.y = newItem.y;
        newItem.y = temp;
        temp = item.z;
        item.z = newItem.z;
        newItem.z = temp;
      });

      this.cardItemList.sort((a, b) => a.z - b.z);
      this.calcCover();
    },
    saveCard() {
      if (!this.tools.save) {
        return false;
      }
      this.tools.save = false;
      this.saveList = this.penddingList.slice(0, 3);
      setTimeout(() => {
        this.saveList.forEach((item, index) => {
          item.style.top = "110%";
          item.style.left = this.leftOffset + index * CardItem.x * 2 + "px";
          this.calcValueList[item.val]--;
        });
      }, 0);
      this.penddingList = this.penddingList.slice(3);
      this.penddingList.forEach((item, index) => {
        item.style.top = "160%";
        item.style.left = this.leftOffset + index * CardItem.x * 2 + "px";
      });
    },
    initGame() {
      this.step = 1;
      this.getMap(this.option);
      this.penddingList = [];
      this.clearList = [];
      this.saveList = [];
      this.tools.save = true;
      this.tools.rand = true;
      this.setCardValue({ maxCardType: Number(this.option.maxCardType) });
      this.calcCover();
    },
    // 表示地图最大为 x * y 张牌，最多有 z 层
    getMap({ x, y, z, cardRandom } = {}) {
      this.maxWidth = (x - 1) * 2;
      this.maxHeight = (y - 1) * 2 + 1;
      const cardMap = new Array(z);
      const cardItemList = [];
      let key = 0;
      // 地图初始化
      for (let k = 0; k < z; k++) {
        cardMap[k] = new Array(this.maxHeight);
        for (let i = 0; i <= this.maxHeight; i++) {
          cardMap[k][i] = new Array(this.maxWidth).fill(0);
        }
      }
      for (let k = 0; k < z; k++) {
        const shrink = Math.floor((z - k) / 3);
        // 行
        for (let i = shrink; i < this.maxHeight - shrink; i++) {
          // 列，对称设置
          const mid = Math.ceil((this.maxWidth - shrink) / 2);
          for (let j = shrink; j <= mid; j++) {
            let canSetCard = true;
            if (j > 0 && cardMap[k][i][j - 1]) {
              // 左边不能有牌
              canSetCard = false;
            } else if (i > 0 && cardMap[k][i - 1][j]) {
              // 上边不能有牌
              canSetCard = false;
            } else if (i > 0 && j > 0 && cardMap[k][i - 1][j - 1]) {
              // 左上不能有牌
              canSetCard = false;
            } else if (i > 0 && cardMap[k][i - 1][j + 1]) {
              // 右上不能有牌
              canSetCard = false;
            } else if (k > 0 && cardMap[k - 1][i][j]) {
              // 正底不能有牌
              canSetCard = false;
            } else if (Math.random() >= cardRandom) {
              canSetCard = false;
            }
            if (canSetCard) {
              key++;
              const cardItem = new CardItem({ x: j, y: i, z: k, key });
              cardMap[k][i][j] = cardItem;
              cardItemList.push(cardItem);
              // 对称放置
              if (j < mid) {
                key++;
                const cardItem = new CardItem({
                  x: this.maxWidth - j,
                  y: i,
                  z: k,
                  key,
                });
                cardMap[k][i][j] = cardItem;
                cardItemList.push(cardItem);
              }
            }
          }
        }
      }
      cardItemList.reverse();
      for (let i = 1; i <= key % 3; i++) {
        const clearItem = cardItemList.pop();
        cardMap[clearItem.z][clearItem.y][clearItem.x] = 0;
      }
      cardItemList.reverse();
      this.cardMap = cardMap;
      this.cardItemList = cardItemList;
    },
    setCardValue({ maxCardType } = {}) {
      // 卡片种类
      const valStack = new Array(maxCardType);
      this.calcValueList = new Array(maxCardType + 1).fill(0);
      // 给卡片设置值
      this.cardItemList.forEach((item) => {
        const value = Math.ceil(Math.random() * maxCardType);
        if (valStack[value]) {
          valStack[value].push(item);
          if (valStack[value].length === 3) {
            valStack[value].forEach((item) => {
              item.setValue(value);
            });
            valStack[value] = null;
          }
        } else {
          valStack[value] = [item];
        }
      });

      let count = 2;
      valStack.forEach((list) => {
        list &&
          list.forEach((item) => {
            count++;
            item.setValue(Math.floor(count / 3));
          });
      });
    },
    // 计算遮挡关系
    calcCover() {
      // 构建一个遮挡 map
      const coverMap = new Array(this.maxHeight);
      for (let i = 0; i <= this.maxHeight; i++) {
        coverMap[i] = new Array(this.maxWidth).fill(false);
      }

      // 从后往前，后面的层数高
      for (let i = this.cardItemList.length - 1; i >= 0; i--) {
        const item = this.cardItemList[i];
        const { x, y } = item;
        if (coverMap[y][x]) {
          item.cover = true;
        } else if (coverMap[y][x + 1]) {
          item.cover = true;
        } else if (coverMap[y + 1][x]) {
          item.cover = true;
        } else if (coverMap[y + 1][x + 1]) {
          item.cover = true;
        } else {
          item.cover = false;
        }
        coverMap[y][x] = true;
        coverMap[y + 1][x] = true;
        coverMap[y][x + 1] = true;
        coverMap[y + 1][x + 1] = true;
      }
    },
    clickSaveCard(item) {
      this.cardItemList.push(item);
      const index = this.saveList.indexOf(item);
      this.saveList = this.saveList
        .slice(0, index)
        .concat(this.saveList.slice(index + 1));
      this.clickCard(item);
    },
    removeThree() {
      let isRemove = false;
      this.penddingList.some((item) => {
        if (this.calcValueList[item.val] === 3) {
          isRemove = true;
          this.penddingList.forEach((newItem) => {
            if (newItem.val === item.val) {
              this.clearList.push(newItem);
            }
          });
          setTimeout(() => {
            this.clearList.forEach((item) => {
              item.style.left = this.leftOffset - 60 + "px";
            });
          }, 100);

          this.penddingList = this.penddingList.filter((newItem) => {
            return newItem.val !== item.val;
          });
          this.penddingList.forEach((item, index) => {
            item.style.top = "160%";
            item.style.left = this.leftOffset + index * CardItem.x * 2 + "px";
          });
          this.calcValueList[item.val] = 0;
          if (this.cardItemList.length === 0) {
            this.step = 2;
            this.result = true;
          }
        }
      });
      if (isRemove) {
        if (
          this.cardItemList.length == 0 &&
          this.penddingList.length == 0 &&
          this.saveList.length == 0
        ) {
          this.toMusic("win");
          Dialog.alert({
            message: "恭喜你，游戏成功！",
          }).then(() => {
            this.tohome();
          });
        } else {
          this.toMusic("tip");
        }
      } else {
        if (this.penddingList.length >= 7) {
          this.step = 2;
          this.result = false;
          this.toMusic("alert");
          Dialog.alert({
            message: "很遗憾，游戏失败！",
          }).then(() => {
            this.initGame();
          });
        } else {
          this.toMusic("click");
        }
      }
    },
    // 点击卡片
    clickCard(item) {
      this.penddingList.push(item);
      const index = this.cardItemList.indexOf(item);
      this.cardItemList = this.cardItemList
        .slice(0, index)
        .concat(this.cardItemList.slice(index + 1));
      this.calcCover();
      this.calcValueList[item.val]++;
      item.style.top = "160%";
      item.style.left =
        this.leftOffset +
        (this.penddingList.length - 1) * CardItem.x * 2 +
        "px";
      this.removeThree();
    },
    toMusic(type) {
      var audio = document.getElementById("tipvideo");
      audio.src = "audio/" + type + ".mp3"; //音乐的路径
      audio.play();
    },
  },
};
</script>

<style>
#app_loading {
  display: none;
}
.app {
  height: 100%;
  position: relative;
}
.home {
  width: 100%;
  height: 100%;
  color: #fff;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
}
.home video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  position: absolute;
  z-index: -1;
  background-color: rgb(0, 0, 0);
}
.htop {
  text-align: center;
}
.htop h3 {
  line-height: 55px;
  margin-top: 45px;
}
.htop p {
  font-size: 13px;
}
.home .van-button {
  margin-bottom: 50px;
}
.aicon {
  position: absolute;
  top: 30px;
  right: 30px;
  z-index: 100;
}
.aicon2 {
  position: absolute;
  top: 30px;
  left: 30px;
  z-index: 100;
}
.game {
  background: url(./assets/image/bg.webp);
  background-size: 100%;
  width: 100%;
  height: 100%;
}
.card-wrap {
  position: relative;
  margin: 10% auto 0 auto;
}

.card-item {
  font-size: 28px;
  text-align: center;
  position: absolute;
  border-radius: 2px;
  box-sizing: border-box;
  background: #ddd;
  opacity: 1;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0px 3px 0 0 #fff, 0 8px 0 0 #ddd, 0 8px 0 2px #333, 0 0 0 2px #333;
}

.card-item:hover {
  transform: scale3d(1.1, 1.1, 1.1);
  z-index: 1;
}
.card-item img {
  width: 100%;
  height: 100%;
}

.item-cover {
  pointer-events: none;
  box-shadow: 0px 3px 0 0 #999, 0 8px 0 0 #666, 0 8px 0 2px #000, 0 0 0 2px #000;
}

.item-cover:after {
  border-radius: 2px;
  content: "";
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
  top: 0;
  background: #000;
  opacity: 0.55;
}

.card-tips {
  white-space: nowrap;
  position: absolute;
  left: 50%;
  top: 130%;
  transform: translate(-50%, 0);
  pointer-events: none;
}

.tools {
  position: absolute;
  bottom: 40px;
  width: 100%;
  left: 0;
  text-align: center;
}
.clear-item {
  pointer-events: none;
}
.penBox {
  width: 100%;
  background-color: #fff;
  border: 5px solid rgb(236, 157, 46);
  position: absolute;
  top: 156%;
  left: -27px;
  padding: 2.8px 21px;
  box-sizing: content-box;
  height: 40px;
  border-radius: 10px;
}
</style>
