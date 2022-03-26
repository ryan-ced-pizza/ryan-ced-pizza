<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="bg-deep-orange-9 text-white">
        <q-btn flat round dense icon="assignment_ind" />
        <q-toolbar-title>
          <!-- {{ state.task }} -->
          Adutukart
        </q-toolbar-title>
        <div class="row q-gutter-sm">
          <q-btn icon="shopping_cart" round dense flat >
            <q-badge v-show="$global.cart.length" color="red" floating>{{ $lodash.sumBy($global.cart, 'quantity') }}</q-badge>
            <q-popup-proxy v-model="$global.cartIsOpen">
              <q-card style="min-width: 300px;">
                <q-card-section class="row q-gutter-sm">
                  <div class="text-h6 row items-center q-gutter-xs">
                    <q-icon name="shopping_cart" />
                    <div>Cart<span v-if="!$global.cart.length"> is empty</span></div>
                  </div>
                  <q-space />
                  <!-- <q-btn v-show="$global.cart.length" @click="clearCart" class="bg-grey-2" size="sm" flat label="clear" v-close-popup/> -->
                  <q-btn v-show="$global.cart.length" :loading="$global.reviewing" @click="review" class="bg-grey-2" size="sm" flat label="review"></q-btn>
                </q-card-section>

                <q-separator />

                <q-card-section style="max-height: 50vh" class="scroll">
                  <q-list>
                    <q-slide-item @right="$global.cart.splice(ind, 1), LocalStorage.set('cartt', $global.cart)" right-color="red" v-for="(ci, ind) in $global.cart" :key="'cart'+ci.id">
                      <q-item>
                        <q-item-section top thumbnail class="q-ml-none">
                          <img :src="$api.defaults.baseURL.replace('/api', '') + ci.images[0].url">
                        </q-item-section>

                        <q-item-section>
                          <q-item-label>{{ ci.name }}</q-item-label>
                          <q-item-label caption>{{ ci.description }}</q-item-label>
                        </q-item-section>

                        <q-item-section side top>
                          <q-item-label class="text-red">{{ ci.price.toLocaleString('en-PH', { style: 'currency', currency: 'PHP' }) }}</q-item-label>
                          <q-item-label caption class="q-gutter-sm row no-wrap items-center">
                            <q-btn @click="() => { if (ci.quantity > 1) ci.quantity-- }" icon="remove" dense size="sm" />
                            <div>{{ci.quantity}}</div>
                            <q-btn @click="() => { if (ci.quantity < ci.orderLimit) ci.quantity++ }" icon="add" dense size="sm" />
                          </q-item-label>
                        </q-item-section>
                      </q-item>
                      <template v-slot:right>
                        <q-icon name="delete" />
                      </template>
                    </q-slide-item>
                  </q-list>
                </q-card-section>

                <q-separator />

                <q-card-actions class="q-gutter-md row" v-show="$global.cart.length">
                  <q-space />
                  <div class="text-subtitle1">
                    Total:
                    <span class="text-red text-h6">{{$lodash.sumBy($global.cart, ci => ci.quantity * ci.price).toLocaleString('en-PH', { style: 'currency', currency: 'PHP' })}}</span>
                  </div>
                  <!-- <q-btn flat label="Decline" color="primary" v-close-popup /> -->
                  <q-btn label="Place Order" color="orange" :loading="$global.loading" >
                    <q-popup-proxy v-model="$global.placingOrder" anchor="top start" self="bottom left">
                      <q-card style="min-width: 300px;">
                        <q-card-section class="row">
                          <div class="text-h6 row items-center q-gutter-xs">
                            <q-icon name="place" />
                            <div>Deliver to</div>
                          </div>
                          <q-space />
                          <!-- <q-btn v-show="$global.cart.legnth" @click="clearCart" class="bg-grey-2" size="sm" flat label="clear" v-close-popup/> -->
                        </q-card-section>

                        <q-separator />

                        <q-card-section style="max-height: 50vh" class="scroll">
                          <q-input label="address" v-model="$global.address"></q-input>
                          <q-input label="landmark" v-model="$global.landmark"></q-input>
                          <q-input label="contact No." v-model="$global.contactNo"></q-input>
                        </q-card-section>

                        <q-separator />

                        <q-card-actions align="right" class="q-gutter-md" v-show="$global.cart.length">
                          <div class="text-subtitle1">
                            Total:
                            <span class="text-red text-h6">{{$lodash.sumBy($global.cart, ci => ci.quantity * ci.price).toLocaleString('en-PH', { style: 'currency', currency: 'PHP' })}}</span>
                          </div>
                          <!-- <q-btn flat label="Decline" color="primary" v-close-popup /> -->
                          <q-btn label="Confirm" color="primary" @click="confirmOrder" :loading="$global.loading" />
                        </q-card-actions>
                      </q-card>
                    </q-popup-proxy>
                  </q-btn>
                </q-card-actions>
              </q-card>
            </q-popup-proxy>
          </q-btn>
          <q-btn v-if="!$global.user" @click="$popupCenter({url: $api.defaults.baseURL + '/connect/google', title: 'Manage Account', w: 900, h: 600})" flat round dense icon="person" class="q-mr-xs" />
          <q-btn v-else color="white" class="text-red" round dense label="TB" >
            <q-popup-proxy breakpoint="0" v-model="$global.profileMenuIsOpen">
              <q-list class="bg-white">
                <q-item clickable v-ripple>
                  <q-item-section>Orders</q-item-section>
                  <q-item-section avatar>
                    <q-icon color="red" name="redeem" />
                  </q-item-section>
                  <q-popup-proxy v-model="$global.ordersIsOpen">
                    <q-card style="min-width: 300px;">
                      <q-card-section class="row">
                        <div class="text-h6 row items-center q-gutter-xs">
                          <q-icon name="delivery_dining" />
                          <div>Orders</div>
                        </div>
                        <q-space />
                        <!-- <q-btn v-show="$global.cart.legnth" @click="clearCart" class="bg-grey-2" size="sm" flat label="clear" v-close-popup/> -->
                      </q-card-section>

                      <q-separator />

                      <q-card-section style="max-height: 50vh" class="scroll">
                        <div class="column col q-gutter-md">
                          <q-list v-for="o in $global.orders" :key="o.refNo">
                            <div class="row items-center q-gutter-md">
                              <div>
                                <q-chip v-if="o.status.data">
                                  <q-avatar :icon="o.status.data.attributes.icon" :color="o.status.data.attributes.color" text-color="white" />
                                  {{ o.status.data.attributes.name }}
                                </q-chip>
                                <q-chip v-else >
                                  <q-avatar icon="pending" color="orange" text-color="white" />
                                  On queue
                                </q-chip>
                              </div>
                              <q-space />
                              <div>{{ o.refNo }}</div>
                              <div class="text-red">{{ $lodash.sumBy(o.itemsQuantity, ci => ci.quantity * o.items.data.filter(i => i.id === ci.id)[0].attributes.price).toLocaleString('en-PH', { style: 'currency', currency: 'PHP' }) }}</div>
                            </div>
                            <q-slide-item right-color="red" v-for="(ci) in o.itemsQuantity" :key="o.refNo+ci.id">
                              <q-item>
                                <q-item-section top thumbnail class="q-ml-none">
                                  <img :src="$api.defaults.baseURL.replace('/api', '') + ci.images[0].url">
                                </q-item-section>

                                <q-item-section>
                                  <q-item-label>{{ ci.name }}</q-item-label>
                                  <q-item-label caption>{{ ci.description }}</q-item-label>
                                </q-item-section>

                                <q-item-section side top>
                                  <q-item-label>{{ o.items.data.filter(i => i.id === ci.id)[0].attributes.price.toLocaleString('en-PH', { style: 'currency', currency: 'PHP' }) }}</q-item-label>
                                  <q-item-label caption class="q-gutter-sm row no-wrap items-center">
                                    <!-- <q-btn @click="() => { if (ci.quantity > 1) ci.quantity-- }" icon="remove" dense size="sm" /> -->
                                    <div class="row no-wrap"><div>Qty: {{ci.quantity}}</div></div>
                                    <!-- <q-btn @click="() => { if (ci.quantity < ci.orderLimit) ci.quantity++ }" icon="add" dense size="sm" /> -->
                                  </q-item-label>
                                </q-item-section>
                              </q-item>
                              <template v-slot:right>
                                <q-icon name="delete" />
                              </template>
                            </q-slide-item>
                          </q-list>
                        </div>
                      </q-card-section>

                      <q-separator />

                      <!-- <q-card-actions align="right" class="q-gutter-md" v-show="$global.cart.length">
                        <div class="text-subtitle1">
                          Total:
                          <span class="text-red text-h6">{{$lodash.sumBy($global.cart, ci => ci.quantity * ci.price).toLocaleString('en-PH', { style: 'currency', currency: 'PHP' })}}</span>
                        </div>
                        <q-btn flat label="Decline" color="primary" v-close-popup />
                        <q-btn label="Confirm" color="primary" @click="confirmOrder" :loading="$global.loading" />
                      </q-card-actions> -->
                    </q-card>
                  </q-popup-proxy>
                </q-item>
                <q-item clickable v-ripple>
                  <q-item-section @click="logout" >Logout</q-item-section>
                  <q-item-section avatar>
                    <q-icon color="grey" name="logout" />
                  </q-item-section>
                </q-item>
              </q-list>
            </q-popup-proxy>
          </q-btn>
        </div>
      </q-toolbar>
    </q-header>

    <!-- <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
    >
      <q-list>
        <q-item-label
          header
        >
          Essential Links
        </q-item-label>

        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer> -->

    <q-page-container>
      <q-page>
        <router-view />
        <div class="fixed-top-right column" style="top: 80px; right: 16px;">
          <div v-for="(o, i) in $global.orders" :key="'activeOrders'+o.refNo" >
            <q-chip clickable v-if="o.status.data && o.status.data.attributes.name.toLowerCase() !== 'delivered'" :ref="el => activeOrders[i] = el" class="shadow-10 shadow-transition bg-white">
              <q-avatar :icon="o.status.data.attributes.icon" :color="o.status.data.attributes.color" text-color="white" />
              <span @click="$global.ordersIsOpen = $global.profileMenuIsOpen = true">
                {{ o.status.data.attributes.name }}
              </span>
            </q-chip>
            <q-chip clickable v-else-if="!o.status.data" :ref="el => activeOrders[i] = el" class="bg-white shadow-10 shadow-transition">
              <q-avatar icon="pending" color="orange" text-color="white" />
              <span @click="$global.ordersIsOpen = $global.profileMenuIsOpen = true">
                On queue
              </span>
            </q-chip>
          </div>
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { getCurrentInstance, onUnmounted, ref } from 'vue'
import { LocalStorage, useQuasar } from 'quasar'
const $q = useQuasar()

