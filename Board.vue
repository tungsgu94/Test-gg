<template>
    <div>
       <!--   <div class="title-aaa"> Please choose 2 pieces of shapes to swap positions and then finish the original picture.</div> -->
   
    <div class="board">
        <div class="back" ref="board">
            <div class="frame-wrapper" :style="frameSize">
                <div class="original"
                    v-if="showingOriginal && image"
                    @click="showingOriginal = false"
                    :style="{ background: `url(${image})`}">
                </div>
                <div class="frame" :style="frameSize">
                    <Tile v-for="tile in tiles"
                        :key="tile.position"
                        :tile="tile"
                        @moving="moveTile"
                        ref="tiles"
                    />
                </div>
            </div>
        </div>
        <!-- <div class="controls" style="display: none;">
            <a class="toggle-original" href="#" @click.prevent="showingOriginal = !showingOriginal">
                Toggle Original Image
            </a>
            <a class="shuffle" href="#" @click.prevent="shuffleTiles">Reshuffle</a>
         <a class="reset" href="#" @click.prevent="reset">Reset</a> 
        </div> -->
        <div class="list-action" v-show="!gameOver">
            <div class="reset" @click.prevent="shuffleTiles"><img src="../assets/img/reset.png"></div>
            <div class="view" @click.prevent="showingOriginal = !showingOriginal"><img src="../assets/img/view.png"></div>
        </div>

        <div class="box-reward" v-show="gameOver">
             <div class="reward-contai">
                 <div class="header">
                        <div class="t">GOLD REWARD</div>
                        <div class="close"></div>
                   </div>
            </div>
        </div>
    </div>
      <gameFinish  v-if="gameF"/>
     </div>
</template>

