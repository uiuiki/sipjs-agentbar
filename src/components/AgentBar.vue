<template>
  <div class="tel-page">
    <div class="call-info" :class="{isCalling: isCalling}">
      <div id="number">{{remoteFsNum}}</div>
      <div id="duration">{{chatTimeText}}</div>

      <audio autoplay ref="remoteAudio" />
    </div>
    <div class="agent-bar">
      <div class="agent-bar-item" @click="showEndCallDialog = true" :class="{itemDisableClick: !isCalling}">
        <img src="@/assets/images/agentbar/guaduan.png" />
        <label>挂断</label>
      </div>
      <div class="agent-bar-item">
        <img src="@/assets/images/agentbar/zhengli.png" />
        <label>整理</label>
      </div>
      <div class="agent-bar-item">
        <img src="@/assets/images/agentbar/zixun.png" />
        <label>咨询</label>
      </div>
      <div class="agent-bar-item">
        <img src="@/assets/images/agentbar/sanfang.png" />
        <label>三方</label>
      </div>
      <div class="agent-bar-item">
        <img src="@/assets/images/agentbar/huiyi.png" />
        <label>会议</label>
      </div>
      <div class="agent-bar-item">
        <img src="@/assets/images/agentbar/zhuanyi.png" />
        <label>转移</label>
      </div>
      <div class="agent-bar-item" @click="requestInvite">
        <img src="@/assets/images/agentbar/waihu.png" />
        <label>外呼</label>
      </div>
      <div>
        <div class="agent-bar-item" v-if="!hasChecked" @click="checkIn">
          <img src="@/assets/images/agentbar/qianru.png" />
          <label>签入</label>
        </div>
        <div class="agent-bar-item" v-else @click="checkOut">
          <img src="@/assets/images/agentbar/qianchu.png" />
          <label>签出</label>
        </div>
      </div>
    </div>

    <el-dialog center :visible.sync="showCallingDialog" width="20%" :close-on-press-escape="false" :close-on-click-modal="false" :show-close="false">
      <div class="call-dialog">
        <el-input class="input-num" v-model="remoteFsNum" placeholder="请输入电话号码" :disabled="isInviting"/>
        <div class="dial-pad" v-if="!isInviting">
          <div class="dial-pad-item" v-for="v in 9" :key="v">
            <el-button class="dial-pad-item-digit" type="primary" circle @click="inputNumber">{{v}}</el-button>
          </div>
          <div class="dial-pad-item">
            <el-button class="dial-pad-item-digit" icon="el-icon-close"  type="primary" circle @click="clearInputNumber" />
          </div>
          <div class="dial-pad-item">
            <el-button class="dial-pad-item-digit" type="primary" circle @click="inputNumber">0</el-button>
          </div>
          <div class="dial-pad-item">
            <el-button class="dial-pad-item-digit" icon="el-icon-delete" type="primary" circle @click="deleteInputNumber" />
          </div>
        </div>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="showCallingDialog = false" :disabled="isInviting">取消</el-button>
        <el-button type="success" @click="call" v-if="!isInviting" :disabled="!enableInvite">呼叫</el-button>
        <el-button type="danger" @click="cancelCall" v-if="isInviting">取消呼叫</el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="showBellingDialog" width="500px" top="30px" class="answer" :modal="false" :close-on-press-escape="false" :close-on-click-modal="false" :show-close="false">
      <!-- <span>{{ remoteFsNum }} 来话</span> -->
      <span slot="title" class="time-out" @click="timeOut">稍后接听</span>
      <div class="a-info">
        <div class="number">{{ remoteFsNum || 13888888888 }}</div>
        <div class="area">贵州 贵阳 电话</div>
        <div class="ivr">IVR转入</div>
        <div class="waiting">等待接听...</div>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="success" size="small" @click="accept" style="margin-right:30px;">接受</el-button>
        <el-button type="danger" size="small" @click="reject">拒绝</el-button>
      </span>
    </el-dialog>

    <el-dialog center :visible.sync="showEndCallDialog" width="30%" :close-on-press-escape="false" :close-on-click-modal="false" :show-close="false">
      <span>确定挂断电话吗</span>
      <span slot="footer" class="dialog-footer">
          <el-button @click="showEndCallDialog = false">取消</el-button>
          <el-button @click="endCall">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import { UA } from 'sip.js'
import { freeSWITCHUserDomain, freeSWITCHWebSocketAddr, stunServerAddr } from '../config'

