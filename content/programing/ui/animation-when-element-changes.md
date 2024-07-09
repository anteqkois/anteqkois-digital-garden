---
title: Animate HTML element when content changes
tags:
  - dev
  - vuejs
  - animation
  - ts
  - ui
---

The main ide is to animate table cell when the contane changes. Due to amount of cells i can't use [Custom Events](https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events). The sollution is to use [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API) to listen for `childList` change type and using changed element append css classes and listeners to remove after animation

## Example in VueJS (with Quasar library)

```vue
<template>
  <q-card>
    <q-table
		<!-- ... -->
    >
			<!-- ... -->
      	<template v-slot:body-cell-orderBook_2="props">
					<q-td
      	    :props="props"
      	    style="min-width: 30px; max-width: 30px"
      	    class="coins-list__body-cell"
      	    :class="selectedCoin?._id === props.row._id ? 'selected' : ''"
      	  >
      	    <div
      	      v-if="typeof props.value === 'number'"
      	      :data-sentiment="props.value > 0 ? 'positive' : props.value < 0 ? 'negative' : ''"
      	      :style="{
								color:
      	          props.value > 0
      	            ? `hsl(177, ${Math.abs(props.value * 100) + 50}%, 47%)`
      	            : props.value < 0
      	              ? `hsl(323, ${Math.abs(props.value * 100) + 50}%, 75%)`
      	              : '',
      	      }"
      	    >
      	      {{ props.value.toFixed(3) }}
      	    </div>
      	    <template v-else>-</template>
      	  </q-td>
      	</template>
			<!-- ... -->
    </q-table>
  </q-card>
</template>

<script lang="ts">
import { useCoinsList } from "src/composables/coins"
import { watch, onMounted, onUnmounted, ref } from "vue"
import {
  Coin,
  CoinsUpdateData,
  WEBSOCKET_EVENT,
} from "src/composables/coins/models"
import { useCoinsWebSocket } from "src/composables/coins/ws"

export default {
  name: "CoinList",
  emits: ["onSelectCoin"],
  setup(props: any, { emit }: any) {
    const {
      coins,
      scrollCoinsTargetRef,
    } = useCoinsList()
    const { connectSocket, waitUntilSocketConnected, unsubscribeFromCoinsUpdate, socket } =
      useCoinsWebSocket()

    const initObservers = () => {
      const observerCallback: MutationCallback = (mutationsList) => {
        for (const mutation of mutationsList) {
          if (mutation.type !== "childList") continue

          const target = mutation.target.parentElement!
          const sentiment = mutation.target?.dataset?.sentiment

          const className = `change_${sentiment}`
          target.classList.add(className)
          target.addEventListener(
            "animationend",
            () => {
              target.classList.remove(className)
            },
            { once: true },
          )
        }
      }

      const observer = new MutationObserver(observerCallback)

      if (!scrollCoinsRef.value) {
        return console.error(`#initObservers missing scrollCoinsRef`)
      }

      const table = scrollCoinsRef.value.$el
      observer.observe(table, {
        childList: true,
        characterData: true,
        subtree: true,
      })
    }

    onMounted(async () => {
      await connectSocket()
      await waitUntilSocketConnected()
      if (!socket.value) return

      socket.value.emit(WEBSOCKET_EVENT.JOIN_COINS_UPDATE)
      setTimeout(() => {
        initObservers()
      }, 2_000)

      socket.value.on(WEBSOCKET_EVENT.COINS_UPDATE, ({ update }: { update: CoinsUpdateData }) => {
        coins.value = coins.value.map((coin) => {
          const updateForCoin = update[coin._id]
          if (!updateForCoin) return coin
          return { ...coin, ...updateForCoin }
        })
      })
    })

    onUnmounted(async () => {
      await unsubscribeFromCoinsUpdate()
    })

    return {
      coins,
      scrollCoinsTargetRef,
    }
  },
}
</script>

<style lang="scss" scoped>
@keyframes fadeInPositive {
  0% {
    background-color: hsl(177, 93%, 47%, 20%);
    color: black;
  }
  20% {
    background-color: hsl(177, 93%, 47%, 60%);
    color: black;
  }
  70% {
    background-color: hsl(177, 93%, 47%, 60%);
    color: black;
  }
  80% {
    background-color: hsl(177, 93%, 47%, 20%);
    color: black;
  }
}
@keyframes fadeInNegative {
  0% {
    background-color: hsl(323, 100%, 75%, 20%);
    color: black;
  }
  20% {
    background-color: hsl(323, 100%, 75%, 60%);
    color: black;
  }
  70% {
    background-color: hsl(323, 100%, 75%, 60%);
    color: black;
  }
  80% {
    background-color: hsl(323, 100%, 75%, 20%);
    color: black;
  }
}

.change_positive {
  animation: fadeInPositive 1.5s ease-in-out;
}
.change_negative {
  animation: fadeInNegative 1.5s ease-in-out;
}
</style>
```
