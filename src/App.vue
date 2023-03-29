<template>
  <div id="page">
    <div
      id="music-bg"
      :style="{ backgroundImage: 'url(' + musicImgSrc[currentMusicNo] + ')' }"
    ></div>
    <div id="black-bg"></div>
    <div id="search-content">
      <div id="searchbox">
        <img
          id="search-icon"
          src="https://cdn-icons-png.flaticon.com/512/54/54481.png"
        />
        <input
          type="text"
          id="search-input"
          placeholder="검색.."
          v-model="inputSongName"
          @keydown.enter="search()"
        />
        <img
          id="new-song"
          src="https://cdn-icons-png.flaticon.com/512/748/748113.png"
          @click="openNewSong()"
        />
      </div>
    </div>
    <div id="play-content">
      <img
        id="shuffle"
        class="icon"
        src="https://cdn-icons-png.flaticon.com/512/2550/2550084.png"
        @click="toggleShuffle()"
      />
      <img
        id="play"
        class="icon"
        :src="pauseImgSrc[isPaused]"
        @mousedown="pausePlay()"
      />
      <img
        id="like"
        class="icon"
        :src="likeButtonSrc[likedMusic[currentMusicNo]]"
        @mousedown="toggleLike()"
        :class="{ 'like-btn-ani': likedMusic[currentMusicNo] }"
      />
    </div>
    <div id="music-content">
      <span
        class="move-song"
        id="move-prev"
        @mouseenter="arrowHalo('move-prev', true)"
        @mouseup="findSong('prev')"
        @mouseleave="arrowHalo('move-prev', false)"
        >←</span
      >
      <div id="music-acontent">
        <img
          id="current-music-img"
          :src="musicImgSrc[currentMusicNo]"
          @mouseenter="mainImgMotion(true)"
          @mousedown="pausePlay()"
          @mouseleave="mainImgMotion(false)"
          :class="{ paused: isPaused }"
        />
        <img
          id="pausing"
          :src="pauseImgSrc[0]"
          v-if="isPaused"
          @mousedown="pausePlay()"
        />
        <p
          id="music-name"
          :style="'color: ' + musicNameColor[likedMusic[currentMusicNo]]"
        >
          {{ musicName[currentMusicNo] }}
        </p>
        <p id="music-singer">{{ musicSinger[currentMusicNo] }}</p>
      </div>
      <span
        class="move-song"
        id="move-next"
        @mouseenter="arrowHalo('move-next', true)"
        @mouseup="findSong('next')"
        @mouseleave="arrowHalo('move-next', false)"
        >→</span
      >
    </div>
    <div id="new-song-page" class="goodbye" @keyup.enter="addNewSong()">
      <div id="new-song-popup">
        <h3>곡 추가</h3>
        <p>곡 제목:</p>
        <input
          type="text"
          placeholder="제목을 입력해 주세요."
          v-model="newSongName"
        />
        <p>가수 이름:</p>
        <input
          type="text"
          placeholder="가수 이름을 입력해 주세요."
          v-model="newSingerName"
        />
        <p>앨범 커버 URL:</p>
        <input
          type="text"
          placeholder="앨범 커버의 링크 주소를 입력해 주세요."
          v-model="newImgUrl"
        />
        <button @click="addNewSong()">추가</button>
      </div>
    </div>
  </div>
</template>

<script>
/*import * as Vibrant from 'node-vibrant'

const v = Vibrant.from('./assets/logo.png')
  .getPalette()
  .then((palette) => console.log(palette))*/

