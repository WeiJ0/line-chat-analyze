<template>
  <div class="bg-[#EDF1D6] h-full min-h-screen w-full">
    <header class="bg-[#9DC08B] py-5">
      <div class="container mx-auto flex items-center px-5 md:px-0">
        <h1 class="text-2xl md:text-4xl font-bold text-[#40513B]">Line 聊天紀錄分析</h1>
        <label for="txtUpload"
          class=" md:text-xl text-center bg-[#EDF1D6] text-[#609966] font-bold ml-5 py-2 px-4 rounded
                                                                                                                      hover:bg-[#40513B] hover:text-[#EDF1D6] cursor-pointer ease-in duration-300">
          上傳 txt 檔案</label>
        <input class=" hidden" type="file" name="txtUpload" id="txtUpload" @change="$event => uploadFile($event)">
        <span class="hidden md:block text-white text-lg font-bold ml-auto">By WeiJie</span>
      </div>
    </header>
    <main>
      <div class="container mx-auto py-10">
        <h2 class="text-xl text-center mb-5 font-bold text-[#40513B]" v-if="txtName != ''">{{ txtName.replace('[LINE]',
          '').replace('.txt', '') }} 聊天分析
        </h2>
        <h2 class="text-xl text-center mb-5 font-bold text-[#40513B]" v-else>請先上傳從 Line 匯出 Txt 檔</h2>
        <template v-if="days.length">
          <h5 class="text-md text-center font-bold">{{ days[0].day }}~{{ days[days.length - 1].day }} ({{ days.length }}天)
          </h5>
        </template>
        <div class="flex flex-wrap">
          <Count title="總計訊息數量" :result="result.messages" :top="tops.messages" icon="fa-comment-dots" />
          <Count title="總計貼圖數量" :result="result.stickers" :top="tops.stickers" icon="fa-sticky-note" />
          <Count title="總計圖片數量" :result="result.images" :top="tops.images" icon="fa-image" />
          <Count title="總計影片數量" :result="result.videos" :top="tops.videos" icon="fa-video" />
        </div>
        <template v-if="names.length > 2">
          <wordCloud title="最常發言的人" :words="names" />
        </template>
        <template v-if="allMessages.length > 2">
          <wordCloud title="最常說的話" :words="allMessages" />
        </template>
      </div>
    </main>
  </div>
</template>

<script>
import Count from './components/Count.vue';
import wordCloud from './components/wordCloud.vue';
import * as dayjs from 'dayjs'

export default {
  name: 'App',
  components: {
    Count,
    wordCloud
  },
  data() {
    return {
      txtName: '',
      txtContent: '',
      days: [],
      result: {
        messages: 0,
        stickers: 0,
        images: 0,
        videos: 0,
      },
      callTimeSecond: 0,
      allMessages: [['loading', 0]],
      names: [['loading', 0]],
      tops: {
        messages: [],
        stickers: [],
        images: [],
        videos: [],
      },
    }
  },
  methods: {
    uploadFile: function (event) {
      var file = event.target.files[0];
      var reader = new FileReader();

      this.txtName = '';
      this.txtContent = '';
      this.days = [];
      this.result = {
        messages: 0,
        stickers: 0,
        images: 0,
        videos: 0,
      };
      this.tops = {
        messages: [],
        stickers: [],
        images: [],
        videos: [],
      };
      this.names = [];
      this.allMessages =[];

      this.txtName = event.target.files[0].name;
      reader.onload = function (event) {
        this.txtContent = event.target.result;
        this.analyzer();
      }.bind(this);
      reader.readAsText(file);
    },
    analyzer() {
      const contents = this.txtContent.split(/[\r\n]+/)
        .filter(content =>
          !content.includes('[LINE]') &&
          !content.includes('儲存日期') &&
          !content.includes('收回訊息') &&
          !content.includes('邀請') &&
          !content.includes('加入') &&
          !content.includes('離開') &&
          !content.includes('退出'));

      let names = [];
      let allMessages = [];
      let messages = [];
      let stickers = [];
      let images = [];
      let videos = [];
      let callTimeSecond = 0;

      const dateReg = /^\d{4}\.\d{2}\.\d{2}\s[\u4e00-\u9fa5]{3}$/;
      const dateReg2 = /(\d{4}\/\d{1,2}\/\d{1,2})（[^\u4e00-\u9fa5]*?週\S*）/;
      const timeReg = /^([01]\d|2[0-3]):[0-5]\d$/;

      let currentDay = '';

      const calcCallTimes = (time) => {
        const timeType = time.split(':').length;

        if (timeType === 2) {
          callTimeSecond += parseInt(time.split(':')[0]) * 60 + parseInt(time.split(':')[1]);
        } else if (timeType === 3) {
          callTimeSecond += parseInt(time.split(':')[0]) * 3600 + parseInt(time.split(':')[1]) * 60 + parseInt(time.split(':')[2]);
        }
      }

      for (let i = 0; i < contents.length; i++) {
        if (dateReg.test(contents[i]) || dateReg2.test(contents[i])) {
          if (currentDay) {
            this.days.push({
              day: currentDay,
              messages: messages,
              stickers: stickers,
              images: images,
              videos: videos
            })

            messages = [];
            stickers = [];
            images = [];
            videos = [];
          }
          currentDay = contents[i];
        } else {
          if (timeReg.test(contents[i].split(/[\t\r\n\s]+/)[0])) {
            const contentArray = contents[i].split(/[\t\r\n\s]+/);
            const name = contentArray[1];
            const type = contentArray[contentArray.length - 1].replace('[', '').replace(']', '');

            names.push(name);

            if (type == '貼圖')
              stickers.push(name);
            else if (type == '圖片' || type == '照片')
              images.push(name);
            else if (type == '影片')
              videos.push(name);
            else {

              allMessages.push(type);
              messages.push(name);

              if (contents[i].includes('通話時間')) {
                calcCallTimes(type);
              }
            }

          } else {
            contents[i - 1] += ' ' + contents[i]; // 將該項目加到上一個項目
            contents.splice(i, 1); // 刪除該項目
            i--; // 因為 i 項目已被刪除，i 需要減 1
          }
        }

        // 若是最後一個項目，且該項目不是日期，則將該項目加入 days
        if (i == contents.length - 1 && this.days.find(day => day.day == currentDay) == undefined) {
          this.days.push({
            day: currentDay,
            messages: messages,
            stickers: stickers,
            images: images,
            videos: videos
          })
        }
      }

      this.callTimeSecond = callTimeSecond;

      const calcCount = arr => Object.entries(
        arr.reduce((acc, curr) => {
          acc[curr] ? acc[curr]++ : (acc[curr] = 1);
          return acc;
        }, {})
      ).map(([text, value]) => ([text, value]))


      this.allMessages = calcCount(allMessages).filter(([text, value]) => value > 1 && !text.includes('http'));

      // 將姓名統計，用來產生文字雲
      this.names = calcCount(names).filter(([text, value]) => value > 5);

      const findTop = (arr) => {
        const count = arr.reduce((acc, curr) => {
          if (typeof acc[curr] === "undefined") {
            acc[curr] = 1;
          } else {
            acc[curr] += 1;
          }
          return acc;
        }, {});
        const countArr = Object.entries(count);
        countArr.sort((a, b) => b[1] - a[1]);
        const topFive = countArr.slice(0, 3);

        return topFive;
      }

      let totalMessageArr = [...this.days.map(day => day.messages)].flat();
      let totalStickerArr = [...this.days.map(day => day.stickers)].flat();
      let totalImageArr = [...this.days.map(day => day.images)].flat();
      let totalVideoArr = [...this.days.map(day => day.videos)].flat();

      this.result.messages = totalMessageArr.length;
      this.result.stickers = totalStickerArr.length;
      this.result.images = totalImageArr.length;
      this.result.videos = totalVideoArr.length;

      this.tops.messages = findTop(totalMessageArr);
      this.tops.stickers = findTop(totalStickerArr);
      this.tops.images = findTop(totalImageArr);
      this.tops.videos = findTop(totalVideoArr);
    }
  },
}
</script>
