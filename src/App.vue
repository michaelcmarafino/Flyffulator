<template>
  <div class="mainpage">
    <div class="tooltip">
      <h5>info</h5>
    </div>
    <leftbar/>

    <div class="themetoggle"><img :src="getIconUrl(themeImage)" @click="toggleTheme"></div>
    
    <div class="content">
      <div class="basestats">
        <img :src="getImageUrl(character.ref.weapon_img)"/>
        <!--<top-stats/>-->
      </div>

      <div class="extensivestats">
        
        <h3>Leveling</h3>
        <div class="extensiverow">
          <experience-per-kill/>
          <kills-per-level/>
          <exp-hp-ratio/>
        </div>

        <div class="extensiverow">
          <hits-per-level/>
        </div>
        
        <h3>Damage</h3>
        <div class="extensiverow">
          <damage-box title="Average AA" skillindex=-1 />
          <damage-box title="N/A" skillindex=0 />
          <damage-box title="N/A" skillindex=1 />
          <damage-box title="N/A" skillindex=2 />
        </div>

        <div class="extensiverow">
          <damage-specifics/>
          <auto-ratio/>
        </div>

        <div class="footer">
          <h5>Flyff simulator built specifically for <a href='https://flyff-api.sniegu.fr/'>Flyff Universe</a> by Frostiae#2809</h5>
          <h5 style="opacity: 0.5;">If you would like to follow development or contribute, visit the <a href="https://github.com/Frostiae/Flyffulator">GitHub</a> page.</h5>
        </div>
      </div>
    </div>

    <rightbar/>
  </div>
</template>

<script>
/* eslint-disable */
import ExperiencePerKill from './components/ExperiencePerKill.vue'
import KillsPerLevel from './components/KillsPerLevel.vue'
import ExpHpRatio from './components/ExpHpRatio.vue'
import DamageBox from './components/DamageBox.vue'
import DamageSpecifics from './components/DamageSpecifics.vue'
import AutoRatio from './components/AutoRatio.vue'
import HitsPerLevel from './components/HitsPerLevel.vue'
import TopStats from './components/TopStats.vue'
import Leftbar from './components/Leftbar.vue'
import Rightbar from './components/Rightbar.vue'
import { Utils } from './calc/utils.js'
import { Billposter, Vagrant } from './calc/jobs'

const utils = new Utils()

