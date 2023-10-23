<template>
  <Teleport to="body">
    <div class="modal-container" v-show="showModal">
      <div class="modal" v-if="showModal">
        <slot />
        <button type="button" class="modal--btn" @click="$emit('hide-modal')">
          Close
        </button>
      </div>
    </div>
  </Teleport>
</template>

<script setup>
defineEmits(["hide-modal"]);
defineProps({
  showModal: {
    type: Boolean,
    default: false,
  },
});
</script>