const activeOrders = ref([])

const app = getCurrentInstance()
const { $api, $global, $nanoid, $qs, $chupipay, $lodash } = app.appContext.config.globalProperties

console.log($global.user)

const timer = setInterval(async () => {
  console.log('activeOrders', activeOrders.value)
  activeOrders.value.forEach((o, i) => {
    setTimeout(() => o.$el.click(), i * 200)
    setTimeout(() => o.$el.click(), i * 500)
  })
  if ($global.user) {
    const query = {
      sort: ['updatedAt:desc'],
      populate: ['items', 'status'],
      filters: {
        user: $global.user.id
      }
    }
    // console.log($qs.stringify(query))
    const { data: { data } } = await $api.get('/orders?' + $qs.stringify(query))
    $global.orders = data.map(o => ({ id: o.id, ...o.attributes }))
    console.log($global.orders, $qs.stringify(query))
  }
}, 2000)

onUnmounted(() => {
  clearInterval(timer)
})

function clearCart () {
  $global.cart = []
  LocalStorage.set('cartt', $global.cart)
}

function logout () {
  $global.cart = []
  $global.user = null
  LocalStorage.remove('userr')
}

async function confirmOrder () {
  $global.loading = true
  LocalStorage.set('addresss', $global.address)
  LocalStorage.set('landmarkk', $global.landmark)
  LocalStorage.set('contactNoo', $global.contactNo)

  const order = {
    data: {
      refNo: new Date().getFullYear() + '-' + $nanoid(),
      items: $global.cart.map(c => c.id),
      itemsQuantity: $global.cart,
      address: $global.address,
      landmark: $global.landmark,
      contactNo: $global.contactNo,
      user: $global.user.id
    }
  }

  console.log('order', order)

  try {
    await $api.post('/orders', order)
    clearCart()
    $global.cartIsOpen = false

    $q.notify({
      message: `${order.data.refNo} order placed`,
      color: 'purple'
    })
  } catch (error) {
    console.log(error)
  }
  $global.loading = false
}