export default {
  name: 'App',
  components: {
    ExperiencePerKill,
    TopStats,
    KillsPerLevel,
    HitsPerLevel,
    ExpHpRatio,
    DamageBox,
    AutoRatio,
    DamageSpecifics,
    Leftbar,
    Rightbar
  },
  data() {
    return {
      darkMode: true,
      themeImage: 'sunstone.png',
      componentbg: '#262626',
      sidepanelbg: '#1d1d1d',
      hcolor: '#F2F2F2',
      pcolor: '#A67041',
      mainbg: 'radial-gradient(ellipse at top left, #1b1b1b 40%, #292929) 100%)',
      mainbg1: '#1b1b1b',
      mainbg2: '#292929',
      character: {
        ref: utils.character
      },
      monsters: utils.getMonstersAtLevel(utils.character.level),
      focusMonster: null,
      jobName: "Vagrant", // This is used only when a character is imported, because we have no other way of changing the select field
      skillIndex: -1    // Auto attack is used initially for all damage calculations
    }
  },
  watch: {
    darkMode() {
      if (this.darkMode) {
          this.componentbg = '#262626'
          this.sidepanelbg = '#1d1d1d'
          this.hcolor = '#F2F2F2'
          this.pcolor = '#A67041'
          this.mainbg1 = '#1b1b1b'
          this.mainbg2 = '#292929'
      } else {
          this.componentbg = '#2e325c'
          this.sidepanelbg = '#1f2342'
          this.hcolor = '#7279aa'
          this.pcolor = '#dadeef'
          this.mainbg1 = '#252849'
          this.mainbg2 = '#1c1e3a'
      }
    },
    'character.ref.level'() { this.updateCharacter() },
    'character.ref.str'() { this.updateCharacter() },
    'character.ref.sta'() { this.updateCharacter() },
    'character.ref.dex'() { this.updateCharacter() },
    'character.ref.int'() { this.updateCharacter() },
    'character.ref.assistInt'() { this.updateCharacter() },
    'character.ref.selfBuffs'() { this.updateCharacter() },
    'character.ref.assistBuffs'() { this.updateCharacter() },
    'character.ref.premiumItems'() { this.updateCharacter() },
    'character.ref.mainhand'() { this.updateCharacter() },
    'character.ref.offhand'() { this.updateCharacter() },
    'character.ref.armor'() { this.updateCharacter() },
    'character.ref.armorUpgrade'() { this.updateCharacter() },
    'character.ref.mainhandUpgrade'() { this.updateCharacter() },
    'character.ref.offhandUpgrade'() { this.updateCharacter() },
    'character.ref.earringR'() { this.updateCharacter() },
    'character.ref.earringL'() { this.updateCharacter() },
    'character.ref.ringR'() { this.updateCharacter() },
    'character.ref.ringL'() { this.updateCharacter() },
    'character.ref.cloak'() { this.updateCharacter() },
    'character.ref.necklace'() { this.updateCharacter() },
    'character.ref.suitPiercing'() { this.updateCharacter() },
    'character.ref.shield'() { this.updateCharacter() },
  },
  created() { this.updateCharacter() },
  methods: {
    toggleTheme() {
      this.darkMode = !this.darkMode
      this.themeImage = this.darkMode ? 'sunstone.png' : 'moonstone.png'
    },
    getExpReward(monster, level) {
      // TODO: Get rid of this function
      return monster.experienceTable[level - 1];
    },
    updateJob() {
      let c = utils.updateJob(this.character.ref, this.jobName)

      if (c) {
        this.character.ref = c
        this.updateCharacter()
      }
    },
    updateCharacter() {
      validateInput(this.character.ref)
      this.character.ref.update()

      // Need to update the character in utils since that is what utils uses
      utils.character = this.character.ref
      this.monsters = utils.getMonstersAtLevel(this.character.ref.level, this.skillIndex)
      this.focusMonster = this.monsters.find(monster => monster.level >= this.character.ref.level) || this.monsters.slice(-1)[0];
      this.character.ref.monsters = this.monsters;
    },
    updateMonsters(index) {
      this.skillIndex = index
      if (this.skillIndex == -1) {
        this.monsters = utils.getMonstersAtLevel(this.character.ref.level, null);
        this.character.ref.monsters = this.monsters;
      } else {
        this.monsters = utils.getMonstersAtLevel(this.character.ref.level, this.skillIndex);
        this.character.ref.monsters = this.monsters;
        this.skillIndex = index;
      }
      
      this.focusMonster = this.monsters.find(monster => monster.level >= this.character.ref.level) || this.monsters.slice(-1)[0];
    },
    getImageUrl(img) {
      var images = require.context('./assets/images/', false, /\.png$/)
      return images('./' + img)
    },
    getIconUrl(img) {
      try {
        var images = require.context('./assets/images/Icons/Items', false, /\.png$/)
        return images('./' + img)
      } catch(error) {
        // dummy icon if icon couldn't be found
        return images('./' + "syssysqueheadrb.png")
      }
    },
    getSkillIconUrl(img) {
      var images = require.context('./assets/images/Icons/Skills/colored', false, /\.png$/)
      return images('./' + img)
    }
  }
}

function validateInput(character) {
  character.level = character.level < 1 ? 1 : character.level;
  character.level = character.level > 120 ? 120 : character.level;
  character.str = character.str < 1 ? 1 : character.str;
  character.sta = character.sta < 1 ? 1 : character.sta;
  character.dex = character.dex < 1 ? 1 : character.dex;
  character.int = character.int < 1 ? 1 : character.int;
  character.assistInt = character.assistInt < 1 ? 1 : character.assistInt;
}
</script>

<style lang='scss'>
* {
  box-sizing: border-box;
}

.themetoggle {
  position: fixed;
  margin-top: 20px;
  margin-left: 350px;
  transition: 0.3s;
  border-radius: 20px;
  box-shadow: #0000001e 0px 0px 0px;
  padding: 5px;
}

.themetoggle:hover {
  cursor: pointer;
  background-color: #70707020;
  box-shadow: #0000001e 0px 5px 20px;
  border-radius: 35px;
}

.mainpage {
  height: 100vh;
  width: 100vw;
  background: radial-gradient(ellipse at top left, v-bind(mainbg1) 40%, v-bind(mainbg2) 100%);
  transition: 0.3s;
}

.stats {
  background-color: v-bind(componentbg);
  transition: 0.3s
}

.sidepanel {
  background-color: v-bind(sidepanelbg);
}

.rightpanel {
  background-color: v-bind(sidepanelbg);
}

.btn-plus {
  color: v-bind(pcolor);
}

body, html {
  overflow: hidden;
  margin: 0;
}

a {
  color: #008ffb;
  text-decoration: none;
  transition: 0.3s;
}

a:hover {
  text-decoration: underline;
  transition: 0.3s;
}

.footer {
  margin-top: 50px;
  padding-bottom: 10px;
}

hr {
  border-color: #F2F2F2;
  opacity: 0.5;
  margin: 8px;
}

p {
  color: v-bind(pcolor);
  margin: 3px;
  transition: 0.3s;
}

