<template>
  <div class="modal" v-if="tile" @click="closeModal">
    <div class="modal-content" @click.stop>
      <div class="color-picker-container">
        <div
          class="colored-div-picker"
          :style="{ backgroundColor: tile.bgColor }"
        ></div>
        <div class="right-side">
          <label>
            Wybierz format:
            <select
              class="formatChoice"
              v-model="tile.colorFormat"
              @change="checkFormat"
            >
              <option value="hex">Hex</option>
              <option value="rgb">RGB</option>
            </select>
          </label>
          <div class="hexInput" v-if="tile.colorFormat === 'hex'">
            <label>
              Hex:
              <input
                v-model="tile.bgColor"
                maxlength="7"
                placeholder="Wpisz kolor hex"
              />
            </label>
          </div>
          <div class="rgbInputs" v-if="tile.colorFormat === 'rgb'">
            <label>
              R:
              <input
                v-model="tile.rgb.r"
                type="number"
                maxlength="3"
                min="0"
                max="255"
                @input="updateColor"
              />
            </label>
            <label>
              G:
              <input
                v-model="tile.rgb.g"
                type="number"
                maxlength="3"
                min="0"
                max="255"
                @input="updateColor"
              />
            </label>
            <label>
              B:
              <input
                v-model="tile.rgb.b"
                type="number"
                maxlength="3"
                min="0"
                max="255"
                @input="updateColor"
              />
            </label>
            <label>
              A:
              <input
                v-model="tile.rgb.a"
                type="number"
                step="0.01"
                min="0"
                max="1"
                maxlength="3"
                @input="updateColor"
              />
            </label>
          </div>
        </div>
      </div>
      <div class="buttons-container">
        <button class="cancel-button" @click="closeModal">Anuluj</button>
        <button class="submit-button" @click="changeColor">Zatwierdź</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch, onMounted } from "vue";

export default {
  props: {
    tile: Object,
  },
  setup(props, { emit }) {
    const tile = ref(props.tile);

    const changeColor = () => {
      if (isValidColor()) {
        emit("changeColor", { ...tile.value });
        emit("showColorPicker", false);
      } else {
        alert("Wprowadź poprawny kolor.");
      }
    };

    const isValidColor = () => {
      if (tile.value.colorFormat === "hex") {
        return /^#[0-9A-Fa-f]{6}$/.test(tile.value.bgColor);
      } else if (tile.value.colorFormat === "rgb") {
        const rgb = tile.value.rgb;
        return (
          rgb.r >= 0 &&
          rgb.r <= 255 &&
          rgb.g >= 0 &&
          rgb.g <= 255 &&
          rgb.b >= 0 &&
          rgb.b <= 255 &&
          rgb.a >= 0 &&
          rgb.a <= 1
        );
      }
      return false;
    };

    const updateColor = () => {
      tile.value.bgColor =
        tile.value.colorFormat === "hex"
          ? rgbToHex(tile.value.rgb.r, tile.value.rgb.g, tile.value.rgb.b)
          : `rgba(${tile.value.rgb.r}, ${tile.value.rgb.g}, ${tile.value.rgb.b}, ${tile.value.rgb.a})`;
    };

    const checkFormat = () => {
      if (tile.value.colorFormat === "rgb") {
        const { r, g, b } = hexToRgb(tile.value.bgColor);
        tile.value.rgb.r = r;
        tile.value.rgb.g = g;
        tile.value.rgb.b = b;
      } else if (tile.value.colorFormat === "hex") {
        tile.value.bgColor = rgbToHex(
          tile.value.rgb.r,
          tile.value.rgb.g,
          tile.value.rgb.b
        );
      }
    };

    const hexToRgb = (hex) => {
      hex = hex.replace(/^#/, "");
      const bigint = parseInt(hex, 16);
      const r = (bigint >> 16) & 255;
      const g = (bigint >> 8) & 255;
      const b = bigint & 255;
      tile.value.rgb.r = r;
      tile.value.rgb.g = g;
      tile.value.rgb.b = b;
      return { r, g, b };
    };

    const rgbToHex = (r, g, b) => {
      const toHex = (c) => {
        const hex = c.toString(16);
        return hex.length === 1 ? "0" + hex : hex;
      };
      return `#${toHex(r)}${toHex(g)}${toHex(b)}`;
    };

    const closeModal = () => {
      emit("showColorPicker", false);
    };

    watch(
      () => props.tile,
      () => {
        tile.value = { ...props.tile };
        if (tile.value.bgColor.startsWith("rgba")) {
          tile.value.colorFormat = "rgb";
        } else if (tile.value.bgColor.startsWith("#")) {
          tile.value.colorFormat = "hex";
        }
      }
    );

    onMounted(() => {
      tile.value = { ...props.tile };
      if (tile.value.bgColor.startsWith("rgba")) {
        tile.value.colorFormat = "rgb";
      } else if (tile.value.bgColor.startsWith("#")) {
        tile.value.colorFormat = "hex";
      }
    });

    return {
      tile,
      changeColor,
      closeModal,
      checkFormat,
      updateColor,
    };
  },
};
</script>
