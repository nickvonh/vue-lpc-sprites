<template>
  <div id="app">
    <button @click="clearCtx">clear</button>
    <div class="choices">
      
      <div v-for="(category, key) in orderedCategory" :key="key">
        <h2>{{category[0]}}</h2>
        <ul>
          <li v-for="(choice, index) in category[1].files" :key="index">
            <span @click="addImage(category[0],choice,category[1].index )">{{choice}}</span>
          </li>
        </ul>
      </div>
    </div>
    <div class="preview">
       <canvas ref="sprites" width="832" height="1344">HTML5 Browser required.</canvas>
    </div>
  </div>
</template>

<script>
import femaleSprites from './assets/female-sprites.json'
export default {
  name: 'app',
  mounted(){
    this.canvas = this.$refs.sprites;
  },
  data () {
    return {
      dir: null,
      canvas: null,
      root: '../sprites/female/',
      sprites: {
        female : femaleSprites
      },
      layers: {
        'accessories': {},
        'behind-body': {},
        'belt': {},
        'body': {},
        'cape': {},
        'ears': {},
        'eyes': {},
        'facial': {},
        'feet': {},
        'hair': {},
        'hands': {},
        'head': {},
        'legs': {},
        'nose': {},
        'shoulders': {},
        'torso': {},
        'weapons': {},
        'weapons-oversize': {}
      },
      images: {

      }
    }
  },
  computed: {
    ctx(){
      return this.canvas.getContext('2d');
    },
    orderedCategory(){
      let sort = []
      for(let each in this.sprites.female){
        let obj = this.sprites.female[each]
        sort.push([each, obj])
      }
      return sort.sort((a,b)=>{
        return a[1].index - b[1].index
      })
    },
    orderedChoices(){
      let sort = []
      if(!!this.layers){
        for(let each in this.layers){
          let obj = JSON.parse(JSON.stringify(this.layers[each]))
          sort.push([each, obj])
        }
        return sort.filter(a => {
          return Object.keys(a[1]).length > 0
        }).sort((a,b)=>{
          return a[1].index - b[1].index
        })
      }
    }
  },
  watch: {
    async orderedChoices(nu,old){
      this.ctx.clearRect(0,0,this.canvas.width,this.canvas.height);
      let promises = [];
      for(let each in nu){
        let obj = nu[each][1]
        if(!this.images[obj.file]){
          promises.push(new Promise((resolve,reject)=>{
              obj.img = new Image();
              obj.img.src = obj.file;
              obj.img.onload = ()=>{
                resolve(obj.img)
              }
            })
          )
        }
      }
      let done = await Promise.all(promises)
      if(!!done){
        console.log(promises)
        for(let each in nu){
          let obj = nu[each][1]
          console.log(obj)
          this.ctx.drawImage(obj.img, 0, 0)
        }
      }
    }
  },
  methods : {
    addImage(dir,file,index){
      let that = this;
      let payload = {
        index: index, 
        file: `${this.root}${dir}/${file}`,
      }
      that.layers[dir] = payload;
    },
    clearCtx(){
      this.ctx.clearRect(0,0,this.canvas.width,this.canvas.height);
      this.layers = {
        'accessories': {},
        'behind-body': {},
        'belt': {},
        'body': {},
        'cape': {},
        'ears': {},
        'eyes': {},
        'facial': {},
        'feet': {},
        'hair': {},
        'hands': {},
        'head': {},
        'legs': {},
        'nose': {},
        'shoulders': {},
        'torso': {},
        'weapons': {},
        'weapons-oversize': {}
      }
    }
  }
}
</script>

<style lang="stylus">
#app 
  font-family 'Avenir', Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  color #2c3e50
  margin-top 60px
  display flex
  height 100vh


.choices
  flex 1
  overflow-y scroll
  ul
    list-style-type none
    display flex
    flex-flow column wrap
    height 200px
  li
      cursor pointer
    span
      font-size 10px
.preview
  transform scale(0.6) translateY(-50px)
  transform-origin top
  img
    position absolute
</style>