export default {
  name: 'telTraffic',
  data () {
    return {
      fsAccount: {
        account: '1015',
        password: 'xiaoi123'
      },
      ua: null, // sip.js UA
      remoteFsNum: '', // 对方 fs 号码
      session: null, // sip.js Session
      hasChecked: false, // 是否已经签入
      showCallingDialog: false, // 显示外呼对话框
      showBellingDialog: false,
      showEndCallDialog: false,

      isInviting: false,
      isCalling: false,

      chatTimeText: '00:00:00', // 通话时长
      chatTimeHour: 0, // 通话时长 - 时
      chatTimeMinute: 0, // 通话时长 - 分
      chatTimeSeconds: 0, // 通话时长 - 秒
      chatTimeIntervalNumber: -1
    }
  },
  beforeCreate () {
    console.log('beforeCreate')
  },
  created () {
    console.log('created')
  },
  beforeMount () {
    console.log('beforeMount')
  },
  mounted () {
    console.log('mounted')
  },
  beforeDestroy () {
    console.log('beforeDestroy')
  },
  destroyed () {
    console.log('destroyed')

    // // 需要签出
    // this.checkOut()
    //
    // // 停止通话计时并重置通话时长
    // this.stopChatTimeInterval()
  },
  computed: {
    enableInvite () {
      return !!this.remoteFsNum
    }
  },
  methods: {
    timeOut () {
      this.$store.commit('isShowDialog', {
        isTimeOut: true,
        remoteFsNum: this.remoteFsNum
      })
    },
    // MARK: - 所有的状态
    // 状态：IDL 状态
    onIDL () {
      this.hasChecked = true

      this.isInviting = false
      this.isCalling = false

      this.showCallingDialog = false
      this.showBellingDialog = false
      this.showEndCallDialog = false

      // this.remoteFsNum = ''
      this.session = null

      // 停止通话计时并重置通话时长
      this.stopChatTimeInterval()

      console.log('改变状态 --> onIDL')
    },
    // 状态：正在呼叫
    onInviting () {
      this.isInviting = true

      console.log('改变状态 --> onInviting')
    },
    // 状态：正在响铃
    onBelling () {
      this.showBellingDialog = true

      console.log('改变状态 --> onBelling')
    },
    // 状态：正在通话中
    onCall () {
      this.isInviting = false
      this.isCalling = true

      this.showCallingDialog = false
      this.showBellingDialog = false

      // 开始计时
      this.chatTimeIntervalNumber = setInterval(this.updateChatTimeInterval, 1000)

      console.log('改变状态 --> onCall')
    },
    // 状态：已签出
    onCheckout () {
      this.hasChecked = false

      console.log('改变状态 --> onCheckout')
    },
    // MARK: - 签入签出
    // 签入：先 start ，然后再 register --> 或者在 config 里面配置自动 register，这里采用这种方式
    checkIn () {
      // --- call start()
      let config = {
        uri: `${this.fsAccount.account}@${freeSWITCHUserDomain}`,
        transportOptions: {
          wsServers: freeSWITCHWebSocketAddr,
          traceSip: true
        },
        sessionDescriptionHandlerFactoryOptions: {
          peerConnectionOptions: {
            rtcConfiguration: {
              iceServers: [{ urls: stunServerAddr }]
            }
          }
        },
        authorizationUser: this.fsAccount.account,
        password: this.fsAccount.password,
        register: true
      }
      this.ua = new UA(config)

      // 调用 start() 连接到 WebSocket 服务器，但是不自动调用 register()
      this.ua.start()
      // --- done call start()

      // 签入成功
      this.ua.on('registered', () => {
        this.$message.success('签入成功')
        console.log('签入成功')

        if (!this.isCalling) {
          // 更新状态
          this.onIDL()
        }
      })
      // 签入失败
      this.ua.on('registrationFailed', (response, cause) => {
        this.$message.error(`签入失败: ${response}`)
        // console.error(`签入失败, response: ${response}, cause: ${cause}`)
        console.error('签入失败')
      })
      // 签出成功
      this.ua.on('unregistered', (response, cause) => {
        this.$message.success('签出成功')
        // console.log(`签出成功, response: ${response}, cause: ${cause}`)
        console.log('签出成功')
      })

      // 通话邀请进来
      this.ua.on('invite', session => {
        this.session = session
        // console.log(session)
        // this.remoteFsNum = session.remoteIdentity.friendlyName
        this.remoteFsNum = session.session.remoteURI.user

        this.session.on('cancel', () => {
          this.$message.info('对方已取消')
          console.log('对方已取消')

          // 更新状态
          this.onIDL()
        })
        this.session.on('accepted', (data) => {
          this.$message.info('呼叫已接受')
          // console.log(`建立通话成功: ${data}`)
          console.log('建立通话成功')

          // 更新状态
          this.onCall()
        })
        this.session.on('failed', (response, cause) => {
          // this.$message.error('建立通话失败')
          // console.error(`建立通话失败, response: ${response}, cause=${cause}`)
          console.error('建立通话失败')

          // 更新状态
          this.onIDL()
        })
        this.session.on('bye', (request) => {
          this.$message.info('通话已结束')
          // console.log(`通话已结束: ${request}`)
          console.log('通话已结束')

          // 更新状态
          this.onIDL()
        })
        this.session.on('terminate', (message, cause) => {
          this.$message.info('对方已挂断')
          // console.log(`对方已挂断,message : ${message}, cause=${cause}`)
          console.log('对方已挂断')

          // 更新状态
          this.onIDL()
        })

        this.session.on('trackAdded', () => {
          let pc = this.session.sessionDescriptionHandler.peerConnection

          // remoteStream
          let remoteStream = new MediaStream()
          pc.getReceivers().forEach(receiver => {
            remoteStream.addTrack(receiver.track)
          })
          this.$refs.remoteAudio.srcObject = remoteStream

          // // localStream
          // let localStream = new MediaStream()
          // pc.getSenders().forEach(sender => {
          //   localStream.addTrack(sender.track)
          // })
          // this.$refs.localAudio.srcObject = localStream
        })

        // 更新状态
        this.onBelling()
      })

      // // --- call register()
      // // 不能马上调用
      // setTimeout(() => {
      //   this.ua.register()
      // }, 2000)
      // // --- call register()

      // 发送消息
      // this.ua.message(`${this.fsAccount.remoteAccount}@${freeSWITCHUserDomain}`, "hello")

      this.ua.on('message', message => {
        console.log(`收到消息：${message.contentType} -- ${message.body}`)
      })

      console.log('调用方法：checkIn()')
    },
    // 签出：先 unregister ，然后再 stop
    checkOut () {
      if (this.ua) {
        this.ua.unregister()
        setTimeout(() => {
          this.ua.stop()
        }, 1000)

        // 更新状态
        this.onCheckout()

        console.log('调用方法：checkOut()')
      }
    },
    // MARK: - 外呼
    call () {
      if (this.ua.isRegistered()) {
        this.session = this.ua.invite(`${this.remoteFsNum}@${freeSWITCHUserDomain}`, {
          sessionDescriptionHandlerOptions: {
            constraints: {
              audio: true,
              video: false
            }
          }
        })

        // 更新状态
        this.onInviting()

        this.session.on('accepted', (data) => {
          this.$message.success('对方已接受')
          // console.log(`对方已接受: ${data}`)
          console.log('对方已接受')

          // 更新状态
          this.onCall()
        })
        this.session.on('rejected', (response, cause) => {
          this.$message.info('对方已拒绝')
          // console.log(`对方已拒绝, response: ${response}, cause=${cause}`)
          console.log('对方已拒绝')

          // 更新状态
          this.onIDL()
        })
        this.session.on('failed', (response, cause) => {
          this.$message.error('呼叫失败')
          // console.log(`呼叫失败, response: ${response}, cause=${cause}`)
          console.log('呼叫失败')

          // 更新状态
          this.onIDL()
        })
        this.session.on('bye', (request) => {
          this.$message.info('对方已挂断')
          // console.log(`对方已挂断: ${request}`)
          console.log('对方已挂断')

          // 更新状态
          this.onIDL()
        })
        this.session.on('terminate', (message, cause) => {
          this.$message.info('通话已结束')
          // console.log(`通话已结束,message : ${message}, cause=${cause}`)
          console.log('通话已结束')

          // 更新状态
          this.onIDL()
        })

        this.session.on('trackAdded', () => {
          let pc = this.session.sessionDescriptionHandler.peerConnection

          // remoteStream
          let remoteStream = new MediaStream()
          pc.getReceivers().forEach((receiver) => {
            remoteStream.addTrack(receiver.track)
          })
          this.$refs.remoteAudio.srcObject = remoteStream

          // // localStream
          // let localStream = new MediaStream()
          // pc.getSenders().forEach((sender) => {
          //   localStream.addTrack(sender.track)
          // })
          // this.$refs.localAudio.srcObject = localStream
        })
      } else {
        this.$message.warning('请先签入')
      }
    },
    cancelCall () {
      if (this.session) {
        this.session.cancel()
        this.$message.info('通话已取消')
        console.log('通话已取消')

        // 更新状态
        this.onIDL()
      }
    },
    endCall () {
      if (this.session) {
        this.session.terminate()
        this.$message.info('通话已结束')
        console.log('通话已结束')

        // 更新状态
        this.onIDL()
      }
    },
    // MARK: - 呼入
    accept () {
      this.session.accept()
      // this.$message.info('呼叫已接受')
      //
      // // 更新状态
      // this.onCall()
    },
    reject () {
      this.session.reject()
      this.$message.info('呼叫已拒绝')
      console.log('呼叫已拒绝')

      // 更新状态
      this.onIDL()
    },
    // MARK: - 拨号键盘相关方法
    // 打开拨号键盘
    requestInvite () {
      if (this.isCalling) {
        this.$message.warning('正在通话中')
      } else if (!this.ua || !this.ua.isRegistered()) {
        this.$message.warning('请先签入')
      } else {
        this.showCallingDialog = true
      }
    },
    // 输入单个数字
    inputNumber (e) {
      this.remoteFsNum += e.target.innerText
    },
    // 清除输入的数字
    clearInputNumber () {
      this.remoteFsNum = ''
    },
    // 删除一个的数字
    deleteInputNumber () {
      if (this.remoteFsNum) {
        this.remoteFsNum = this.remoteFsNum.substring(0, this.remoteFsNum.length - 1)
      }
    },
    // MARK: - 通话时长相关方法
    // 定时更新通话时长
    updateChatTimeInterval () {
      this.chatTimeSeconds += 1
      if (this.chatTimeSeconds >= 60) {
        this.chatTimeSeconds = 0
        this.chatTimeMinute += 1
      }
      if (this.chatTimeMinute >= 60) {
        this.chatTimeMinute = 0
        this.chatTimeHour += 1
      }

      // 小于 10 补 0
      let h = this.chatTimeHour < 10 ? `0${this.chatTimeHour}` : this.chatTimeHour
      let m = this.chatTimeMinute < 10 ? `0${this.chatTimeMinute}` : this.chatTimeMinute
      let s = this.chatTimeSeconds < 10 ? `0${this.chatTimeSeconds}` : this.chatTimeSeconds

      this.chatTimeText = `${h}:${m}:${s}`
    },
    // 停止通话计时并重置通话时长
    stopChatTimeInterval () {
      clearInterval(this.chatTimeIntervalNumber)

      this.chatTimeText = '00:00:00'
      this.chatTimeHour = 0
      this.chatTimeMinute = 0
      this.chatTimeSeconds = 0
    }
  }
}
</script>
<style lang="scss" scoped>
  $common-margin: 17px;
  .tel-page {
    width: 100%;
    height: 53px;
    background-color: #F1F2F3;
    display: flex;
    align-items: center;

    .isCalling {
      background-color: #DDE5F4;
    }

    .call-info {
      height: 100%;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      padding-left: 14px;
      padding-right: 14px;
      margin-left: 2 * $common-margin;
      margin-right: $common-margin;

      #number {
        font-size: 20px;
      }
      #duration {
        font-size: 12px;
      }
    }
    .agent-bar {
      width: 50%;
      display: flex;
      align-items: center;
      cursor: pointer;

      .agent-bar-item {
        display: flex;
        flex-direction: column;
        margin-left: $common-margin;
        margin-right: $common-margin;
      }

      .itemDisableClick {
        pointer-events: none;
      }
    }

    .call-dialog {
      display: flex;
      flex-direction: column;
      align-items: center;

      .input-num {
        width: 70%;
      }

      .dial-pad {
        width: 70%;
        margin-top: 16px;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: wrap;

        .dial-pad-item {
          margin: 14px;

          .dial-pad-item-digit {
            width: 40px;
            height: 40px;
          }
        }
      }
    }
  }
  .answer {
    .el-button {
      width: 110px;
    }
  }
</style>
<style>
  .answer .el-dialog {
    background: #263238;
    color: #FFFFFF !important;
  }
  .answer .el-dialog__footer {
    display: flex;
    justify-content: space-around;
    height: 70px;
  }
</style>
