<template>
  <main>
    <button
      v-for="(identity, index) in identities"
      :key="index"
      :style="{
        'border-color': identity.colorCode,
        color: identity.colorCode,
      }"
      @click="switchIdentity(identity)">
      <div
        :style="{
          background: identity.colorCode,
          'mask-image': `url(${identity.iconUrl})`,
        }"
        class="icon"
      />
      <span class="identity-name">{{ identity.name }}</span>
      <p>Shortcut : {{ String.fromCharCode(49 + index) }}</p>
    </button>
  </main>
</template>

<script type="application/javascript">
const KEYDOWN = 'keydown';

export default {
  data() {
    return {
      identities: [],
    };
  },
  async created() {
    this.identities = await browser.contextualIdentities.query({});
  },
  mounted() {
    document.addEventListener(KEYDOWN, this.keyDownEvent);
  },
  beforeDestroy() {
    document.removeEventListener(KEYDOWN, this.keyDownEvent);
  },
  methods: {
    keyDownEvent({ keyCode }) {
      const i = keyCode - 49;
      if (i >= 0 && this.identities.length > (i + 1)) {
        this.switchIdentity(this.identities[i]);
      }
    },
    async getCurrentTab() {
      return browser.tabs.getCurrent();
    },
    async switchIdentity({cookieStoreId}) {
      const {id: currentId} = await this.getCurrentTab();

      await browser.tabs.create({ cookieStoreId });
      await browser.tabs.remove(currentId);
    },
  },
}
</script>

<style lang="scss" scoped>
main {
  list-style: none;
  display: flex;
  button {
    background: none;
    border: 1px solid;
    border-radius: 20px;
    margin: 10px;
    padding: 10px;
    flex-grow: 1;
  }
  .icon {
    height: 10em;
    width: 100%;
    mask: no-repeat 50% 50%;
  }
  .identity-name {
    font-size: 2em;
    font-weight: bold;
  }
}
</style>
