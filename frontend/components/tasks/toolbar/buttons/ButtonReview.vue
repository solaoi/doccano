<template>
  <v-tooltip bottom>
    <template #activator="{ on }">
      <v-btn
        v-shortkey.once="{enter: ['enter'], a: ['a']}"
        icon
        v-on="on"
        @shortkey="$emit('click:review')"
        @click="$emit('click:review')"
      >
        <v-icon v-if="isReviewd" style="color: #4CAF50">
          {{ mdiCheck }}
        </v-icon>
        <v-icon v-else style="color: #F44336">
          {{ mdiClose }}
        </v-icon>
      </v-btn>
    </template>
    <span v-if="isReviewd">{{ $t('annotation.checkedTooltip') }}</span>
    <span v-else>{{ $t('annotation.notCheckedTooltip') }}</span>
  </v-tooltip>
</template>

<script lang="ts">
import Vue from 'vue'
import { mdiClose, mdiCheck } from '@mdi/js'

export default Vue.extend({
  props: {
    isReviewd: {
      type: Boolean,
      default: false,
      required: true
    }
  },

  data() {
    return {
      mdiClose,
      mdiCheck
    }
  }
})
</script>
