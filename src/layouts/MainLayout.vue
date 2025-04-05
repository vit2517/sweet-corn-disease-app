<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title>
          Quasar App
        </q-toolbar-title>

        <div>Quasar v{{ $q.version }}</div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
    >
      <q-list>
        <q-item-label
          header
        >
          ประเภทของโรค ข้อมูลเพิ่มเติม
        </q-item-label>

        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from 'vue'
import EssentialLink from 'components/EssentialLink.vue'

const linksList = [
  {
    title: 'โรคราน้ำค้าง',
    caption: 'การป้องกันรักษา',
    icon: 'Corn',
    link: 'https://www.doa.go.th/fc/nakhonsawan/?p=1441'
  },
  {
    title: 'โรคใบด่างแคระ',
    caption: 'การป้องกันรักษา',
    icon: 'Corn',
    link: 'https://nsfcrc-news.blogspot.com/2008/04/blog-post_18.html'
  },
  {
    title: 'โรคใบไหม้แผลใหญ่',
    caption: 'การป้องกันรักษา',
    icon: 'Corn',
    link: 'https://nsfcrc-news.blogspot.com/2015/11/blog-post_13.html'
  },
  {
    title: 'โรคใบไหม้แผลเล็ก',
    caption: 'การป้องกันรักษา',
    icon: 'Corn',
    link: 'https://www.doa.go.th/fc/nakhonsawan/?p=3793'
  },
  {
    title: 'โรคใบด่างอ้อย',
    caption: 'การป้องกันรักษา',
    icon: 'Corn',
    link: 'https://www.doa.go.th/fc/nakhonsawan/?p=3281'
  },
  {
    title: 'โรคราสนิม',
    caption: 'การป้องกันรักษา',
    icon: 'Corn',
    link: 'https://nsfcrc-news.blogspot.com/2009/10/blog-post_12.html'
  },
  {
    // title: 'Quasar Awesome',
    // caption: 'Community Quasar projects',
    // icon: 'favorite',
    // link: 'https://awesome.quasar.dev'
  }
]

export default defineComponent({
  name: 'MainLayout',

  components: {
    EssentialLink
  },

  setup () {
    const leftDrawerOpen = ref(false)

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer () {
        leftDrawerOpen.value = !leftDrawerOpen.value
      }
    }
  }
})
</script>