export default {
  name: 'App',
  data() {
    return {
      shuffle: false,
      currentMusicNo: 1,
      musicImgSrc: [
        '',
        'https://images.genius.com/c055b09c6bfc9b9c62d14e7beb0ee77d.1000x1000x1.jpg',
        'https://images.genius.com/6010080e44b74d10971417cc9c629998.1000x1000x1.jpg',
        'https://i0.wp.com/colorcodedlyrics.com/wp-content/uploads/2019/02/LOONA-X-X-1.jpg?fit=1000%2C1000&ssl=1',
        'https://upload.wikimedia.org/wikipedia/commons/c/c7/The_Dark_Side_of_the_Moon_Cover.svg',
        'https://i0.wp.com/m.izm.co.kr/wp-content/uploads/2021/12/%EC%B0%BD%EB%AA%A8_%EC%95%A8%EB%B2%94%EC%BB%A4%EB%B2%84.jpg?fit=1000%2C1000&ssl=1',
        'https://images.genius.com/fd2967b81129659ae94a021e1774be0d.1000x1000x1.jpg',
        'https://lh3.googleusercontent.com/ew3gzk5RCGo9VrMsyLyRHpfVIfnAziPqtCIqw-_c6NrsY9zyVzaBhLj3n4OKvn54UUIw5A8v_6aw8tbh=w544-h544-l90-rj',
        'https://lh3.googleusercontent.com/mJlDsJvEthfhzUaaFVI5J_4TdjOpbsm1VhxplrtURuNfzpbTe_C6RDsHri7Aa_v8Hr53m6HdsAL5TW4=w544-h544-l90-rj',
      ],
      musicName: [
        '',
        'Teddy Bear',
        'ASAP',
        'Butterfly',
        'The Great Gig In The Sky',
        'Hyperstar',
        'WHEN I MOVE',
        'One & Only',
        '5분만',
      ],
      musicSinger: [
        '',
        'STAYC',
        'STAYC',
        'LOONA (이달의 소녀)',
        'Pink Floyd',
        '창모',
        'KARA',
        '고원 (이달의 소녀)',
        'Hash Swan (해쉬 스완)',
      ],
      isPaused: 0,
      pauseImgSrc: [
        'https://cdn-icons-png.flaticon.com/512/3669/3669483.png',
        'https://cdn-icons-png.flaticon.com/512/27/27223.png',
      ],
      likedMusic: [],
      likeButtonSrc: [
        'https://cdn-icons-png.flaticon.com/512/1077/1077035.png',
        'https://cdn-icons-png.flaticon.com/512/1077/1077086.png',
      ],
      musicNameColor: ['rgba(207, 207, 207, 0.8)', 'rgba(255, 242, 0, 0.8);'],
      inputSongName: '',
      newSongName: '',
      newSingerName: '',
      newImgUrl: '',
    }
  },
  mounted() {
    for (let a in this.musicName) {
      this.likedMusic.push(0)
      console.log(a)
    }
  },
  methods: {
    findSong(toWhere) {
      var nextMusic = 0
      const musicLength = this.musicName.length - 1
      console.log(`music length is ${musicLength}`)

      /**choose what's next! */
      if (this.shuffle)
        while (nextMusic === this.currentMusicNo || nextMusic === 0) {
          nextMusic = parseInt(Math.random() * musicLength) + 1
        }
      else if (toWhere === 'prev')
        this.currentMusicNo > 1
          ? (nextMusic = parseInt(this.currentMusicNo) - 1)
          : (nextMusic = musicLength)
      else if (toWhere === 'next')
        this.currentMusicNo < musicLength
          ? (nextMusic = parseInt(this.currentMusicNo) + 1)
          : (nextMusic = 1)
      console.log(` next music number is ${nextMusic}`)
      this.changeSong(toWhere, nextMusic)
    },
    changeSong(toWhere, nextMusic) {
      const musicImg = document.getElementById('music-acontent')
      /**animation and change */
      musicImg.classList.add(`music-img-${toWhere}`)
      document.getElementById(`move-${toWhere}`).classList.add(`go-${toWhere}`)
      setTimeout(() => {
        this.currentMusicNo = nextMusic
        document.getElementById('current-music-img').src =
          this.musicImgSrc[this.currentMusicNo]
        document.getElementById('music-bg').style.backgroundImage = `url(${
          this.musicImgSrc[this.currentMusicNo]
        })`
        this.isPaused = 0
      }, 200)
      setTimeout(() => {
        musicImg.classList.remove(`music-img-${toWhere}`)
        document
          .getElementById(`move-${toWhere}`)
          .classList.remove(`go-${toWhere}`)
      }, 400)
    },
    toggleShuffle() {
      const shuffleButton = document.getElementById('shuffle')
      this.shuffle = !this.shuffle
      this.shuffle
        ? (shuffleButton.style.filter = 'invert(90)')
        : (shuffleButton.style.filter = '')
    },
    pausePlay() {
      this.isPaused ? (this.isPaused = 0) : (this.isPaused = 1)
    },

    toggleLike() {
      const likeButton = document.getElementById('like')

      if (!this.likedMusic[this.currentMusicNo]) {
        likeButton.style.filter = 'invert(90)'
      } else {
        likeButton.style.filter = ''
      }
      this.likedMusic[this.currentMusicNo]
        ? (this.likedMusic[this.currentMusicNo] = 0)
        : (this.likedMusic[this.currentMusicNo] = 1)
    },
    arrowHalo(direction, isOn) {
      var victim = document.getElementById(direction)
      victim.classList.toggle(`${direction}-ani`)
      const haloColor = isOn
        ? 'rgba(161, 161, 161, 0.1)'
        : 'rgba(161, 161, 161, 0)'
      const arrowColor = isOn
        ? 'rgba(255, 255, 255, 0.9)'
        : 'rgba(211, 211, 211, 0.9)'
      victim.style.backgroundColor = haloColor
      victim.style.color = arrowColor
    },
    mainImgMotion(isOn) {
      const img = document.getElementById('current-music-img')
      img.classList.toggle('main-img-ani')
      isOn
        ? (img.style.animationDirection = 'alternate')
        : (img.style.animationDirection = 'alternate-reverse')
    },
    search() {
      var victim = 0
      if (this.inputSongName !== '') {
        for (let i in this.musicName) {
          if (
            this.inputSongName.toLowerCase().replace(/\s+/g, '') ===
            this.musicName[i].toLowerCase().replace(/\s+/g, '')
          ) {
            victim = i
            break
          }
        }
        this.inputSongName = ''
      }
      console.log(victim)
      if (victim !== 0) {
        this.changeSong('next', victim)
      }
    },

    openNewSong() {
      const page = document.getElementById('new-song-page')
      page.classList.remove('goodbye')
      page.style.zIndex = 5
    },
    addNewSong() {
      if (
        this.newSongName !== '' &&
        this.newSingerName !== '' &&
        this.newImgUrl !== ''
      ) {
        this.musicName.push(this.newSongName)
        this.musicSinger.push(this.newSingerName)
        this.musicImgSrc.push(this.newImgUrl)
        this.likedMusic.push(0)
      }
      this.newSongName = ''
      this.newSingerName = ''
      this.newImgUrl = ''
      document.getElementById('new-song-page').classList.add('goodbye')
      document.getElementById('new-song-page').style.zIndex = 0
    },
  },
}
</script>