<script>
import _ from 'lodash'
import Tile from './Tile'
import loadImage from 'blueimp-load-image'
import gameFinish from "./Finish"
import { mapGetters, mapActions } from 'vuex'
let backupTiles = null
export default {
    components: { Tile,  gameFinish},
    data () {
        return {
            showingOriginal: false,
            tiles: [],
            tileSize: {
              width: 0,
              height: 0
            },
            image: null,
            gameF:false,
            gameOver: false,
            count: 0
        }
    },
    props: {
        img: {
            type: String,
            required: true
        },
        game_id: {
            type: Number,
            required: true
        }
    },
    watch: {
        "valid": function() {
            var t = this
            if (this.valid) {
                if (this.getWithFriend == 1) {
                    this.customInvite()
                    this.set_with_friend(0)
                }else if (this.getWithFriend == 2) {
                    this.customChallenge()
                    this.set_with_friend(0)
                }

                if (this.getFrom == "anonymous") {
                    switch(this.totalTiles){
                        case 3*3:
                            this.set_coins(this.getCoins + 100)
                            window.FBInstant.player.setDataAsync ({
                                data: t.getDataGame
                            });
                            break;
                        case 4*4:
                            this.set_coins(this.getCoins + 200)
                            window.FBInstant.player.setDataAsync ({
                                data: t.getDataGame
                            });
                            break;
                        case 5*5:
                            this.set_coins(this.getCoins + 300)
                            window.FBInstant.player.setDataAsync ({
                                data: t.getDataGame
                            });
                            break;
                        case 6*6:
                            this.set_coins(this.getCoins + 400)
                            window.FBInstant.player.setDataAsync ({
                                data: t.getDataGame
                            });
                            break;
                        case 7*7:
                            this.set_coins(this.getCoins + 500)
                            window.FBInstant.player.setDataAsync ({
                                data: t.getDataGame
                            });
                            break;
                        case 8*8:
                            this.set_coins(this.getCoins + 600)
                            window.FBInstant.player.setDataAsync ({
                                data: t.getDataGame
                            });
                            break;
                    }
                }
                this.$emit("gameDone")
                this.gameF = true;
                this.gameOver = true
                this.showingOriginal = true
                setTimeout(function(){ this.gameF = false }.bind(this), 3000)
            }
        }
    },
    computed: {
        ...mapGetters(["getUserInfo", "getFrom", "getWithFriend", "getEntry", "horizontal", "vertical", "getCoins", 'getDataGame']),
        frameSize () {
          return {
            width: `${this.tileSize.width * this.horizontal}px`,
            height: `${this.tileSize.height * this.vertical}px`
          }
        },
        totalTiles () {
          return this.horizontal * this.vertical
        },
        valid () {
            if (!this.tiles.length) {
                return false
            }
            for (let i = 0; i < this.totalTiles; ++i) {
                // console.log(this.tiles[i].styles.order, this.tiles[i].position, i)
                if (this.tiles[i].styles.order != this.tiles[i].position) {
                   
                    return false;
                }
            }
            return true
        },
    },
    mounted: function() {
        this.start()
    },
    methods: {
        ...mapActions(["set_with_friend", "set_coins"]),
        start () {
            var t = this
            const img = new Image()
            loadImage(t.img, canvas => {
                t.image = canvas.toDataURL()
                t.tileSize.width = Math.floor(canvas.width / t.horizontal - 5)
                t.tileSize.height = Math.floor(canvas.height / t.vertical)
                t.generateTiles()
                t.shuffleTiles()
            }, {
                maxWidth: t.$refs.board.clientWidth,
                maxHeight: 600,
                minWidth: 200,
                minHeight: 200,
                canvas: true
            })
            
        },
        generateTiles () {
            this.tiles = []
            for (let i = 0; i < this.totalTiles; ++i) {
                this.tiles.push({
                    styles: {
                        background: i === this.totalTiles-1 ? 'transparent' : `url(${this.image})`,
                        backgroundPositionX: '-' + (i % this.horizontal) * this.tileSize.width + 'px',
                        backgroundPositionY: '-' + Math.floor(i / this.horizontal) * this.tileSize.height + 'px',
                        width: `${this.tileSize.width}px`,
                        height: `${this.tileSize.height}px`,
                        order: i
                    },
                    position: i,
                    isEmpty: i === this.totalTiles-1
                })
            }
            
        },
        shuffleTiles () {
            for (let i = 0, j = this.totalTiles * 5; i < j; ++i) {
                const emptyTile = this.tiles.find(t => t.isEmpty)
                const movableTiles = this.tiles.filter(t => {
                    return this.getAdjacentOrders(emptyTile).indexOf(t.styles.order) > -1
                })
                this.switchTiles(emptyTile, _.sample(movableTiles))
            }
            backupTiles = JSON.stringify(this.tiles)
        },

        moveTile (tile) {
            console.log("move")
            if (tile.isEmpty) {
                return
            }
            const target = this.tiles.find(t => {
                return t.isEmpty && this.getAdjacentOrders(tile).indexOf(t.styles.order) > -1
            })
            target && this.switchTiles(target, tile), this.count++
        },
        switchTiles (a, b) {
            [a.styles.order, b.styles.order] = [b.styles.order, a.styles.order]
        },
        customInvite: function(){
            var t = this
            window.FBInstant.updateAsync({
                action: 'CUSTOM',
                cta: 'Play',
                image: this.img,
                text: {
                    default: `${t.getUserInfo.name} has completed ${t.horizontal * t.vertical} puzzle pieces in ${t.count} times of arrangements`,
                    localizations: {
                        vi_VN: `${t.getUserInfo.name} đã hoàn thành ${t.horizontal * t.vertical} mảnh ghép trong ${t.count} lần sắp xếp`,
                        en_US: `${t.getUserInfo.name} has completed ${t.horizontal * t.vertical} puzzle pieces in ${t.count} times of arrangements`,
                        in_ID: `${t.getUserInfo.name} telah menyelesaikan ${t.horizontal * t.vertical} potongan puzzle dalam ${t.count} kali pengaturan`,
                        ar_AR: `أكمل ${t.getUserInfo.name} قطع الألغاز ${t.horizontal * t.vertical} في ${t.count} مرات من الترتيبات`,
                        th_TH: `${t.getUserInfo.name} ทำชิ้นส่วนปริศนาครบ ${t.horizontal * t.vertical} ใน ${t.count} ครั้ง`,
                        es_US: `${t.getUserInfo.name} ha completado ${t.horizontal * t.vertical} piezas del rompecabezas en ${t.count} tiempos de arreglos`
                    }
                },
                template: 'VILLAGE_INVASION',
                data: {
                    mess: {
                        id: t.game_id,
                        playerId: t.getUserInfo.id,
                        playerName: t.getUserInfo.name,
                        playerPic: t.getUserInfo.picture,
                        count: t.count
                    }
                },
                strategy: 'IMMEDIATE',
                notification: 'NO_PUSH',
            }).then(function() {
                console.log("send success")
            });
        },
        customChallenge: function(){
            var t = this
            window.FBInstant.updateAsync({
                action: 'CUSTOM',
                cta: 'Challenge',
                image: this.img,
                text: {
                    default: `${t.getUserInfo.name} has finished ${(t.getEntry.mess.count > t.count) ? 'faster than you' : 'slower than you' }. Let you challenge it again!`,
                    localizations: {
                        vi_VN: `${t.getUserInfo.name} đã sắp xếp ${(t.getEntry.mess.count > t.count) ? 'nhanh hơn bạn' : 'chậm hơn bạn' }. Thử thách lại nào!`,
                        en_US: `${t.getUserInfo.name} has finished ${(t.getEntry.mess.count > t.count) ? 'faster than you' : 'slower than you' }. Let you challenge it again!`,
                        in_ID: `${t.getUserInfo.name} telah menyelesaikan ${(t.getEntry.mess.count> t.count) ? 'lebih cepat dari Anda' : 'lebih lambat dari Anda'}. Biarkan Anda menantangnya lagi!`,
                        th_TH: `${t.getUserInfo.name} เสร็จสิ้น ${(t.getEntry.mess.count> t.count) ? 'เร็วกว่าคุณ' : 'ช้ากว่าคุณ'} ให้คุณท้าทายอีกครั้ง!`,
                        es_US: `${t.getUserInfo.name} ha terminado ${(t.getEntry.mess.count> t.count) ? 'más rápido que tú' : 'más lento que tú'}. ¡Deja que lo desafíes de nuevo!`
                    }
                },
                template: 'VILLAGE_INVASION',
                data: {
                    mess: {
                        id: t.game_id,
                        playerId: t.getUserInfo.id,
                        playerName: t.getUserInfo.name,
                        playerPic: t.getUserInfo.picture
                    },
                    send: {
                        playerId: t.getEntry.mess.playerId,
                        playerName: t.getEntry.mess.playerName,
                        playerPic: t.getEntry.mess.playerPic
                    }
                },
                strategy: 'IMMEDIATE',
                notification: 'NO_PUSH',
            }).then(function() {
                console.log("send success")
            });
        },
        getAdjacentOrders (tile) {
            const pos = tile.styles.order
            return [
                pos % this.horizontal ? pos - 1 : null,
                (pos + 1) % this.horizontal ? pos + 1 : null,
                pos - this.horizontal,
                pos + this.horizontal
            ]
        },
        reset () {
            this.tiles = JSON.parse(backupTiles)
        },
    }
}
</script>

<style lang="scss">
.frame-wrapper {
  margin: 0 auto;
  position: relative;
 
  .original {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
  }
  p.win {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    background: rgba(0, 0, 0, .5);
    color: #fff;
    font-size: 40px;
    margin: 0 0;
    background: rgba(43, 181, 82, 0.7);
    text-transform: uppercase;
  }
}
.frame {
  display: flex;
  flex-wrap: wrap;
  background: #612211 url('../assets/board.jpg');
  background-size: cover;
}
.controls {
  margin-top: 30px;
  a {
    display: inline-block;
    text-decoration: none;
    padding: 6px 12px;
    background: #f78403;
    color: #fff;
    border-radius: 3px;
    &.toggle-original {
      background: #d05b88;
    }
    &.restart {
      background: #368ba0;
    }
    &.shuffle {
      background: #3ebb5c;
    }
  }
}
</style>