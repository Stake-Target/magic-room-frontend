<template>
  <div :class="$style.room">
    <div :class="$style.item" v-for="(chair, i) in chairs" :key="chair.index" ref="poufs" @click="onTouch(i)">
      <div :class="$style.chair">
        <ui-pouf
          :class="$style.pouf"
          :priority="chair.isPriority"
          :my="chair.isMy"
          :empty="chair.isEmpty"
        />
        <div :class="$style.name" v-if="room.names[i]">{{ room.names[i] }}</div>
        <div :class="$style.events">
          <transition-group :name="$style.events" tag="div">
            <div :class="$style.event" v-for="event in events[i]" :key="event.id">
              <span :class="$style.icon">
                <template v-if="event.type === 'Vip'">🏵</template>
                <template v-else-if="event.type === 'Reward'">🏅</template>
                <template v-else-if="event.type === 'Winner'">🏆</template>
              </span>
              <span>+</span>
              <img :class="$style.coin" src="~/assets/images/coin.png" alt="" />
              <span>{{ event.value | number }}</span>
              <span :class="$style.currency">MGT</span>
            </div>
          </transition-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import UiPouf from "~/components/ui/pouf"

export default {
  name: 'SectionRoomChairs',
  components: {
    UiPouf
  },
  data () {
    return {
      events: [
        [], [], [], [], [], [], [], [], [], []
      ]
    }
  },
  computed: {
    ...mapState('account', ['account']),
    ...mapState('room', ['room']),
    chairs () {
      const emptyAddress = '0x0000000000000000000000000000000000000000'
      return this.room.chairs.map((chair, i) => {
        return {
          index: i,
          isPriority: [1, 2, 3].includes(i),
          isEmpty: emptyAddress === chair,
          isMy: this.account && this.account.address.toUpperCase() === chair.toUpperCase(),
          address: chair
        }
      })
    }
  },
  methods: {
    addEvent (event) {
      const id = Math.random()
      const chairIndex = event.data.reward.index
      this.events[chairIndex].push({
        id,
        type: event.event,
        value: event.data.reward.value
      })
      setTimeout(() => {
        const index = this.events[chairIndex].findIndex(i => i.id === id)
        this.events[chairIndex].splice(index, 1)
      }, 4000)
    },
    onTouch (index) {
      this.$refs.poufs[index].classList.add('touched')
      setTimeout(() => {
        this.$refs.poufs[index].classList.remove('touched')
      }, 200)
    }
  },
  mounted() {
    this.$web3.game.addEventsListener((event) => {
      if (['Reward', 'Vip', 'Winner'].includes(event.event)) {
        this.addEvent(event)
      }
    })
    // setInterval(() => {
    //   this.addEvent({
    //     event: 'Vip',
    //     data: {
    //       chairIndex: 1,
    //       value: 10
    //     }
    //   })
    // }, 1200)
  }
}
</script>

<style lang="scss" module>
.room {
}
.item {
  position: absolute;
  width: 28%;
  user-select: none;
  transition: transform 0.4s cubic-bezier(0.65, 2.28, 0.42,-0.12);
  &:global(.touched) {
    transition: transform 0.2s ease;
    transform: translate(0, -30px);
  }
  &:nth-child(1) {
    top: 10%;
    left: 0;
    .pouf {
      transform: scale(-1, 1);
    }
  }
  &:nth-child(2) {
    top: 10%;
    left: 24%;
    .chair {
      transform: rotate(10deg);
    }
  }
  &:nth-child(3) {
    top: 12%;
    left: 48%;
  }
  &:nth-child(4) {
    top: 14%;
    left: 72%;
    .chair {
      transform: rotate(-10deg);
    }
  }

  &:nth-child(5) {
    top: 42%;
    left: 5%;
    .pouf {
      transform: scale(-1, 1);
    }
  }
  &:nth-child(6) {
    top: 43%;
    left: 34%;
  }
  &:nth-child(7) {
    top: 40%;
    left: 65%;
    .chair {
      transform: rotate(8deg);
    }
  }

  &:nth-child(8) {
    top: 76%;
    left: 2%;
    .chair {
      transform: rotate(6deg);
    }
    .pouf {
      transform: scale(-1, 1);
    }
  }
  &:nth-child(9) {
    top: 74%;
    left: 28%;
    .chair {
      transform: rotate(-5deg);
    }
  }
  &:nth-child(10) {
    top: 76%;
    left: 60%;
  }
  img {
    width: 100%;
  }
}
.name {
  position: absolute;
  left: 50%;
  bottom: 20%;
  background-color: rgba(255, 255, 255, 0.8);
  display: inline-block;
  padding: 4px 15px;
  border-radius: 8px;
  font-weight: bold;
  font-size: 13px;
  transform: translate(-50%, 0);
}
.events {
  position: absolute;
  top: 40%;
  right: 10%;
  &:global(-enter-active) {
    transition: transform 1.6s ease, opacity 1.6s ease;
  }
  &:global(-leave-active) {
    transition: opacity 0.5s ease;
  }

  &:global(-enter) {
    transform: translate(0, 100%);
    opacity: 0;
  }
  &:global(-leave-to) {
    opacity: 0;
  }
}
.event {
  position: absolute;
  top: 0;
  right: 0;
  font-size: 18px;
  display: flex;
  align-items: center;
  color: greenyellow;
  white-space: nowrap;
  z-index: 4;
  .icon {
    margin-right: 4px;
  }
  .coin {
    width: 20px;
    height: 20px;
    margin: 0 4px;
  }
  .currency {
    font-size: 12px;
    align-self: flex-end;
  }
}
.event + .event {
  margin-top: 30px;
}
//
//.events-enter-active,
//.events-leave-active {
//  transition: all 0.5s ease;
//}
//.events-enter-from,
//.events-leave-to {
//  opacity: 0;
//  transform: translateX(30px);
//}
</style>