<style>
div {
  display: inline;
}

p,
h3 {
  display: inline;
  color: rgba(234, 233, 233, 0.9);
}

span {
  display: inline;
}

#page {
  width: 100vw;
  height: 100vh;
  position: relative;
  display: block;
  -webkit-user-select: none;
}

#music-bg {
  width: 100vw;
  height: 100vh;
  position: absolute;
  background-image: url('https://images.genius.com/c055b09c6bfc9b9c62d14e7beb0ee77d.1000x1000x1.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  filter: blur(10px);
  display: block;
  transform: scale(1.08);
}

#black-bg {
  width: 100vw;
  height: 100vh;
  position: absolute;
  background: linear-gradient(rgba(0, 0, 0, 0.22), rgba(0, 0, 0, 0.9));
  transform: scale(1.08);
}

#search-content {
  width: 100vw;
  height: 22%;
  background: linear-gradient(rgba(0, 0, 0, 0.4), rgba(0, 0, 0, 0));
  display: flex;
  position: absolute;
  top: -6%;
  flex-flow: column wrap;
  justify-content: center;
  align-content: center;
  opacity: 0.01;
  transform: scale(1.05);
  transition: all 0.6s ease;
  z-index: 1;
}

#search-content:hover {
  top: -3%;
  opacity: 1;
}

#searchbox {
  background-color: rgba(255, 255, 255, 0.3);
  width: 35%;
  height: 40%;
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  align-content: center;
  border-radius: 40px;
}

#search-icon {
  position: relative;
  top: 10%;
  filter: invert(90%) brightness(100%);
  width: 20px;
  height: 20px;
  margin-right: 10px;
}

#search-input {
  width: 78%;
  height: 50%;
  border-radius: 12px;
  background-color: rgba(255, 255, 255, 0);
  color: white;
  margin-right: 20px;
}

