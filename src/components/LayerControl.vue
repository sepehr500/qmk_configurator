<template>
  <div>
    <p>
      <label>{{ $t('layer.label') }}:</label>
    </p>
    <div class="layers">
      <!-- prettier-ignore -->
      <div
        class="layer"
        :class="layer.clazz"
        v-for="layer in layers"
        :key="layer.id"
        @click="clicked(layer.id)"
      >{{ layer.name }}</div>
    </div>
    <div class="layer-modifiers">
      <button
        :aria-label="$t('layer.title')"
        style="margin-bottom: 10px;"
        class="clear-button"
        v-tooltip="$t('layer.title')"
        @click="clearLayer"
      >
        <font-awesome-icon icon="trash" size="lg" fixed-width />
      </button>
      <button
        :aria-label="$t('layer.clone')"
        class="clear-button"
        v-tooltip="$t('layer.clone')"
        @click="cloneLayer"
        :disabled="keymap.length >= maxLayers"
      >
        <font-awesome-icon icon="copy" size="lg" fixed-width />
      </button>
    </div>
  </div>
</template>
<script>
import { MAX_LAYERS } from '../store/modules/constants.js';
import rangeRight from 'lodash/rangeRight';
import isUndefined from 'lodash/isUndefined';
import { mapState, mapGetters, mapMutations } from 'vuex';
export default {
  name: 'layer-control',
  computed: {
    ...mapState('keymap', ['layer', 'keymap']),
    ...mapState('app', ['configuratorSettings']),
    ...mapGetters('keymap', ['getLayer']),
    maxLayers() {
      return MAX_LAYERS;
    },
    layers() {
      let layers = rangeRight(16).map(layer => {
        let clazz = [layer];
        let _layer = this.getLayer(layer);
        if (!isUndefined(_layer)) {
          clazz.push('non-empty');
        }
        if (this.layer == layer) {
          clazz.push('active');
        }
        return {
          id: layer,
          name: layer,
          clazz: clazz.join(' ')
        };
      });

      return layers;
    },
    defaultClearLayerCode() {
      return this.configuratorSettings.clearLayerDefault ? 'KC_TRNS' : 'KC_NO';
    }
  },
  methods: {
    ...mapMutations('keymap', ['changeLayer', 'initLayer', 'duplicateLayer']),
    clicked(id) {
      if (isUndefined(this.getLayer(id))) {
        this.initLayer({
          layer: id,
          code: this.defaultClearLayerCode
        });
      }
      this.changeLayer(id);
    },
    cloneLayer() {
      this.duplicateLayer({ layer: this.layer });
    },
    clearLayer() {
      if (confirm(this.$t('layer.confirm'))) {
        this.initLayer({
          layer: this.layer,
          code: this.defaultClearLayerCode
        });
        this.$store.commit('keymap/setDirty');
      }
    }
  }
};
</script>
<style>
.clear-button {
  line-height: 100%;
  margin: 0;
  border-radius: 3px;
  border: 0px solid;
  padding: 6px 8px;
  cursor: pointer;
}
.clear-button:disabled {
  background: gray;
  cursor: auto;
}
.layer-modifiers {
  display: flex;
  flex-direction: column;
  padding: 0 13px;
}
</style>
