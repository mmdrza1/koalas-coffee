<template>
  <div ref="card" class="light full-width d-flex rounded pa-2 shadow">
    <div
      :class="[
        cartCard.image,
        'secondary pa-2 ml-4 rounded d-flex justify-center align-center',
      ]"
    >
      <img class="d-shadow" :src="image" alt="src" />
    </div>
    <div
      class="
        full-height
        grow-1
        d-flex
        flex-column
        justify-space-between
        overflow-hidden
        ml-2
      "
    >
      <h3 class="primary-color mb-4 font-size-8 regular more">
        {{ title || "بدون نام" }}
        <br />
        <span v-if="has_mill" class="bold">
          ( {{ has_mill && mill?.title ? mill.title : "بدون آسیاب" }} )
        </span>
      </h3>
      <div>
        <p class="mb-2">قیمت نهایی:</p>
        <p class="bold primary-color font-size-12">
          {{ finalPrice }}
        </p>
      </div>
    </div>
    <div class="d-flex flex-column align-end justify-space-between px-2">
      <div
        :class="[
          cartCard.qty,
          { 'flex-column': screenSize.smAndDown },
          'd-flex justify-space-between align-center',
        ]"
      >
        <button class="mx-2" @click="increase">
          <span class="icon-plus" />
        </button>
        <p class="bold primary secondary-color px-4 py-2 rounded">
          {{ qty }}
        </p>
        <button class="mx-2" @click="decrease">
          <span class="icon-minus" />
        </button>
      </div>
      <button v-if="!screenSize.smAndDown" @click="remove" class="mx-2">
        <span class="icon-delete" />
      </button>
    </div>
  </div>
</template>

<script>
import { mapState } from "pinia";
import { useGlobalVariable } from "@/store";
import { convertToRls } from "@/helpers/text";
export default {
  props: {
    id: {},
    image: {},
    title: {},
    price: {},
    has_mill: {},
    mill: {},
    qty: {
      type: Number,
      required: true,
    },
  },
  emits: ["increase", "decrease", "remove"],
  data() {
    return {
      touchStartPos: 0,
      touchEndPos: 0,
    };
  },
  computed: {
    ...mapState(useGlobalVariable, ["screenSize"]),
    finalPrice() {
      return convertToRls(this.price * this.qty) || 0;
    },
  },
  methods: {
    increase() {
      this.$emit("increase", { id: this.id, mill: this.mill });
    },
    decrease() {
      this.$emit("decrease", { id: this.id, mill: this.mill });
    },
    remove() {
      return this.$emit("remove", { id: this.id, mill: this.mill });
    },
  },
  watch: {
    qty(value) {
      if (!value) {
        return this.$emit("remove", this.id);
      }
    },
  },
  mounted() {
    this.$refs.card.addEventListener("touchstart", (e) => {
      this.touchStartPos = e.touches[0].clientX;
    });

    this.$refs.card.addEventListener("touchmove", (e) => {
      let distance = e.touches[0].clientX - this.touchStartPos;
      if (distance > 100 || distance < -100) {
        this.$refs.card.style.transform = `translate(${distance}px)`;
        this.touchEndtPos = e.touches[0].clientX;
      }
    });

    this.$refs.card.addEventListener("touchend", () => {
      this.touchStartPos = 0;
      if (
        Math.abs(this.touchEndtPos) - this.touchStartPos >
        this.$refs.card.clientWidth
      ) {
        return this.$emit("remove", this.id);
      } else {
        this.$refs.card.style.transform = `translate(0px)`;
      }
    });
  },
};
</script>

<style lang="scss" module="cartCard">
.image {
  width: 25%;
  min-width: 25%;

  img {
    aspect-ratio: 1;
    object-fit: contain;
    width: 100%;
  }
}
.qty {
  width: fit-content;
}
</style>