function review () {
  $global.reviewing = true
  $global.cartIsOpen = true
  let msg = $global.cart.map(i => {
    return `${i.name}. ${i.price} Pesos, ${i.quantity > 1 ? i.quantity + ' quantity' : ''} .`
  }).join('. ')

  msg += `. Total of ${$lodash.sumBy($global.cart, i => i.quantity * i.price)} Pesos`

  $chupipay.say(msg, {
    onEnd () {
      $global.reviewing = false
    }
  })
}

const commands = [
  {
    indexes: ['review my cart', 'check my cart', 'review cart', 'check cart', 'cheque cart', 'cheque my cart', 'open cart', 'open my cart'], // These spoken words will trigger the execution of the command
    action: function () { // Action to be executed when a index match with spoken word
      console.log('review my cart')
      review()
    }
  },
  {
    indexes: ['place order'], // These spoken words will trigger the execution of the command
    action: function () { // Action to be executed when a index match with spoken word
      $global.placingOrder = true
      $chupipay.say('please confirm your delivery address and contact info.')
    }
  }
]
$chupipay.addCommands(commands)
$chupipay.on(['Good morning']).then(function (i) {
  console.log('Triggered')
})

$chupipay.on(['what is the menu?', 'what\'s the menu?']).then(function (i) {
  const msg = $global.items.map(i => {
    return `${i.name}. ${i.price} Pesos`
  }).join('. ')

  $chupipay.say(msg)
})

</script>
