<template>
  <div id="app">
    <div
      contenteditable="true"
      id="fake-textarea"
      class="fake-textarea"
      @input="onInput"
      v-html="result"
      spellcheck="false"
    ></div>
    <p>
      phrase: {{ this.phrase }}
    </p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data: function() {
    return {
      keyword: '',
      result: '',
      grayListed: [],
      blackListed: [],
      undefined: [],
      phrase: '',
      secondsBeforeCheck: .5,
      timeout: null
    }
  },
  components: {},
  methods: {
    async check() {
      if(!this.keyword.length) {
        return;
      }
      const url = 'https://api.sikayetvar.com/dictionary/phrase/check';
      const { data } = await axios.post(
        url, 
        this.keyword,
        {
          headers: {
            'Content-Type': 'text/plain'
          }
        }
      );
      
      this.grayListed = data.grayListed;
      this.blackListed = data.blackListed;
      this.undefined = data.undefined;
      this.phrase = data.phrase;

      this.result = '';
      this.keyword.split(' ').map((newWord) => {
        const word = newWord.replace(' ', '').toLowerCase();
        const grayIndex = this.grayListed.indexOf(word);
        const blackIndex = this.blackListed.indexOf(word);
        const undefinedIndex = this.undefined.indexOf(word);
        if(grayIndex !== -1) {
          this.result += `<span class="graylisted">${newWord}</span>`;
        } else if(blackIndex !== -1) {
          this.result += `<span class="blacklisted">${newWord}</span>`;
        } else if(undefinedIndex !== -1) {
          this.result += `<span class="undefined">${newWord}</span>`;
        } else {
          this.result += `${newWord}`;
        }
      });

      this.result = this.result.replace(/&nbsp;/g, '');

      setTimeout(() => {
        this.setCursor();
      }, 0);
    },
    async onInput(e) {
      this.keyword = e.target.innerText.replace(/<\/?span[^>]*>/g, '');
      this.keyword = this.keyword.replace(/&nbsp;/g, '');

      if(this.timeout === null) {
        this.timeout = setTimeout(async () => {
          await this.check();
        }, this.secondsBeforeCheck * 1000);
      } else {
        clearTimeout(this.timeout);
        this.timeout = setTimeout(async () => {
          await this.check();
        }, this.secondsBeforeCheck * 1000);
      }
    },
    setCursor() {
      let el = document.getElementById("fake-textarea");
      let range = document.createRange();
      let sel = window.getSelection();

      sel.removeAllRanges()
      range.selectNodeContents(el);
      range.collapse(false);
      sel.addRange(range);
      el.focus();

      // const innerText = el.innerText.replace(/<\/?span[^>]*>/g,"");
      // console.log(el.childNodes[0]);
      // range.setStart(el.childNodes[0], 5);
      // range.collapse(true);
      
      // sel.removeAllRanges();
      // sel.addRange(range);
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.fake-textarea {
  border: 1px solid black;
  height: 200px;
  width: 400px;
  text-align: left;
  padding: .5rem;
}

.graylisted {
  position: relative;
}

.graylisted::after {
  content: '1';
  vertical-align: sub;
  font-size: x-small;
  position: absolute;
  top: 12px;
  right: -4px;
}

.blacklisted {
  text-decoration: line-through;
  color: red;
}

.undefined {
  text-decoration: underline;
  text-decoration-color: gray;
}

.sub {
  vertical-align: sub;
  font-size: x-small;
}
</style>