#new-song {
  position: relative;
  top: 10%;
  filter: invert(90%) brightness(100%);
  width: 20px;
  height: 20px;
  transition: all 0.3s ease;
}

#new-song:hover {
  transform: scale(1.25) rotate(45deg);
}

#new-song-page {
  position: absolute;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  flex-flow: row wrap;
  justify-content: center;
  align-content: center;
  z-index: 0;
}

.goodbye {
  display: none;
  opacity: 0;
  z-index: -1;
}

#new-song-popup {
  width: 30%;
  height: 60%;
  background-color: rgb(180, 180, 180);
  display: flex;
  flex-flow: column wrap;
  align-content: center;
  justify-content: space-around;
  border-radius: 30px;
}

#new-song-popup > input {
  height: 8%;
  width: 65%;
  background-color: rgba(255, 255, 255, 0.6);
  border-radius: 10px;
}

#new-song-popup > button {
  height: 10%;
  width: 30%;
  position: relative;
  left: 15%;
}

#new-song-popup > p {
  color: rgb(0, 0, 0);
}

#music-content {
  position: absolute;
  top: 18%;
  display: flex;
  justify-content: space-between;
  align-content: center;
  flex-flow: column wrap;
  height: 85%;
  width: 100%;
  transition: all 0.8s ease;
  z-index: 1;
}

.move-up {
  animation-name: moveUp;
  animation-direction: alternate;
  animation-duration: 0.5s;
}

.move-song {
  position: relative;
  font-size: 9em;
  font-weight: bolder;
  color: rgba(246, 246, 246, 0.716);
  background-color: rgba(9, 9, 9, 0);
  border-radius: 80px;
  width: 1em;
  height: 1em;
}

#move-prev {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-content: center;
  position: absolute;
  top: 15%;
  left: 23%;
  z-index: 1;
}

.move-prev-ani {
  animation-name: overLeft;
  animation-duration: 0.12s;
  animation-direction: alternate;
  animation-iteration-count: 2;
}

.go-prev {
  animation-name: goLeft;
  animation-duration: 0.4s;
  animation-direction: normal;
  animation-iteration-count: 1;
}

#move-next {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-content: center;
  position: absolute;
  top: 15%;
  right: 23%;
  z-index: 1;
}

.move-next-ani {
  animation-name: overRight;
  animation-duration: 0.12s;
  animation-direction: alternate;
  animation-iteration-count: 2;
}

.go-next {
  animation-name: goRight;
  animation-duration: 0.4s;
  animation-direction: normal;
  animation-iteration-count: 1;
}

#music-acontent {
  width: 50%;
  height: 100%;
  justify-content: space-between;
  align-content: center;
  display: flex;
  position: relative;
  flex-flow: column wrap;
}

#music-name {
  position: relative;
  font-size: 2.5em;
  font-weight: bold;
  display: flex;
  justify-content: center;
  bottom: 4%;
}

#music-singer {
  display: flex;
  position: relative;
  bottom: 14%;
  justify-content: center;
  font-size: 1.2em;
  color: rgba(207, 207, 207, 0.7);
}

#pausing {
  display: flex;
  position: absolute;
  width: 11pc;
  height: 11pc;
  flex-flow: row wrap;
  justify-content: center;
  align-self: center;
  filter: invert(70) drop-shadow(0px 0px 50px rgba(0, 0, 0, 1));
  opacity: 0.8;
  top: 15%;
  transition: all 0.56s ease;
  z-index: 2;
}

#current-music-img {
  display: flex;
  position: relative;
  flex-flow: row wrap;
  justify-content: center;
  align-self: center;
  width: 22pc;
  height: 22pc;
  transition: all 0.56s ease;
  filter: drop-shadow(0px 0px 90px rgba(244, 244, 244, 0.25));
  -webkit-user-drag: none;
}

#current-music-img:hover ~ #pausing {
  width: 12pc;
  height: 12pc;
}

#current-music-img:hover {
  width: 23pc;
  height: 23pc;
  filter: drop-shadow(0px 0px 90px rgba(244, 244, 244, 0.35));
}

.paused {
  filter: blur(5px);
  opacity: 0.5;
}