h1, h2, h3, h4, h5 {
  color: v-bind(hcolor);
  font-weight: 500;
  margin-bottom: 8px;
  transition: 0.3s;
}

h5 {
  margin: 3px;
}

ul, ol {
  font-size: 13px;
  color: v-bind(hcolor);
  transition: 0.3s;
  font-weight: 500;
}

img#bg-radial {
  position: absolute;
  width: 550px;
  display: block;
  margin-right: 50%;
  margin-left: 20%;
  margin-top: -120px;
  opacity: 0.8;
  z-index: 0;
}

option {
  color: black;
}

.content {
  display: flex;
  flex-direction: column;
  padding-top: 20px;
  overflow-y: auto;
  overflow-x: hidden;
  height: 100vh;
  margin-left: auto;
  margin-right: auto;
  width: 50vw;
  scrollbar-width: none;

  .basestats {
    display: flex;
    flex-direction: row;
    justify-content: space-evenly;

    img {
      width: 313px;
      height: 313px;
    }
  }

  .extensivestats {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
    margin: 30px;
    margin-top: -20px;
    padding-bottom: 20px;

    .extensiverow {
      display: flex;
      flex-direction: row;
      margin-top: 10px;
      padding-bottom: 10px;
      flex-wrap: wrap;

      .extensivebasic {
        display: flex;
        flex-direction: column;
        background-color: v-bind(componentbg);
        width: 200px;
        height: 180px;
        border-radius: 20px;
        box-shadow: #0000001e 0px 5px 5px;
        transition: 0.3s;
        margin-right: 10px;
        margin-bottom: 10px;

        p {
          font-size: 35px;
          font-weight: 500;
          margin-left: 20px;
        }

        h3, h5 {
          margin-left: 20px;
        }
      }

      .extensivechart {
        background-color: v-bind(componentbg);
        height: 180px;
        width: 300px;
        border-radius: 20px;
        box-shadow: #0000001e 0px 5px 5px;
        transition: 0.3s;
        margin-right: 10px;
        margin-bottom: 10px;
      }

      .big-chart-disclaimer {
        display: block;
      }

      .extensivechart#big {
        display: none;
        background-color: v-bind(componentbg);
        height: 400px;
        width: 820px;
        border-radius: 20px;
        box-shadow: #0000001e 0px 5px 5px;
        transition: 0.3s;
        margin-right: 10px;

        h1 {
          color: v-bind(pcolor);
        }
      }

      @media only screen and (min-width: 1450px) {
        .extensivechart#big {
          display: block;
        }

        .big-chart-disclaimer {
          display: none;
        }
      }

      .extensivechart#big:hover {
        box-shadow: #0000001e 0px 10px 20px;
        transition: 0.3s;
      }

      .extensivechart#hover {
        box-shadow: #0000001e 0px 10px 20px;
        transition: 0.3s;
      }

      .extensivebasic:hover {
        box-shadow: #0000001e 0px 10px 20px;
        transition: 0.3s;
        transform: scale(1.05);
        cursor: pointer;
      }
    }
  }
}

@media only screen and (min-width: 1450px) {
  .content {
    width: 60vw;
  }
}

input[type=number], input[type=text] {
  -moz-appearance: textfield;
  appearance: textfield;
  background: none;
  border: none;
  width: 50px;
  text-align: center;
  color: v-bind(pcolor);
  font-weight: 500;
  font-family: 'Roboto';
  margin: 2px;
}

.tooltip {
  display: none;
  position: absolute;
  background-color: v-bind(sidepanelbg);
  border-radius: 10px;
  width: 275px;
  box-shadow: 3px 3px 10px rgba(0, 0, 0, 0.407);
  color: #5975cf;
  padding: 15px;
  z-index: 99;
}

p#info {
  display: none;
}

.info-tooltip {
  position: absolute;
  right: 0;
  margin-right: 20px;
  margin-top: 22px;
  transition: 0.3s;
  border-radius: 15px;
  width: 22px;
  opacity: 0.8;
  z-index: 99;
}

.info-tooltip:hover {
  box-shadow: 3px 3px 5px rgba(0, 0, 0, 0.307);
  opacity: 0.5;
  transition: 0.3s;
}

input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
}

input[type=checkbox]#buffs {
  align-self: center;
}

#app {
  font-family: 'Roboto', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height: 100vh;
  width: 100vw;
  overflow-x: hidden;
}

::-webkit-scrollbar {
  width: 12px;
}

::-webkit-scrollbar-track {
  background: none;
}

::-webkit-scrollbar-thumb {
  background-color: v-bind(sidepanelbg);
  border-radius: 15px;
  border: 3px solid v-bind(componentbg);
  margin: 10px;
}
</style>