.music-img-prev {
  animation-name: imgGoLeft;
  animation-duration: 0.4s;
  animation-direction: normal;
  animation-iteration-count: 1;
}

.music-img-next {
  animation-name: imgGoRight;
  animation-duration: 0.4s;
  animation-direction: normal;
  animation-iteration-count: 1;
}

#play-content {
  width: 100vw;
  height: 15%;
  display: flex;
  flex-flow: row wrap;
  align-content: flex-start;
  justify-content: center;
  position: absolute;
  background: linear-gradient(rgba(0, 0, 0, 0), rgba(0, 0, 0, 0.5));
  transform: scale(1.04);
  bottom: -2%;
  opacity: 1%;
  transition: all 0.5s ease;
  z-index: 2;
}

#play-content:hover {
  bottom: -1%;
  opacity: 100%;
}

#play-content:hover + #music-content {
  top: 12%;
}

#hover-only {
  display: flex;
  position: absolute;
  width: 100vw;
  height: 11%;
  flex-wrap: wrap;
  align-content: flex-end;
  bottom: 0%;
  z-index: 3;
}

#hover-only:hover + #music-content {
  top: 15%;
}

.icon {
  filter: invert(50%) brightness(100%);
  width: 35px;
  height: 35px;
  padding-right: 25px;
}

#play {
  filter: invert(75%) brightness(100%);
  width: 50px;
  height: 50px;
  transition: all 0.3s ease;
}

#play:hover {
  transform: scale(1.15);
}

#shuffle,
#like {
  top: 6px;
  position: relative;
  transition: all 0.3s ease;
}

#shuffle:hover {
  transform: scale(1.15);
}

#like {
  padding-right: 0px;
  transition: all 0.3s ease;
}

.like-btn-ani {
  animation-name: likeMusic;
  animation-duration: 0.4s;
  animation-direction: normal;
  animation-iteration-count: 1;
}

#like:hover {
  transform: scale(1.15);
}

.liked {
  filter: hue-rotate(180deg);
}

.fade {
  opacity: 0;
}

@keyframes likeMusic {
  from {
    width: 35px;
    height: 35px;
  }
  10% {
    width: 30px;
    height: 30px;
  }
  30% {
    width: 40px;
    height: 40px;
  }
  60% {
    width: 33px;
    height: 33px;
  }
  80% {
    width: 36px;
    height: 36px;
  }
  to {
    width: 35px;
    height: 35px;
  }
}

@keyframes overRight {
  from {
    right: 23%;
  }
  50% {
    right: 22.4%;
    transform: rotate(-4deg);
  }
  to {
    right: 23%;
  }
}

@keyframes overLeft {
  from {
    left: 23%;
  }
  50% {
    left: 22.4%;
    transform: rotate(4deg);
  }
  to {
    left: 23%;
  }
}

@keyframes goRight {
  from {
    right: 23%;
    opacity: 1;
  }
  25% {
    right: 16%;
    opacity: 0;
  }
  75% {
    right: 26%;
    opacity: 0;
  }
  to {
    right: 23%;
    opacity: 1;
  }
}

@keyframes goLeft {
  from {
    left: 23%;
    opacity: 1;
  }
  25% {
    left: 16%;
    opacity: 0;
  }
  75% {
    left: 26%;
    opacity: 0;
  }
  to {
    left: 23%;
    opacity: 1;
  }
}

@keyframes imgGoRight {
  from {
    opacity: 1;
    left: 0%;
    transform: scale(1) rotate(0deg);
  }
  40% {
    opacity: 0;
    left: -5%;
    transform: scale(0.8) rotate(-5deg);
  }
  60% {
    opacity: 0;
    left: 5%;
    transform: scale(0.8) rotate(5deg);
  }
  to {
    opacity: 1;
    left: 0%;
    transform: scale(1) rotate(0deg);
  }
}

@keyframes imgGoLeft {
  from {
    opacity: 1;
    right: 0%;
    transform: scale(1) rotate(0deg);
  }
  40% {
    opacity: 0;
    right: -5%;
    transform: scale(0.8) rotate(5deg);
  }
  60% {
    opacity: 0;
    right: 5%;
    transform: scale(0.8) rotate(-5deg);
  }
  to {
    opacity: 1;
    right: 0%;
    transform: scale(1) rotate(0deg);
  }
}
</style>
